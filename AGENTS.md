# AGENTS.md - Directrices de Desarrollo para Agentes Autónomos

Este documento contiene las reglas de operación, directrices de arquitectura y estándares de codificación obligatorios para cualquier agente de inteligencia artificial (o subagente) que trabaje en el repositorio **HospisaludWeb**.

---

## 1. Principios Operativos Inquebrantables

1. **Alineación con GEMINI.md:** Antes de escribir una sola línea de código, el agente debe verificar que la implementación cumpla con las reglas de negocio descritas en [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md). Queda prohibido asumir o inventar reglas de negocio no acordadas.
2. **Documentación Obligatoria en Castellano:** Todo método, clase, interfaz, función o tipo debe estar documentado con **JSDoc/TSDoc en castellano**. El código debe explicar claramente qué hace, qué recibe y qué retorna.
3. **Domain-Driven Design (DDD) Estricto:**
   * La lógica de negocio reside únicamente en `domain/` y `application/`.
   * La capa de presentación (Server Actions, componentes de UI) **nunca** debe acceder directamente a la base de datos ni a entidades de TypeORM. Siempre debe interactuar mediante **Casos de Uso** (*Use Cases*).
   * La capa de dominio es agnóstica a TypeORM, Next.js o librerías externas.
4. **Patrón de Resultado (Result Pattern):**
   * No usar `throw new Error(...)` para casos de negocio previsibles (ej. falta de stock, credenciales incorrectas, producto no encontrado).
   * Retornar siempre objetos del tipo `Result<T, E>` utilizando `Result.ok(valor)` o `Result.fail(error)`.
5. **Base de Datos y Persistencia:**
   * `synchronize: false` está activo de forma estricta. Todo cambio en tablas o columnas debe realizarse mediante **migraciones formales de TypeORM**.
6. **Desarrollo Guiado por Especificaciones (Spec-Driven Development):**
   * Antes de implementar un módulo complejo, se debe redactar o consultar su especificación en formato Kiro / EARS en la carpeta `specs/`.

---

## 2. Sintaxis EARS para Requerimientos y Casos de Uso

Cuando un agente redacte especificaciones o documente casos de uso, debe emplear las cinco variantes estándar de **EARS** (*Easy Approach to Requirements Syntax*):

1. **Ubicua (Ubiquitous):**
   * *Patrón:* El sistema debe `<acción>`.
   * *Ejemplo:* El sistema debe mantener los precios base de todos los productos expresados en USD.
2. **Impulsada por Eventos (Event-driven):**
   * *Patrón:* **Cuando** `<evento>`, el sistema debe `<acción>`.
   * *Ejemplo:* **Cuando** el usuario confirma el pedido, el sistema debe bloquear el stock de los productos seleccionados y asignar el estado 'Pendiente por Verificación'.
3. **Impulsada por Estados (State-driven):**
   * *Patrón:* **Mientras** `<estado>`, el sistema debe `<acción>`.
   * *Ejemplo:* **Mientras** la plataforma esté fuera del horario operativo (9:01 PM a 7:59 AM), el sistema debe deshabilitar los botones de compra y operar en modo consulta de catálogo.
4. **Comportamiento No Deseado / Errores (Unwanted behavior):**
   * *Patrón:* **Si** `<condición de error>`, **entonces** el sistema debe `<acción>`.
   * *Ejemplo:* **Si** el usuario cambia de sede y el carrito contiene productos sin stock en la nueva sede, **entonces** el sistema debe resaltar los ítems en rojo y bloquear el botón de pago.
5. **Opcional (Optional features):**
   * *Patrón:* **Donde** `<característica esté presente>`, el sistema debe `<acción>`.
   * *Ejemplo:* **Donde** un producto sea de Venta Controlada, el sistema debe desplegar la advertencia legal y exigir la confirmación de presentación de récipe físico original.

---

## 3. Estructura de Directorios (Monolito Modular DDD)

Todo el código fuente debe ubicarse bajo `src/` respetando los Bounded Contexts:

