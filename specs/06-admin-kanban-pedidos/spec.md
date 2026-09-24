# Spec: Panel Administrativo, Roles y Tablero Kanban de Pedidos

**Módulo:** 06 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
El personal operativo de cada farmacia física y los administradores generales necesitan una herramienta visual en tiempo real para gestionar el flujo de vida de los pedidos, verificar comprobantes de pago (Pago Móvil, Binance Pay, Efectivo), auditar recetas médicas para venta controlada, y gestionar aprobaciones inter-sedes (caso especial Guanipa), todo dentro de la propia plataforma web sin dependencias externas como WhatsApp.

---

## 2. Historias de Usuario
- **US-06.1:** Como Operador de Despacho, quiero ver un tablero tipo Kanban con columnas por estado para mover y organizar las órdenes entrantes.
- **US-06.2:** Como Operador de Despacho, quiero recibir una alerta sonora/visual en la web cuando entre un nuevo pedido en estado `PENDIENTE_VERIFICACION`.
- **US-06.3:** Como Operador de El Tigre, quiero tener un buzón de pedidos en estado `EN_ESPERA_GUANIPA` para aprobar o rechazar manualmente el envío cruzado con tarifa de $5 USD hacia Guanipa.
- **US-06.4:** Como Administrador de Sede, quiero ver únicamente los pedidos e inventarios pertenecientes a mi sucursal asignada.
- **US-06.5:** Como Super Administrador, quiero tener visibilidad y control global de las 4 sedes de forma consolidada o filtrada.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-06.1:** El sistema deberá restringir la visualización de órdenes en el tablero Kanban a la sede asignada al operador/administrador de sede (a excepción del `SUPER_ADMIN`, quien puede alternar o ver todas).
- **AC-06.2:** El sistema deberá registrar un log inmutable en `order_status_logs` por cada transición de estado, indicando quién realizó el cambio y la fecha/hora exacta.

### Dirigido por Evento
- **CUANDO** el operador verifica la referencia de Pago Móvil o Binance Pay y hace clic en "Verificar y Preparar", el sistema deberá mover la orden al estado `EN_PREPARACION` y descontar formalmente las unidades de `branch_inventories`.
- **CUANDO** un operador de El Tigre hace clic en "Aprobar Despacho Guanipa" en una orden `EN_ESPERA_GUANIPA`, el sistema deberá pasar la orden a `EN_PREPARACION`, fijar `delivery_fee_usd = 5.00` y asignar formalmente la sede de El Tigre como sucursal despachadora.
- **CUANDO** entra una nueva orden en el sistema, el tablero Kanban deberá actualizarse en tiempo real (vía polling corto o SSE/WebSockets) y emitir un tono de notificación discreto en el navegador del operador.
- **CUANDO** el motorizado o cajero entrega el pedido y lo marca como `ENTREGADO`, el sistema deberá cerrar el ciclo de la orden.

### Comportamiento no deseado
- **SI** un pedido tiene productos de Venta Controlada y el operador no ha marcado la casilla "Receta física verificada", **ENTONCES** el sistema deberá bloquear la transición a `ENTREGADO`.

---

## 4. Modelo de Datos Relevante
- `orders`: `id`, `order_number`, `branch_id`, `delivery_type`, `order_status`, `payment_method`, `payment_reference`, `cash_denomination_usd`, `cash_change_usd`, `is_prescription_verified`, `approved_by`.
- `order_status_logs`: `id`, `order_id`, `previous_status`, `new_status`, `changed_by`, `notes`.
- `users`: `role` (`SUPER_ADMIN`, `BRANCH_ADMIN`, `OPERATOR`).

---

## 5. Contratos de Server Actions
- `getKanbanOrders(branchId?: string, statusFilter?: OrderStatus)`: Lista pedidos agrupados por columna.
- `updateOrderStatusAction(orderId: string, newStatus: OrderStatus, notes?: string)`: Realiza la transición con auditoría.
- `approveGuanipaOrderAction(orderId: string, elTigreBranchId: string)`: Aprueba despacho especial.
- `verifyPrescriptionAction(orderId: string, isVerified: boolean)`.

---

## 6. Casos Límite
- Dos operadores intentan verificar la misma orden al mismo tiempo: Bloqueo optimista con verificación de versión o estado previo.
- Rechazo del despacho a Guanipa por falta de acuerdo con el cliente: Pasar la orden a estado `CANCELADO` y liberar la reserva de stock.

---

## 7. Fuera de Alcance
- Integraciones con WhatsApp API (las notificaciones operan 100% web).
