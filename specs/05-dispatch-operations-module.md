# Especificación de Requerimientos: Módulo de Operaciones de Despacho y Entrega

**Módulo:** `orders` (Subcontexto Operativo)  
**Bounded Context:** Tablero de Despacho, Verificación de Pagos, Transiciones de Estado y Logística de Entrega  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
Este módulo gobierna las operaciones internas en cada sucursal farmacéutica de **Hospisalud**. El Operador de Despacho supervisa en tiempo real las órdenes entrantes, verifica comprobantes de pago (Pago Móvil y Binance Pay), coordina el armado digital de los paquetes, comisiona las entregas a los repartidores motorizados y formaliza las entregas y retiros en taquilla.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-DSP-01`: El sistema debe presentar al Operador de Despacho un tablero kanban/tabla filtrado exclusivamente por su sede asignada.
2. `EARS-DSP-02`: El sistema debe realizar polling corto (TanStack Query cada 5 a 10 segundos) sobre los pedidos pendientes de verificación.
3. `EARS-DSP-03`: El sistema debe reproducir una alerta sonora auditiva (*chime*) y una notificación visual cada vez que se detecte un nuevo pedido en estado `PENDIENTE_VERIFICACION`.
4. `EARS-DSP-04`: El sistema debe presentar la comanda y lista de empaque de forma 100% digital en pantalla, sin requerir impresión de tickets térmicos desde la aplicación web.

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
5. `EARS-DSP-05`: **Cuando** el operador verifica la validez del comprobante bancario o aprueba la orden en efectivo, el sistema debe cambiar el estado del pedido a `EN_PREPARACION` y registrar el usuario auditor.
6. `EARS-DSP-06`: **Cuando** el operador rechaza un pago por inconsistencia o fraude, el sistema debe exigir un motivo obligatorio de rechazo, revertir el estado a `RECHAZADO`, reponer inmediatamente el stock de los productos al inventario de la sede y mostrar los datos del cliente para contacto telefónico.
7. `EARS-DSP-07`: **Cuando** el paquete esté empaquetado para Retiro en Taquilla, el operador debe cambiar el estado a `LISTO_PARA_RETIRO`.
8. `EARS-DSP-08`: **Cuando** el paquete es entregado al repartidor motorizado, el operador debe cambiar el estado a `EN_CAMINO`.
9. `EARS-DSP-09`: **Cuando** el repartidor motorizado notifica telefónicamente o en persona la entrega exitosa (y liquida el efectivo en caja si aplica), el operador debe marcar el pedido como `ENTREGADO`.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
10. `EARS-DSP-10`: **Mientras** un pedido permanezca en estado `PENDIENTE_VERIFICACION`, el stock asociado debe mantenerse bloqueado impidiendo su venta a otros clientes.
11. `EARS-DSP-11`: **Mientras** un pedido contenga medicamentos de Venta Controlada, el tablero del operador debe destacar una insignia de alerta legal indicando que la entrega solo puede efectuarse previa retención/sellado del récipe médico físico original.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
12. `EARS-DSP-12`: **Si** un cliente no presenta el récipe médico original al momento de la entrega por delivery, **entonces** el motorizado debe devolver el medicamento a la farmacia, el operador debe cambiar el pedido a `CANCELADO_SIN_RECIPE`, reintegrar el costo de los productos reteniendo la tarifa de delivery por concepto de viaje realizado, y reponer el stock.
13. `EARS-DSP-13`: **Si** un cliente solicita anular su pedido, **entonces** la anulación debe ser gestionada exclusivamente por el operador de farmacia mediante llamada telefónica; el cliente no dispone de opción de cancelación unilateral desde la web una vez confirmada la orden.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/orders/
├── domain/
│   ├── events/
│   │   ├── OrderCreatedEvent.ts
│   │   ├── OrderVerifiedEvent.ts
│   │   ├── OrderRejectedEvent.ts
│   │   └── OrderDeliveredEvent.ts
│   └── services/
│       └── OrderStateMachine.ts
├── application/
│   ├── dtos/
│   │   ├── BranchOrdersQueryDTO.ts
│   │   ├── VerifyPaymentCommandDTO.ts
│   │   ├── RejectOrderCommandDTO.ts
│   │   └── TransitionOrderStatusDTO.ts
│   └── use-cases/
│       ├── GetBranchPendingOrdersUseCase.ts
│       ├── VerifyPaymentUseCase.ts
│       ├── RejectOrderWithStockRestorationUseCase.ts
│       └── UpdateOrderStatusUseCase.ts
├── infrastructure/
│   └── persistence/
│       └── TypeOrmOrderStateRepository.ts
└── presentation/
    ├── actions/
    │   ├── verifyPaymentAction.ts
    │   ├── rejectOrderAction.ts
    │   └── updateOrderStatusAction.ts
    └── components/
        ├── DispatchDashboard.tsx
        ├── OrderKanbanBoard.tsx
        ├── DigitalPackingModal.tsx
        ├── RejectOrderDialog.tsx
        └── ChimeAudioPlayer.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `VerifyPaymentUseCase`
* **Entrada:** `VerifyPaymentCommandDTO` (`orderId: string`, `operatorUserId: string`).
* **Lógica:**
  1. Cargar la orden y validar que su estado actual sea `PENDIENTE_VERIFICACION`.
  2. Aplicar transición mediante `OrderStateMachine` a `EN_PREPARACION`.
  3. Registrar `verified_by = operatorUserId`.
  4. Registrar la traza en `order_status_history`.
* **Salida:** `Result<void, AppError>`.

### 4.2 `RejectOrderWithStockRestorationUseCase`
* **Entrada:** `RejectOrderCommandDTO` (`orderId: string`, `reason: string`, `operatorUserId: string`).
* **Lógica (Transacción Atómica):**
  1. Cargar la orden con sus `order_items`.
  2. Validar que la orden esté en `PENDIENTE_VERIFICACION`.
  3. Cambiar estado a `RECHAZADO` registrando el motivo en `rejection_reason`.
  4. Por cada ítem, sumar la cantidad de vuelta a `branch_inventories` de la sede correspondiente (`stock = stock + quantity`).
  5. Registrar la traza en `order_status_history`.
* **Salida:** `Result<void, AppError>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `OrderStateMachine.spec.ts`:
  * Debe permitir transiciones válidas: `PENDIENTE` ➔ `EN_PREPARACION` ➔ `LISTO/EN_CAMINO` ➔ `ENTREGADO`.
  * Debe rechazar transiciones inválidas (ej. de `PENDIENTE` directamente a `ENTREGADO`).
* `RejectOrderWithStockRestorationUseCase.spec.ts`:
  * Al rechazar una orden con 2 unidades de un producto, el stock de la sede debe incrementarse exactamente en 2 unidades.
  * Debe requerir un motivo de rechazo no vacío.
