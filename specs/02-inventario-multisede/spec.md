# Spec: Inventario Multisede y Reserva Temporal de Stock

**Módulo:** 02 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
Hospisalud cuenta con 4 sedes físicas con inventarios independientes. Si un cliente compra en la sede Centro o Hospital, el stock no debe cruzarse indebidamente. Asimismo, durante el proceso de pago se debe garantizar que el producto no sea adquirido por otro usuario en esos minutos críticos (reserva temporal de 15 min), y si un producto se agota en la sede actual pero existe en otra, la plataforma debe informar la disponibilidad cruzada para guiar al usuario.

---

## 2. Historias de Usuario
- **US-02.1:** Como cliente, quiero ver el stock real del producto en la sede activa seleccionada.
- **US-02.2:** Como cliente, si un producto está agotado en mi sede actual, quiero ver un aviso claro indicando en qué otras sedes sí está disponible.
- **US-02.3:** Como cliente, al entrar al checkout para pagar, quiero que mis productos queden reservados por 15 minutos para asegurar mi compra.
- **US-02.4:** Como administrador de sede, quiero ajustar y actualizar manualmente el stock de mi sucursal sin afectar a las demás sucursales.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-02.1:** El sistema deberá calcular el stock disponible para venta como: `stock_efectivo = stock_quantity - reservas_activas`.
- **AC-02.2:** El sistema deberá mantener registros de stock completamente aislados por cada una de las 4 sedes (`HOSPITAL`, `CENTRO`, `PETRUCCI`, `GUANIPA`).

### Dirigido por Evento
- **CUANDO** el cliente ingresa a la pantalla de pago (Checkout), el sistema deberá crear un registro en `stock_reservations` con vigencia exacta de 15 minutos (`expires_at = NOW() + 15 minutes`).
- **CUANDO** el operador confirma la verificación del pago (o el pedido en efectivo), el sistema deberá descontar formalmente las unidades de `branch_inventories` y marcar la reserva como `COMMITTED`.
- **CUANDO** expira el plazo de 15 minutos sin confirmación de orden, el sistema deberá marcar la reserva como `RELEASED` y reintegrar el stock a la disponibilidad pública.

### Dirigido por Estado
- **MIENTRAS** el stock de un producto sea igual a `0` en la sede seleccionada pero mayor a `0` en otra sede, el sistema deberá mostrar la etiqueta: *"Agotado en esta sede. Disponible en [Nombre de Sede X]"*.

### Comportamiento no deseado
- **SI** un usuario intenta agregar al carrito una cantidad superior al stock disponible (`stock_efectivo`), **ENTONCES** el sistema deberá rechazar la acción con el mensaje: *"Solo quedan X unidades disponibles en esta sede"*.

---

## 4. Modelo de Datos Relevante
- `branch_inventories`: `id`, `branch_id`, `product_id`, `stock_quantity`, `updated_at`.
- `stock_reservations`: `id`, `branch_id`, `product_id`, `session_or_user_id`, `quantity`, `expires_at`, `status` (`ACTIVE`, `RELEASED`, `COMMITTED`).
- `branches`: `id`, `code`, `name`.

---

## 5. Contratos de Server Actions / Cronjobs
- `checkProductAvailability(productId: string, currentBranchId: string)`: Devuelve `{ currentStock: number, otherBranches: { branchName: string, stock: number }[] }`.
- `createStockReservation(sessionId: string, items: { productId: string, branchId: string, quantity: number }[])`: Retorna `{ success: boolean, expiresAt: Date, reservationId: string }`.
- `releaseExpiredReservationsCron()`: Tarea de limpieza periódica ejecutada cada 1-2 minutos para liberar reservas donde `expires_at < NOW() AND status = 'ACTIVE'`.
- `updateBranchStock(branchId: string, productId: string, newQuantity: number)`: Solo ejecutable por `BRANCH_ADMIN` o `SUPER_ADMIN`.

---

## 6. Casos Límite
- Concurrencia al reservar la última unidad: Manejo con transacción PostgreSQL `SELECT ... FOR UPDATE` para evitar condiciones de carrera (*race conditions*).
- El usuario abandona el checkout antes de los 15 minutos: El cronjob libera automáticamente las unidades sin requerir acción del usuario.

---

## 7. Fuera de Alcance
- Traspasos automáticos de stock entre sedes físicas sin orden de compra.
