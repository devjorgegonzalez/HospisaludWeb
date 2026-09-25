# Plan: Gestión de Cuentas, Usuarios y Libreta de Direcciones

**Módulo:** 07 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/07-usuarios-cuentas/spec.md)

---

## 1. Enfoque Arquitectónico
- **Patrón Adaptador para Autenticación:** Se definirá la interfaz `src/domain/auth/ports.ts` con una implementación mock en memoria / cookies simples para la fase de desarrollo (`src/adapters/auth/cookie-mock-auth.ts`). Esto permite desarrollar y probar el panel de usuario, libretas de direcciones y roles de empleados sin bloquearse por la decisión final del proveedor de Auth.
- **Acción Re-order (Tratamientos Continuos):** Algoritmo que lee los `order_items` de la orden previa, consulta `branch_inventories` de la sede activa del usuario, y retorna los ítems listos para despachar al carrito.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/auth/ports.ts`: Interfaz agnóstica `AuthAdapter` y tipos de sesión.
- `src/domain/account/types.ts`: Tipos `UserAddress`, `OrderHistorySummary`.
- `src/adapters/auth/cookie-mock-auth.ts`: Adaptador temporal para testing de roles y sesiones.
- `src/actions/account-actions.ts`: Server Actions para libreta de direcciones y re-order.
- `src/app/account/orders/page.tsx`: Vista de historial de compras con botón "Repetir Pedido".
- `src/app/account/addresses/page.tsx`: Vista para añadir y eliminar direcciones guardadas.
- `src/components/account/AddressCard.tsx`: Tarjeta visual con badge de zona Urbana/Interurbana.
- `src/components/account/ReorderButton.tsx`: Botón interactivo que añade productos al carrito.

---

## 3. Dependencias Nuevas & Componentes shadcn/ui
- **Componentes shadcn/ui requeridos:** `Card`, `Button`, `Table`, `Dialog`, `Input`, `Badge`, `Form`.
- Ninguna dependencia adicional pesada requerida en esta fase.

---

## 4. Decisiones Técnicas
- **Aislamiento Total de Auth:** El código de negocio consumirá únicamente `getCurrentUser()`. Cuando el cliente decida el proveedor definitivo (NextAuth, Supabase o Clerk), solo se reemplazará la clase adaptadora sin tocar componentes ni Server Actions.
