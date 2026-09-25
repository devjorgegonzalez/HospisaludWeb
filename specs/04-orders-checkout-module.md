# Especificación de Requerimientos: Módulo de Pedidos, Carrito y Checkout

**Módulo:** `orders`  
**Bounded Context:** Carrito de Compras, Cálculo de Delivery, Pasarela de Checkout y Creación de Pedidos  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
Este módulo orquesta la experiencia transaccional central de **Hospisalud**. Abarca la gestión del carrito de compras, la integración con la API de Google Maps para la fijación del pin geográfico de entrega, el cálculo automático de tarifas por zonas poligonales, la exoneración condicional de flete para compras $\ge$ $25 USD, el bloqueo atómico de inventario y el procesamiento de pagos mediante Pago Móvil, Binance Pay y Efectivo en Divisas.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-ORD-01`: El sistema debe exigir obligatoriamente para toda compra los siguientes datos del cliente: Cédula de Identidad o RIF, Nombre y Apellido, y Número de Teléfono móvil.
2. `EARS-ORD-02`: El sistema debe calcular el costo de delivery en base a las siguientes zonas poligonales:
   * **Zona Urbana (Cercana):** $2.00 USD
   * **Zona Interurbana (Media distancia):** $3.00 USD
   * **Zona Urbana (Muy retirada):** $4.00 USD
   * **San Tomé:** $5.00 USD (tarifa especial)
3. `EARS-ORD-03`: El sistema debe descontar/bloquear el inventario de la sede de forma atómica en el instante exacto en que el usuario presiona **"Confirmar Pedido"**, sin reservas previas mientras navega.

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
4. `EARS-ORD-04`: **Cuando** el usuario coloca o desplaza el pin en el mapa interactivo de Google Maps durante el checkout, el sistema debe calcular la zona poligonal correspondiente y actualizar en tiempo real la tarifa de delivery en el desglose de precios.
5. `EARS-ORD-05`: **Cuando** el subtotal de productos de una orden sea igual o mayor a **$25.00 USD**, el sistema debe aplicar automáticamente un costo de delivery de **$0.00 USD**, siempre que el destino corresponda a zonas urbanas locales ($2, $3 y $4 USD) de El Tigre o Guanipa.
6. `EARS-ORD-06`: **Cuando** la entrega sea en San Tomé ($5.00 USD) o constituya un despacho interurbano cruzado El Tigre ➔ Guanipa ($5.00 USD), el sistema debe mantener íntegra la tarifa de **$5.00 USD**, independientemente de si el subtotal supera los $25.00 USD.
7. `EARS-ORD-07`: **Cuando** el cliente selecciona pago en Efectivo Divisas para Delivery, el sistema debe exigir la selección de la denominación del billete a entregar ($10, $20, $50, $100) o indicar "Monto Exacto", calculando el vuelto a entregar en USD y mostrando la advertencia legal de billetes en buen estado físico.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
8. `EARS-ORD-08`: **Mientras** el carrito contenga al menos un producto clasificado como `VENTA_CONTROLADA`, el sistema debe requerir que el usuario marque una casilla obligatoria de confirmación: *"Acepto presentar el récipe médico original y cédula de identidad física al momento de la entrega; entiendo que de no presentarlos el medicamento no será entregado y la tarifa de delivery no es reembolsable"*.
9. `EARS-ORD-09`: **Mientras** no se haya presionado el botón "Confirmar Pedido", el sistema no debe bloquear ni retener unidades en el inventario de la base de datos.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
10. `EARS-ORD-10`: **Si** al presionar "Confirmar Pedido" otro usuario compró las últimas unidades y no hay stock suficiente en la sede, **entonces** el sistema debe cancelar la transacción, restaurar el carrito y notificar al cliente del cambio de stock.
11. `EARS-ORD-11`: **Si** el usuario no acepta la cláusula de récipe para medicamentos de venta controlada, **entonces** el botón de pago debe permanecer bloqueado.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/orders/
├── domain/
│   ├── entities/
│   │   ├── Order.ts
│   │   ├── OrderItem.ts
│   │   ├── Cart.ts
│   │   └── CartItem.ts
│   ├── value-objects/
│   │   ├── OrderNumber.ts
│   │   ├── OrderStatus.ts
│   │   ├── PaymentDetails.ts
│   │   ├── DeliveryAddress.ts
│   │   └── DeliveryFee.ts
│   ├── repositories/
│   │   ├── IOrderRepository.ts
│   │   └── IDeliveryZoneRepository.ts
│   ├── services/
│   │   ├── DeliveryCalculatorService.ts
│   │   └── OrderRoutingService.ts
│   └── errors/
│       ├── InsufficientStockAtCheckoutError.ts
│       └── ControlledMedAgreementRequiredError.ts
├── application/
│   ├── dtos/
│   │   ├── CreateOrderCommandDTO.ts
│   │   ├── OrderSummaryResponseDTO.ts
│   │   ├── CalculateDeliveryQueryDTO.ts
│   │   └── DeliveryCalculationResultDTO.ts
│   └── use-cases/
│       ├── CalculateDeliveryFeeUseCase.ts
│       ├── CreateOrderUseCase.ts
│       ├── GetOrderByIdUseCase.ts
│       └── ReorderPreviousOrderUseCase.ts
├── infrastructure/
│   ├── entities/
│   │   ├── OrderEntity.ts
│   │   ├── OrderItemEntity.ts
│   │   ├── DeliveryZoneEntity.ts
│   │   └── OrderStatusHistoryEntity.ts
│   ├── mappers/
│   │   └── OrderMapper.ts
│   ├── services/
│   │   └── GoogleMapsPolygonService.ts
│   └── repositories/
│       ├── TypeOrmOrderRepository.ts
│       └── TypeOrmDeliveryZoneRepository.ts
└── presentation/
    ├── actions/
    │   ├── calculateDeliveryAction.ts
    │   └── submitOrderAction.ts
    ├── hooks/
    │   └── useCart.ts
    └── components/
        ├── CartDrawer.tsx
        ├── CheckoutForm.tsx
        ├── InteractiveGoogleMap.tsx
        ├── PaymentMethodSelector.tsx
        ├── ControlledMedNotice.tsx
        └── OrderSuccessCard.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `CalculateDeliveryFeeUseCase`
* **Entrada:** `CalculateDeliveryQueryDTO` (`latitude: number`, `longitude: number`, `subtotalUSD: number`, `originBranchId: string`).
* **Lógica:**
  1. Utilizar `GoogleMapsPolygonService` para identificar en qué zona poligonal cae el punto geográfico.
  2. Determinar si es San Tomé o despacho interurbano El Tigre ➔ Guanipa ($5.00 USD fijos).
  3. Si la zona es urbana local ($2, $3, $4 USD) y `subtotalUSD >= 25.00`, asignar tarifa $0.00 USD.
  4. De lo contrario, asignar la tarifa base de la zona.
* **Salida:** `Result<DeliveryCalculationResultDTO, AppError>`.

### 4.2 `CreateOrderUseCase`
* **Entrada:** `CreateOrderCommandDTO`.
* **Lógica (Transacción Atómica en Base de Datos):**
  1. Verificar si la hora se encuentra en la ventana operativa (8:00 AM a 9:00 PM).
  2. Obtener la tasa BCV oficial del día.
  3. Bloquear pesimistamente las filas de inventario (`SELECT ... FOR UPDATE`) en `branch_inventories` para todos los productos de la orden.
  4. Verificar que `stock >= cantidad_solicitada` para cada producto.
  5. Descontar las cantidades del inventario.
  6. Calcular y congelar importes en USD y VES.
  7. Insertar el registro en `orders`, los registros en `order_items` y la traza en `order_status_history` con estado inicial `PENDIENTE_VERIFICACION`.
* **Salida:** `Result<OrderSummaryResponseDTO, AppError>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `DeliveryCalculatorService.spec.ts`:
  * Subtotal $24.99 en Zona Urbana 1 ($2) -> Tarifa $2.00 USD.
  * Subtotal $25.00 en Zona Urbana 1 ($2) -> Tarifa $0.00 USD.
  * Subtotal $100.00 en San Tomé -> Tarifa $5.00 USD (sin exoneración).
  * Subtotal $50.00 en despacho interurbano El Tigre ➔ Guanipa -> Tarifa $5.00 USD.
* `CreateOrderUseCase.spec.ts`:
  * Debe rechazar la orden si algún producto tiene stock insuficiente al confirmar.
  * Debe generar un código de orden con formato válido (`HOSP-YYYYMMDD-XXXX`).
  * Debe congelar la tasa BCV y los montos calculados de forma inmutable.
