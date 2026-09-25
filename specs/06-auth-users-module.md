# Especificación de Requerimientos: Módulo de Usuarios, Autenticación y Control de Accesos

**Módulo:** `auth`  
**Bounded Context:** Autenticación JWT, Roles RBAC, Clientes e Invitados  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
El módulo de autenticación gestiona la identidad, seguridad y permisos para todos los actores de **Hospisalud**. Soporta un modelo dual de compra (Compra Rápida como Invitado sin contraseña y Cuenta Registrada con credenciales seguras), vinculación histórica automática de compras por Cédula, y control de acceso basado en roles (**RBAC**) para los cuatro perfiles del sistema: **Super Administrador**, **Administrador de Sede**, **Operador de Despacho** y **Cliente**.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-AUT-01`: El sistema debe emitir tokens JWT cifrados almacenados exclusivamente en cookies con directivas `httpOnly`, `Secure` y `SameSite=Lax`.
2. `EARS-AUT-02`: El sistema debe aplicar control de acceso basado en roles (RBAC) con 4 niveles de permisos:
   * **Super Administrador:** Acceso irrestricto a las 4 sedes, fijación de tasa BCV, alta/edición de catálogo y tablas de ventas consolidadas.
   * **Administrador de Sede:** Acceso exclusivo a su sede, ajuste de stock y tablas locales de ventas e incidencias.
   * **Operador de Despacho:** Tablero de despacho en vivo y verificación de pedidos de su sede asignada.
   * **Cliente:** Perfil, libreta de direcciones, historial de órdenes y seguimiento en vivo.
3. `EARS-AUT-03`: El sistema debe ofrecer formularios de autenticación, libreta de direcciones y tablas de pedidos 100% responsivas, con inputs optimizados para dispositivos móviles (tipo `tel` y `numeric` para Cédula y Teléfono, teclado `email` para correo).

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
3. `EARS-AUT-03`: **Cuando** un usuario compra en modalidad de **Invitado (Guest Checkout)**, el sistema debe registrar el pedido asociando su Cédula, Nombre y Teléfono sin exigir la creación de contraseña.
4. `EARS-AUT-04`: **Cuando** un usuario se registra formalmente en la plataforma con su Cédula de Identidad, el sistema debe buscar y vincular automáticamente a su nuevo ID de usuario todos los pedidos históricos realizados previamente como invitado con esa misma Cédula.
5. `EARS-AUT-05`: **Cuando** un cliente registrado presiona el botón "Repetir Pedido" (*Re-order*) en su historial de compras, el sistema debe cargar los productos en el carrito actual y revalidar el stock disponible en la sede activa.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
6. `EARS-AUT-06`: **Mientras** un usuario mantenga una sesión autenticada como Administrador de Sede u Operador de Despacho, el sistema debe restringir todas sus consultas y acciones exclusivamente a los registros pertenecientes a su `assigned_branch_id`.
7. `EARS-AUT-07`: **Mientras** un usuario actúe como Invitado, el sistema no debe exponer panel de seguimiento web ni libreta de direcciones, coordinándose el despacho exclusivamente vía telefónica.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
8. `EARS-AUT-08`: **Si** un usuario no autenticado o con rol insuficiente intenta acceder a rutas protegidas `/admin/*` o `/operaciones/*`, **entonces** el Middleware de Next.js debe interceptar la solicitud y redirigir a la pantalla de login correspondiente.
9. `EARS-AUT-09`: **Si** se ingresan credenciales incorrectas en el login, **entonces** el sistema debe retornar un mensaje de error genérico (*"Credenciales incorrectas"*) sin revelar si el usuario o contraseña específicos fallaron.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/auth/
├── domain/
│   ├── entities/
│   │   ├── User.ts
│   │   └── UserAddress.ts
│   ├── value-objects/
│   │   ├── CedulaRIF.ts
│   │   ├── Email.ts
│   │   ├── PasswordHash.ts
│   │   └── UserRole.ts
│   ├── repositories/
│   │   └── IUserRepository.ts
│   ├── services/
│   │   ├── PasswordHasherService.ts
│   │   └── TokenService.ts
│   └── errors/
│       ├── InvalidCredentialsError.ts
│       └── UnauthorizedAccessError.ts
├── application/
│   ├── dtos/
│   │   ├── RegisterUserDTO.ts
│   │   ├── LoginCredentialsDTO.ts
│   │   ├── AuthSessionDTO.ts
│   │   └── UserProfileDTO.ts
│   └── use-cases/
│       ├── RegisterUserUseCase.ts
│       ├── LoginUserUseCase.ts
│       ├── LinkGuestOrdersUseCase.ts
│       ├── GetUserProfileUseCase.ts
│       └── SaveUserAddressUseCase.ts
├── infrastructure/
│   ├── entities/
│   │   ├── UserEntity.ts
│   │   └── UserAddressEntity.ts
│   ├── mappers/
│   │   └── UserMapper.ts
│   ├── services/
│   │   ├── BcryptPasswordHasher.ts
│   │   └── JoseTokenService.ts
│   └── repositories/
│       └── TypeOrmUserRepository.ts
└── presentation/
    ├── actions/
    │   ├── loginAction.ts
    │   ├── registerAction.ts
    │   ├── logoutAction.ts
    │   └── saveAddressAction.ts
    ├── middleware/
    │   └── authMiddleware.ts
    └── components/
        ├── LoginForm.tsx
        ├── RegisterForm.tsx
        ├── SavedAddressesList.tsx
        └── OrderHistoryTable.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `RegisterUserUseCase`
* **Entrada:** `RegisterUserDTO` (`idDocument: string`, `fullName: string`, `email: string`, `phone: string`, `password: string`).
* **Lógica:**
  1. Validar formato de Cédula/RIF y correo electrónico.
  2. Verificar que no exista una cuenta activa con ese correo o documento.
  3. Hashear la contraseña con `BcryptPasswordHasher` (cost factor 10).
  4. Crear entidad `User` con rol `CLIENTE`.
  5. Guardar en base de datos.
  6. Invocar `LinkGuestOrdersUseCase` para actualizar pedidos anteriores con el mismo `id_document`.
  7. Generar token JWT de sesión e inyectar cookie HTTP-only.
* **Salida:** `Result<AuthSessionDTO, AppError>`.

### 4.2 `LinkGuestOrdersUseCase`
* **Entrada:** `userId: string`, `idDocument: string`.
* **Lógica:**
  1. Ejecutar sentencia UPDATE en base de datos sobre la tabla `orders`:
     `UPDATE orders SET customer_id = :userId WHERE customer_id_document = :idDocument AND customer_id IS NULL`.
  2. Retornar cantidad de pedidos vinculados.
* **Salida:** `Result<number, never>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `CedulaRIF.spec.ts`:
  * Debe aceptar formatos válidos venezolanos (ej. `V-12345678`, `J-123456789`, `E-87654321`).
  * Debe rechazar caracteres no válidos o formatos vacíos.
* `RegisterUserUseCase.spec.ts`:
  * Al registrarse con éxito, debe ejecutar la vinculación de pedidos históricos bajo esa Cédula.
  * No debe almacenar nunca contraseñas en texto plano.
