# Plan: Inventario Multisede y Reserva Temporal de Stock

**Módulo:** 02 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/02-inventario-multisede/spec.md)

---

## 1. Enfoque Arquitectónico
- **Atomicidad Transaccional:** El bloqueo de inventario en `createStockReservation` utilizará transacciones PostgreSQL con `SERIALIZABLE` o `SELECT ... FOR UPDATE` sobre la fila correspondiente de `branch_inventories` para garantizar que dos usuarios no reserven la misma última unidad.
- **Worker / Cron de Liberación:**
  - En desarrollo/Next.js: Endpoint interno `/api/cron/release-reservations` protegido por `CRON_SECRET`, invocable vía Vercel Cron, GitHub Actions, o llamada programada.
  - Como fallback pasivo: Cada consulta de cálculo de stock disponible incluye automáticamente la condición `AND (expires_at > NOW() OR status != 'ACTIVE')`, garantizando que una reserva expirada deje de bloquear stock de inmediato, incluso antes de que corra el cron de actualización.
- **Cross-branch Availability Hook:** Un hook en frontend `useCrossAvailability(productId, currentBranchId)` para renderizar alertas dinámicas en la ficha del producto.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/inventory/types.ts`: Tipos `BranchInventory`, `StockReservation`, `AvailabilityResult`.
- `src/domain/inventory/inventory-service.ts`: Lógica pura de cálculo de stock disponible y evaluación de reservas.
- `src/actions/inventory-actions.ts`: Server Actions para consultar stock cruzado, reservar unidades y ajustar inventario manual.
- `src/app/api/cron/release-reservations/route.ts`: Route Handler de Next.js para ejecutar la liberación periódica de reservas vencidas.
- `src/components/storefront/CrossAvailabilityNotice.tsx`: Badge visual que alerta si está agotado en la sede actual pero disponible en otra.
- `src/components/admin/StockTable.tsx`: Tabla de inventario para Administradores de Sede con edición inline de cantidades.

---

## 3. Dependencias Nuevas
- `date-fns`: Manipulación precisa de fechas y cálculo de tiempo restante de reserva de 15 minutos (`formatDistanceToNow`).

---

## 4. Decisiones Técnicas
- **Identificador de Sesión:** Para usuarios no registrados (guests), la reserva se asocia a un UUID generado en la cookie de sesión del carrito (`x-cart-session-id`).
- **Reserva no destructiva:** `branch_inventories.stock_quantity` no se modifica durante la reserva; solo se descuenta de forma permanente cuando el pedido es formalizado o verificado.
