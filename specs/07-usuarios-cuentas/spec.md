# Spec: Gestión de Cuentas, Usuarios y Libreta de Direcciones

**Módulo:** 07 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
Los clientes recurrentes necesitan consultar su historial de compras para repetir pedidos frecuentes de tratamientos continuos (re-order con un clic) y gestionar una libreta de direcciones habituales (Casa, Oficina, Familiares). Además, el sistema requiere una arquitectura de usuarios limpia y desacoplada, ya que el método definitivo de autenticación está marcado como 📌 **[PENDIENTE DEFINICIÓN]**.

---

## 2. Historias de Usuario
- **US-07.1:** Como cliente registrado, quiero consultar mi historial de compras pasadas con sus estados detallados.
- **US-07.2:** Como cliente con tratamiento recurrente, quiero presionar el botón "Repetir Pedido" para cargar automáticamente los mismos medicamentos en mi carrito actual.
- **US-07.3:** Como cliente registrado, quiero guardar múltiples direcciones en mi libreta personal y seleccionarlas con un clic durante el checkout.
- **US-07.4:** Como cliente invitado, quiero poder registrarme posteriormente utilizando mi correo y vincular mis pedidos históricos si coinciden con mi cédula o teléfono.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-07.1:** El sistema deberá mantener un diseño modular y agnóstico para la autenticación mediante la interfaz abstracta `AuthAdapter`, facilitando la futura conexión con NextAuth, Supabase Auth o JWT sin alterar el dominio.

### Dirigido por Evento
- **CUANDO** un cliente registrado hace clic en "Repetir Pedido" en su historial, el sistema deberá verificar la disponibilidad de stock en su sede activa y añadir los productos disponibles al carrito actual.
- **CUANDO** un cliente registrado agrega una nueva dirección a su libreta, el sistema deberá guardarla asociando la zona correspondiente (`URBANA` o `INTERURBANA`).
- **CUANDO** el cliente llega al checkout estando autenticado, el sistema deberá prellenar automáticamente sus datos personales y permitir seleccionar una dirección de su libreta.

### Comportamiento no deseado
- **SI** al intentar "Repetir Pedido" uno de los productos originales está agotado en la sede actual, **ENTONCES** el sistema deberá agregar los productos que sí tengan stock y emitir una advertencia indicando qué ítems no pudieron incluirse.

---

## 4. Modelo de Datos Relevante
- `users`: `id`, `role`, `full_name`, `cedula`, `phone`, `email`, `password_hash`, `is_active`.
- `user_addresses`: `id`, `user_id`, `title`, `address_line`, `zone_type`, `latitude`, `longitude`, `reference_notes`.
- `orders`: `user_id`, `order_number`, `created_at`, `total_usd`.

---

## 5. Contratos de Server Actions / Puertos de Autenticación
- `interface AuthAdapter`:
  - `getCurrentUser(): Promise<UserSession | null>`
  - `signIn(credentials: LoginDTO): Promise<AuthResult>`
  - `signOut(): Promise<void>`
- `getUserOrders(userId: string)`: Devuelve historial de órdenes.
- `reorderAction(orderId: string, currentBranchId: string)`: Devuelve `{ addedItems: CartItem[], unavailableItems: string[] }`.
- `saveAddressAction(addressData: CreateAddressDTO)`: Registra dirección en la libreta.
- `deleteAddressAction(addressId: string)`.

---

## 6. Casos Límite
- Usuario compra como invitado y luego se crea cuenta: Script o acción para asociar órdenes históricas buscando por coincidencia de cédula y teléfono.

---

## 7. Fuera de Alcance
- Integración final de proveedores específicos de autenticación (se deja la capa de abstracción lista y un mock de sesión funcional para desarrollo).
