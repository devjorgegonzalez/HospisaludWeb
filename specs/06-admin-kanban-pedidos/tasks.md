# Tasks: Panel Administrativo, Roles y Tablero Kanban de Pedidos

**Módulo:** 06 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/06-admin-kanban-pedidos/spec.md)

---

- [ ] **Task 06.1:** Definir tipos del tablero Kanban en `src/domain/admin/types.ts`.
- [ ] **Task 06.2:** Implementar Server Action `getKanbanOrders` con filtrado por rol y sede del operador (satisface AC-06.1).
- [ ] **Task 06.3:** Implementar Server Action `updateOrderStatusAction` con registro en `order_status_logs` y descuento de stock al verificar pago.
- [ ] **Task 06.4:** Implementar Server Action `approveGuanipaOrderAction` para resolver pedidos en espera hacia Guanipa con tarifa de $5.
- [ ] **Task 06.5:** Construir hook `useRealtimeOrders.ts` con polling y disparador de sonido para alertas internas en la web.
- [ ] **Task 06.6:** Construir componente `KanbanCard.tsx` con badges de método de pago, referencia y estado de receta.
- [ ] **Task 06.7:** Construir modal `OrderVerificationModal.tsx` para verificación de referencias y recetas.
- [ ] **Task 06.8:** Ensamblar vista del tablero en `src/app/admin/kanban/page.tsx`.
- [ ] **Task 06.9:** Construir vista de buzón de aprobaciones en `src/app/admin/guanipa-approvals/page.tsx`.
- [ ] **Task 06.10:** Escribir tests de integración para las transiciones de estado de pedidos y la auditoría en logs.
