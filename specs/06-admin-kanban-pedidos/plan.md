# Plan: Panel Administrativo, Roles y Tablero Kanban de Pedidos

**Módulo:** 06 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/06-admin-kanban-pedidos/spec.md)

---

## 1. Enfoque Arquitectónico
- **Tablero Kanban con Drag-and-Drop / Botones de Acción Rápida:** Implementación con `@hello-pangea/dnd` o botones directos de transición de estado para agilizar la operación en pantallas táctiles o móviles de repartidores.
- **Mecanismo de Tiempo Real:** Polling reactivo cada 10 segundos con `useSWR` o `React Query`, con hook de audio nativo Web Audio API (`new Audio('/sounds/notification.mp3').play()`) ante nuevos pedidos detectados.
- **Buzón Guanipa:** Tab o vista filtrada con badge en rojo indicando órdenes en espera de aprobación de El Tigre.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/admin/types.ts`: Tipos del Kanban, columnas y filtros operativos.
- `src/actions/admin-order-actions.ts`: Server Actions para transiciones de estado, auditoría y aprobación Guanipa.
- `src/app/admin/kanban/page.tsx`: Vista principal del tablero Kanban.
- `src/app/admin/guanipa-approvals/page.tsx`: Buzón exclusivo de órdenes inter-sedes Guanipa.
- `src/components/admin/KanbanBoard.tsx`: Contenedor del tablero con columnas.
- `src/components/admin/KanbanCard.tsx`: Tarjeta con número de orden, cliente, total USD/VES, método de pago y referencia.
- `src/components/admin/OrderVerificationModal.tsx`: Modal para cotejar referencia bancaria, billete de efectivo y receta física.
- `src/hooks/useRealtimeOrders.ts`: Hook con polling de 10s y reproducción de sonido.

---

## 3. Dependencias Nuevas & Componentes shadcn/ui
- **Componentes shadcn/ui requeridos:** `Card`, `Badge`, `Dialog`, `Button`, `ScrollArea`, `Tabs`, `Sonner`.
- `swr`: Para revalidación periódica ligera en segundo plano en el tablero.

---

## 4. Decisiones Técnicas
- **Auditoría Estricta:** Toda transición genera una fila en `order_status_logs` asociando el ID del usuario autenticado que autorizó el cambio.
