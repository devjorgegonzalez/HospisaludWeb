# IMPLEMENTATION_PLAN.md - Plan Maestro de Implementación Paso a Paso

Este documento establece la hoja de ruta cronológica, exhaustiva y modular para el desarrollo de la plataforma web de **Hospisalud**. Cada fase y tarea está diseñada para ejecutarse bajo **Spec-Driven Development (SDD)**, garantizando que el equipo de desarrollo o los agentes de IA dispongan de entradas, acciones, artefactos generados y criterios de verificación verificables con **Vitest** en cada etapa.

---

## Índice de Fases

1. [Fase 0: Infraestructura Docker y Scaffolding del Proyecto](#fase-0-infraestructura-docker-y-scaffolding-del-proyecto)
2. [Fase 1: Núcleo Compartido (Shared Core) y Persistencia TypeORM](#fase-1-núcleo-compartido-shared-core-y-persistencia-typeorm)
3. [Fase 2: Módulo de Sucursales (`branches`) y Moneda/BCV (`currency`)](#fase-2-módulo-de-sucursales-branches-y-monedabcv-currency)
4. [Fase 3: Módulo de Catálogo e Inventario Multisede (`catalog`)](#fase-3-módulo-de-catálogo-e-inventario-multisede-catalog)
5. [Fase 4: Módulo de Autenticación, Usuarios y Roles (`auth`)](#fase-4-módulo-de-autenticación-usuarios-y-roles-auth)
6. [Fase 5: Módulo de Carrito, Geolocalización y Checkout (`orders`)](#fase-5-módulo-de-carrito-geolocalización-y-checkout-orders)
7. [Fase 6: Tablero de Despacho y Operaciones en Sede (`orders/dispatch`)](#fase-6-tablero-de-despacho-y-operaciones-en-sede-ordersdispatch)
8. [Fase 7: Auditoría de Calidad, Integración Final y Despliegue](#fase-7-auditoría-de-calidad-integración-final-y-despliegue)

---

## Fase 0: Infraestructura Docker y Scaffolding del Proyecto

### Objetivo
Establecer el entorno contenerizado local con los puertos asignados y la base del proyecto Next.js con TypeScript, Tailwind CSS, Shadcn UI y Vitest.

### Tareas Detalladas:
- [ ] **0.1 Orquestación Docker:**
  * Crear `docker-compose.yml` configurando tres servicios:
    1. `web`: Next.js exponiendo el puerto `9251:9251`.
    2. `db`: PostgreSQL 16 Alpine exponiendo el puerto `9252:5432` con volumen persistente `pgdata`.
    3. `azurite`: Emulador de Azure Blob Storage exponiendo el puerto `9253:10000` con volumen `azuritedata`.
  * Crear `.env.example` y `.env.local` con las cadenas de conexión y puertos.
  * *Verificación:* Ejecutar `docker compose up -d` y validar conectividad a los puertos `9251`, `9252` y `9253`.
- [ ] **0.2 Scaffolding Next.js (App Router) y Sistema de Diseño Responsivo ("Clinical Precision"):**
  * Inicializar el proyecto con Next.js 14+ (App Router, TypeScript estricto, Tailwind CSS configurado para enfoque Mobile-First).
  * Configurar `tsconfig.json` con soporte para decoradores y alias de rutas (`@/modules/*`, `@/shared/*`, `@/components/*`).
  * Integrar tokens de diseño y paleta de `design/DESIGN.md` en `tailwind.config.ts`:
    * Colores: `navy-deep` (`#2B3467`), `powder-blue` (`#BAD7E9`), `clinical-red` (`#EB455F`), `surface-ivory` (`#FCFFE7`), `surface-subtle` (`#F8FAFC`), `border-subtle` (`#E2E8F0`).
    * Tipografías de Google Fonts: *Plus Jakarta Sans* (encabezados), *Inter* (cuerpo y formularios), *JetBrains Mono* (códigos de barras y tasas BCV).
  * Instalar e inicializar primitivos de **Shadcn UI** (`components.json`) adaptando sus variables CSS al tema *Clinical Precision*.
  * Configurar **TanStack Table** (`@tanstack/react-table`) con soporte para vistas responsivas en pantallas móviles.
- [ ] **0.3 Configuración del Framework de Pruebas (Vitest):**
  * Instalar y configurar `vitest`, `@testing-library/react` y `jsdom`.
  * Configurar script `npm run test` en `package.json`.
  * *Verificación:* Ejecutar prueba de humo con Vitest verificando salida limpia en verde.

---

## Fase 1: Núcleo Compartido (Shared Core) y Persistencia TypeORM

### Objetivo
Implementar los bloques de construcción universales de Domain-Driven Design y configurar TypeORM con migraciones versionadas estrictas (`synchronize: false`).

### Tareas Detalladas:
- [ ] **1.1 Implementación del Patrón de Resultado Funcional:**
  * Crear en `src/shared/domain/Result.ts` el tipo discriminado `Result<T, E>` y los constructores `Result.ok(val)` y `Result.fail(err)`.
  * Crear la jerarquía base de errores `AppError` en `src/shared/domain/AppError.ts`.
  * Crear pruebas unitarias en `src/shared/domain/Result.spec.ts`.
- [ ] **1.2 Configuración del DataSource de TypeORM:**
  * Crear `src/shared/infrastructure/persistence/data-source.ts` con patrón Singleton adaptado a Next.js App Router.
  * Configurar scripts de migración en `package.json`: `typeorm:migration:generate`, `typeorm:migration:run`, `typeorm:migration:revert`.
- [ ] **1.3 Primera Migración Formal de Base de Datos:**
  * Crear entidades TypeORM iniciales mapeando exactamente el esquema definido en [DATA_MODEL.md](file:///h:/Repos/HospisaludWeb/DATA_MODEL.md).
  * Generar y ejecutar la migración inicial: `001-InitialMigration.ts`.
  * *Verificación:* Comprobar en PostgreSQL (puerto `9252`) la creación de las 15 tablas relacionales con sus índices y llaves foráneas.
- [ ] **1.4 Seed Data Inicial de Sucursales y Metadatos:**
  * Crear script de siembra (`seed.ts`) que inserte:
    * Las 4 sedes físicas (**Hospital**, **Centro**, **Petrucci**, **Guanipa**).
    * Cuentas demo de Pago Móvil y Binance Pay para cada sede.
    * Polígonos base de las zonas de delivery ($2, $3, $4 y $5).
    * Usuario inicial con rol `SUPER_ADMIN`.

---

## Fase 2: Módulo de Sucursales (`branches`) y Moneda/BCV (`currency`)

### Objetivo
Construir los contextos que controlan la identidad física de las sedes, la geolocalización inicial, el selector manual, la conversión monetaria y la ventana horaria (8:00 AM a 9:00 PM).

### Tareas Detalladas:
- [ ] **2.1 Capa de Dominio de Sucursales y Moneda:**
  * Implementar entidades `Branch`, `BranchPaymentMethod` y `ExchangeRate`.
  * Implementar Value Objects: `GeoCoordinates`, `BranchCode`, `BCVRateValue`, `MoneyUSD`, `MoneyVES`.
  * Implementar servicio de dominio `OperatingHoursWindow` (validación de rango 8:00 AM a 9:00 PM).
  * Crear pruebas unitarias en Vitest para cada entidad y Value Object.
- [ ] **2.2 Casos de Uso de Sucursales y Horario:**
  * `ListActiveBranchesUseCase`: Retorna las 4 sucursales activas.
  * `DetermineNearestBranchUseCase`: Calcula distancia Euclidiana/Haversine y retorna la sede más cercana o Sede Hospital como fallback.
  * `CheckOperatingHoursUseCase`: Determina si la tienda está abierta para compras o en modo catálogo.
  * `UpdateBCVRateUseCase`: Permite al Super Admin registrar la nueva tasa del día.
- [ ] **2.3 Capa de Presentación (UI & Server Actions):**
  * Crear componente `BranchSelector` en el encabezado global para cambio manual de sede.
  * Implementar `ClosedStoreBanner`: Banner visible fuera del horario (9:01 PM a 7:59 AM) alertando sobre el modo catálogo.
  * Implementar `AdminRateUpdateCard`: Tarjeta en el panel administrativo para carga diaria de la tasa oficial.

---

## Fase 3: Módulo de Catálogo e Inventario Multisede (`catalog`)

### Objetivo
Implementar la gestión completa de productos, principios activos, laboratorios, almacenamiento de imágenes en Azurite y el motor de recomendaciones por principio activo.

### Tareas Detalladas:
- [ ] **3.1 Capa de Dominio de Catálogo:**
  * Entidades `Product`, `Category`, `Subcategory`, `ActiveIngredient`, `Laboratory`.
  * Value Objects: `Barcode`, `SaleType` (`VENTA_LIBRE` vs `VENTA_CONTROLADA`).
  * Interfaz de repositorio `IProductRepository`.
  * Pruebas unitarias en `Product.spec.ts`.
- [ ] **3.2 Integración de Almacenamiento Azure Blob / Azurite:**
  * Implementar `AzureBlobStorageClient` en `infrastructure/storage/` apuntando a `http://localhost:9253` en local.
  * Crear servicio de subida de imágenes con generación de nombres UUID y compresión WebP.
- [ ] **3.3 Casos de Uso del Catálogo:**
  * `SearchProductsUseCase`: Búsqueda predictiva con filtros por sede, categoría y texto tridimensional (nombre, principio activo, marca).
  * `GetSubstitutesByActiveIngredientUseCase`: Sugiere medicamentos sustitutos con el mismo principio activo y stock > 0 en la sede activa cuando el original esté agotado.
  * `AdjustBranchStockUseCase`: Actualización individual de stock por sede en panel administrativo.
- [ ] **3.4 Vistas de Usuario y Panel Administrativo:**
  * `ProductCard` y `ProductGrid`: Listado de catálogo con precios en USD y VES a la tasa del día.
  * `ControlledMedBadge`: Etiqueta visible (*⚠️ Medicamento de Venta Controlada*).
  * `AdminProductForm`: Formulario individual para alta y edición de productos con subida de imagen a Azurite.

---

## Fase 4: Módulo de Autenticación, Usuarios y Roles (`auth`)

### Objetivo
Establecer la seguridad basada en JWT con cookies `httpOnly`, soporte para invitados (Guest Checkout) y vinculación histórica automática de pedidos por Cédula.

### Tareas Detalladas:
- [ ] **4.1 Capa de Dominio de Autenticación:**
  * Entidades `User` y `UserAddress`.
  * Value Objects: `CedulaRIF`, `Email`, `UserRole` (`SUPER_ADMIN`, `ADMIN_SEDE`, `OPERADOR`, `CLIENTE`).
  * Pruebas unitarias para validación de formato de Cédula/RIF venezolana.
- [ ] **4.2 Servicios de Infraestructura Criptográfica y Tokens:**
  * Implementar `BcryptPasswordHasher` con salt factor 10.
  * Implementar `JoseTokenService` emitiendo JWT firmados con expiración configurable.
- [ ] **4.3 Casos de Uso de Autenticación:**
  * `RegisterUserUseCase`: Registro de cuenta formal con Cédula, Correo y Contraseña.
  * `LinkGuestOrdersUseCase`: Actualiza retroactivamente `customer_id` en todos los pedidos previos hechos con esa misma Cédula como invitado.
  * `LoginUserUseCase`: Verificación segura de credenciales e inyección de cookie `httpOnly`.
- [ ] **4.4 Middleware de Protección y RBAC:**
  * Configurar `middleware.ts` en la raíz de Next.js protegiendo rutas `/admin/*` y `/operaciones/*`.
  * Componentes de UI: `LoginForm`, `RegisterForm`, `SavedAddressesList`.

---

## Fase 5: Módulo de Carrito, Geolocalización y Checkout (`orders`)

### Objetivo
Desarrollar el flujo central de compra: carrito multisede con revalidación, mapa de Google Maps con pin de delivery, cálculo de zonas ($2, $3, $4, $5), exoneración a $0 para subtotal $\ge$ $25 y descuento atómico de inventario.

### Tareas Detalladas:
- [ ] **5.1 Estado del Carrito y Revalidación de Stock Multisede:**
  * Implementar hook/store `useCart` en el cliente.
  * **Regla de negocio crítica:** Al cambiar de sede en el header, revalidar cantidades; si falta stock en la nueva sede, marcar el producto en rojo (*"Sin stock en esta sede"*) y deshabilitar el checkout.
- [ ] **5.2 Integración de Google Maps y Cálculo de Zonas:**
  * Implementar componente `InteractiveGoogleMap` con Google Maps JavaScript API + Geometry Library.
  * Permitir posicionar el pin geográfico y calcular en tiempo real a qué polígono de zona pertenece el destino.
  * Implementar servicio `DeliveryCalculatorService`:
    * Zonas locales: $2.00, $3.00, $4.00 USD.
    * San Tomé y Despacho Cruzado El Tigre ➔ Guanipa: $5.00 USD.
    * Exoneración a $0.00 USD si `subtotal >= 25.00 USD` (exclusivo para zonas locales de $2, $3 y $4 USD; no aplica para San Tomé ni cruce El Tigre-Guanipa).
- [ ] **5.3 Pasarela de Checkout y Bloqueo Atómico de Stock:**
  * Formulario de datos: Cédula/RIF, Nombre, Teléfono obligatorio, Dirección.
  * Selector de método de pago:
    * **Pago Móvil:** Despliega datos de la sede despachadora e ingresa referencia.
    * **Binance Pay:** Despliega QR y Pay ID de la sede e ingresa número de transacción.
    * **Efectivo en Divisas:** Selección de denominación de billete ($10, $20, $50), cálculo de vuelto en USD y advertencia de billetes en buen estado.
  * Aceptación de términos legales para medicamentos de Venta Controlada.
  * `CreateOrderUseCase`: Transacción atómica que bloquea stock (`SELECT ... FOR UPDATE`), descuenta unidades y congela tasa BCV y montos en la orden con estado inicial `PENDIENTE_VERIFICACION`.

---

## Fase 6: Tablero de Despacho y Operaciones en Sede (`orders/dispatch`)

### Objetivo
Construir el panel operativo en tiempo real para los operadores de cada sucursal farmacéutica, alertas auditivas y ciclo de vida de las órdenes.

### Tareas Detalladas:
- [ ] **6.1 Tablero en Tiempo Real con Polling y Alerta Sonora:**
  * Implementar componente `DispatchDashboard` utilizando **TanStack Table** filtrado por la sede del operador.
  * Configurar polling corto con **TanStack Query** (cada 5 a 10 segundos).
  * Integrar `ChimeAudioPlayer` que reproduce el sonido auditivo (*chime*) cada vez que ingresa una nueva orden `PENDIENTE_VERIFICACION`.
- [ ] **6.2 Despacho Digital y Empaque en Pantalla:**
  * Implementar `DigitalPackingModal`: Visualización clara de los medicamentos a empacar con sus presentaciones y cantidades (sin impresión de comandas térmicas).
  * Casilla de verificación para medicamentos de Venta Controlada exigiendo al operador confirmar físicamente el récipe original al despachar.
- [ ] **6.3 Flujo de Verificación y Rechazo de Pagos:**
  * `VerifyPaymentUseCase`: Valida el pago y avanza a `EN_PREPARACION`.
  * `RejectOrderWithStockRestorationUseCase`: Operador rechaza con motivo obligatorio (ej. comprobante inválido), reponiendo inmediatamente el stock al inventario y mostrando el teléfono para llamada directa al cliente.
- [ ] **6.4 Cierre de Venta y Logística con Repartidores:**
  * Transición a `LISTO_PARA_RETIRO` (Pickup en taquilla express).
  * Transición a `EN_CAMINO` (Despacho con motorizado sin usuario en el sistema).
  * Operador marca `ENTREGADO` una vez que el motorizado confirma la entrega vía llamada telefónica.
  * Manejo de excepción de entrega fallida por falta de récipe médico físico: anulación con retención del costo de flete y devolución del medicamento a sede.

---

## Fase 7: Auditoría de Calidad, Integración Final y Despliegue

### Objetivo
Verificar el cumplimiento estricto de todas las directrices de `AGENTS.md`, ejecutar pruebas automatizadas y dejar el entorno listo para producción.

### Tareas Detalladas:
- [ ] **7.1 Verificación de Documentación en Castellano:**
  * Ejecutar linter para asegurar que el 100% de métodos, clases e interfaces cuenten con JSDoc en castellano.
- [ ] **7.2 Cobertura de Pruebas Unitarias:**
  * Ejecutar suite completa con `npm run test` (Vitest), asegurando que todas las entidades de dominio y casos de uso pasen satisfactoriamente.
- [ ] **7.3 Pruebas de Integración y Flujo Completo:**
  * Probar flujo completo de compra como invitado y registrado.
  * Probar regla de cambio de sede con carrito lleno.
  * Probar cálculo de tarifas y exoneración de delivery a $25.
  * Probar bloqueo y reposición atómica de stock.
- [ ] **7.4 Auditoría de Diseño Responsivo (Mobile-First):**
  * Verificar en navegadores móviles (iOS Safari, Android Chrome) y viewports responsive (375px, 768px, 1024px, 1440px).
  * Comprobar que no exista desbordamiento horizontal (*horizontal overflow*).
  * Validar usabilidad de botones táctiles (mínimo 44x44px), legibilidad de textos y navegación con una mano.
- [ ] **7.5 Contenerización Final:**
  * Compilar imagen de producción Docker optimizada en multistage (`standalone` mode en Next.js).
