# Tasks: Gestión de Cuentas, Usuarios y Libreta de Direcciones

**Módulo:** 07 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/07-usuarios-cuentas/spec.md)

---

- [ ] **Task 07.1:** Definir puerto `AuthAdapter` y tipos de sesión en `src/domain/auth/ports.ts` (satisface AC-07.1).
- [ ] **Task 07.2:** Implementar adaptador provisional `cookie-mock-auth.ts` para pruebas locales con roles.
- [ ] **Task 07.3:** Implementar Server Actions para libreta de direcciones en `src/actions/account-actions.ts` (`saveAddressAction`, `deleteAddressAction`).
- [ ] **Task 07.4:** Implementar Server Action `reorderAction` para verificación y carga de medicamentos al carrito (satisface EARS evento re-order).
- [ ] **Task 07.5:** Construir interfaz de libreta de direcciones en `src/app/account/addresses/page.tsx`.
- [ ] **Task 07.6:** Construir vista de historial de pedidos en `src/app/account/orders/page.tsx` con componente `ReorderButton.tsx`.
- [ ] **Task 07.7:** Escribir tests unitarios para la lógica de re-order ante productos parcialmente agotados.
