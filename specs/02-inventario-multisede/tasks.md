# Tasks: Inventario Multisede y Reserva Temporal de Stock

**Módulo:** 02 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/02-inventario-multisede/spec.md)

---

- [ ] **Task 02.1:** Definir tipos de inventario y reservas en `src/domain/inventory/types.ts`.
- [ ] **Task 02.2:** Implementar función de cálculo de stock disponible en `inventory-service.ts` considerando stock físico menos reservas activas no expiradas (satisface AC-02.1).
- [ ] **Task 02.3:** Crear Server Action `checkProductAvailability` para consultar disponibilidad en la sede actual y disponibilidad cruzada en las 3 sedes restantes.
- [ ] **Task 02.4:** Crear componente `CrossAvailabilityNotice.tsx` para mostrar "Agotado en esta sede. Disponible en [Sede X]".
- [ ] **Task 02.5:** Implementar Server Action `createStockReservation` con transacción PostgreSQL bloqueante (`FOR UPDATE`) para reservar stock por 15 minutos en checkout.
- [ ] **Task 02.6:** Implementar Route Handler `/api/cron/release-reservations` para marcar como `RELEASED` reservas con `expires_at < NOW()`.
- [ ] **Task 02.7:** Implementar Server Action `updateBranchStock` para ajuste manual de stock por parte de administradores de sede (con control RBAC).
- [ ] **Task 02.8:** Crear interfaz de administración `StockTable.tsx` para edición rápida de cantidades por sucursal.
- [ ] **Task 02.9:** Escribir tests de concurrencia simulando dos reservas simultáneas sobre la última unidad de stock.