```text
src/
├── app/                              # Next.js App Router (Páginas, layouts, rutas de API)
├── components/                       # Componentes compartidos de Shadcn UI y elementos globales
│   └── ui/                           # Primitivos de Shadcn
├── modules/
│   ├── catalog/                      # Catálogo, categorías, principios activos, inventario
│   │   ├── domain/                   # Entidades (Product, Category), Value Objects, IProductRepository
│   │   ├── application/              # Casos de uso (SearchProductsUseCase, AdjustStockUseCase), DTOs
│   │   ├── infrastructure/           # TypeORM Entities, TypeOrmProductRepository, StorageClient
│   │   └── presentation/             # Server Actions, componentes específicos del catálogo
│   ├── orders/                       # Carrito, pedidos, pagos, cálculo de delivery
│   │   ├── domain/                   # Order, OrderItem, DeliveryZone, IOrderRepository
│   │   ├── application/              # CreateOrderUseCase, VerifyPaymentUseCase, DTOs
│   │   ├── infrastructure/           # TypeORM Entities, GoogleMapsClient, PollingService
│   │   └── presentation/             # CheckoutForm, OperatorDashboard, CartWidget
│   ├── branches/                     # Sucursales, horarios, cuentas bancarias
│   ├── currency/                     # Tasa BCV, conversiones USD/VES
│   └── auth/                         # Autenticación JWT, roles RBAC, clientes e invitados
└── shared/                           # Núcleo compartido entre módulos
    ├── domain/                       # Result<T, E>, AppError, Value Objects universales
    └── infrastructure/               # DataSource de TypeORM, configuración de Azurite/Azure
```

---

## 4. Reglas Específicas por Capa

### 4.1 Capa de Dominio (`domain/`)
* Cero dependencias de librerías externas o frameworks (solo TypeScript puro).
* Las entidades encapsulan sus propias reglas de invariancia.
* Las interfaces de repositorios no contienen lógica de base de datos (`save`, `findById`, `findByCriteria`).

### 4.2 Capa de Aplicación (`application/`)
* Los casos de uso son clases con un único método público `execute(...)`.
* Reciben DTOs planos y retornan `Result<ResponseDTO, AppError>`.
* Inyectan repositorios a través de interfaces de dominio.

### 4.3 Capa de Infraestructura (`infrastructure/`)
* Implementa las interfaces de repositorio mapeando entre entidades de dominio puras y entidades de persistencia TypeORM (`Mappers`).
* Centraliza las conexiones a PostgreSQL y al emulador Azurite para Azure Blob Storage.

### 4.4 Capa de Presentación (`presentation/` y `app/`)
* Los Server Actions invocan los casos de uso correspondientes y devuelven el objeto `Result` serializado.
* Los componentes cliente utilizan Shadcn UI y Tailwind CSS.
* Las tablas del panel administrativo utilizan **TanStack Table** con paginación y filtros en el cliente/servidor.

---

## 5. Estrategia de Pruebas con Vitest

* Cada caso de uso y entidad de dominio debe contar con su correspondiente archivo de pruebas unitarias (`*.spec.ts`).
* Se deben usar dobles de prueba / mocks en memoria para las interfaces de repositorio en las pruebas de casos de uso.
* Comandos esperados:
  * `npm run test` (ejecuta Vitest).
  * `npm run test:watch` (modo interactivo).

---

## 6. Checklist de Verificación de Calidad

Antes de considerar una tarea completada, el agente debe verificar:
- [ ] ¿Todos los métodos y clases cuentan con JSDoc en castellano?
- [ ] ¿Se respetó el patrón de resultado (`Result.ok` / `Result.fail`) sin lanzar excepciones de negocio?
- [ ] ¿La lógica de negocio se encuentra en el caso de uso y no en el Server Action ni en el componente?
- [ ] ¿Los nombres de variables, métodos y tipos son descriptivos y coherentes?
- [ ] ¿Las pruebas de Vitest (`npm run test`) se ejecutan y pasan sin errores?
- [ ] ¿Las entidades de TypeORM tienen sus respectivas migraciones generadas y no usan `synchronize: true`?
