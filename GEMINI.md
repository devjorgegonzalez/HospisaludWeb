# HospisaludWeb - Contexto Maestro del Proyecto

Este documento define la arquitectura técnica, las reglas de negocio, los estándares de código y las directrices operativas para el desarrollo de la plataforma web de la red de farmacias **Hospisalud**. Todos los desarrolladores y agentes de inteligencia artificial deben seguir estas directrices de forma estricta.

---

## 1. Visión General del Negocio

**Hospisalud** es una red de cuatro (4) sucursales farmacéuticas ubicadas en las ciudades de El Tigre y San José de Guanipa (Estado Anzoátegui, Venezuela):
1. **Sede 1:** Hospital (El Tigre)
2. **Sede 2:** Centro (El Tigre)
3. **Sede 3:** Petrucci (El Tigre)
4. **Sede 4:** Guanipa (San José de Guanipa)

El objetivo de la plataforma web es comercializar productos farmacéuticos y de cuidado personal en línea, ofreciendo modalidades de **Retiro en Taquilla (Pickup Express)** y **Despacho a Domicilio (Delivery)** con enrutamiento inteligente de inventario y geolocalización.

---

## 2. Reglas de Negocio Fundamentales

### 2.1 Horario Operativo de Compras
* **Ventana de Venta Web:** Estrictamente de **8:00 AM a 9:00 PM** (hora local de Venezuela `America/Caracas`), unificada para las cuatro sedes.
* **Modo Fuera de Horario (9:01 PM a 7:59 AM):**
  * La plataforma opera únicamente en modo **Catálogo / Consulta**.
  * Los usuarios pueden navegar productos, consultar fichas técnicas y verificar disponibilidad.
  * Se bloquean las opciones de agregar al carrito y procesar checkout, mostrando un banner informativo con el horario de atención.

### 2.2 Gestión de Moneda y Tasa BCV
* **Moneda Base:** Todos los precios en base de datos están fijados en dólares estadounidenses (**USD**).
* **Conversión a Bolívares (VES):** Se realiza según la tasa oficial publicada por el Banco Central de Venezuela (**BCV**).
* **Actualización de Tasa:** El Super Administrador actualiza la tasa oficial diariamente antes de las 8:00 AM mediante un botón/formulario en el panel administrativo.
* **Congelación de Precios:** Al momento en que el cliente presiona *"Confirmar Pedido"*, el total en USD, el total en VES y la tasa BCV aplicada quedan **inmutables y congelados** en el registro histórico de la orden.

### 2.3 Geolocalización y Selección de Sede
* **Ingreso Inicial:** El sistema solicita permiso GPS para detectar la ubicación del usuario y sugerir la sucursal más cercana. Si se deniega el permiso o falla la detección, se asigna la **Sede Hospital** por defecto.
* **Selector Manual:** El usuario puede cambiar su sede activa en la barra superior en cualquier momento.
* **Comportamiento del Carrito al Cambiar de Sede:**
  * Los productos existentes se conservan en el carrito.
  * El sistema revalida las cantidades contra el stock de la nueva sede seleccionada.
  * Si un ítem no posee stock suficiente en la nueva sede, se destaca con una alerta visual (*"Sin stock en esta sede"*) y se deshabilita el botón de checkout hasta que el usuario ajuste la cantidad, elimine el ítem o regrese a la sede con stock.

### 2.4 Catálogo, Metadatos y Búsqueda
* **Estructura de Metadatos del Producto:**
  * Código de barras, Nombre comercial, Principio activo / Componente, Laboratorio / Marca.
  * Categoría principal, Subcategoría, Categorías especiales (*Promociones*, *Ofertas*).
  * Presentación, Concentración, Vía de administración, Foto/Imagen del producto.
  * Tipo de Venta: **Venta Libre** o **Venta Controlada**.
* **Búsqueda Predictiva:** Por nombre comercial, principio activo o marca.
* **Sugerencias por Principio Activo:** Cuando un producto esté agotado en la sede activa, la interfaz muestra en qué otras sedes está disponible y sugiere automáticamente medicamentos sustitutos o equivalentes con el **mismo principio activo** que sí posean existencia en la sede actual.

### 2.5 Medicamentos de Venta Controlada
* **Identificación:** Etiqueta visual de alerta destacada en ficha y carrito (*⚠️ Medicamento de Venta Controlada*).
* **Checkout:** Aceptación obligatoria de cláusula que exige la presentación física del récipe médico original y cédula de identidad al momento de la entrega.
* **Validación Presencial:** No se solicita carga digital de fotos del récipe en la web; la verificación es 100% física en taquilla o por el motorizado en delivery.
* **Protocolo de Entrega Fallida por Falta de Récipe:**
  * El motorizado retiene el medicamento y lo devuelve a la sede.
  * En pagos electrónicos (Pago Móvil / Binance), se reembolsa el costo de los medicamentos pero **no se reembolsa el costo del servicio de delivery** por concepto de traslado efectuado.
  * En pedidos contra entrega en efectivo, la orden se cancela en su totalidad.

### 2.6 Logística de Delivery y Tarifas
* **Ubicación de Entrega:** Determinada mediante mapa interactivo (Google Maps) donde el usuario posiciona un pin geográfico para el cálculo automático de zona y tarifa.
* **Tarifas Base:**
  * **Zona Urbana (Cercana):** $2.00 USD
  * **Zona Interurbana (Media distancia):** $3.00 USD
  * **Zona Urbana (Muy retirada):** $4.00 USD
  * **San Tomé:** $5.00 USD
* **Política de Delivery Gratuito (Subtotal $\ge$ $25.00 USD):**
  * Aplica costo **$0.00 USD** exclusivamente en rutas locales urbanas de El Tigre y locales de Guanipa (tarifas de $2, $3 y $4 USD).
  * **Excepciones estrictas:** La ruta hacia San Tomé ($5.00 USD) y el despacho interurbano El Tigre ➔ Guanipa ($5.00 USD) mantienen su costo íntegro sin exoneración.
* **Regla Especial Guanipa:** Pedidos con entrega en Guanipa se asignan a Sede Guanipa. Si esta no cuenta con el 100% de los productos, se enruta a sedes de El Tigre con tarifa fijada en $5.00 USD.

### 2.7 Reserva y Descuento de Inventario
* **Sin Reserva Prematura:** No se bloquea stock mientras el cliente navega o llena formularios en pantalla.
* **Momento de Bloqueo:** El inventario se descuenta/bloquea en el instante exacto en que el usuario hace clic en *"Confirmar Pedido"*, quedando en estado `Pendiente por Verificación`.
* **Rechazo de Pago:** Si el operador rechaza el pago por referencia errónea o comprobante inválido, el stock se repone inmediatamente al inventario y el operador contacta al cliente vía llamada telefónica.

### 2.8 Medios de Pago y Gestión Bancaria
* **Cuentas Independientes:** Cada sede física dispone de sus propios datos bancarios para Pago Móvil y cuenta Binance Pay. El checkout despliega los datos de la sucursal despachadora.
* **Efectivo en Divisas:** En modalidad delivery, el cliente indica si paga con monto exacto o la denominación del billete (ej. $10, $20, $50) para recibir vuelto en efectivo USD. Se advierte la exigencia de billetes en buen estado físico.
* **Sin Integración WhatsApp:** La plataforma gestiona las órdenes y sus estados de forma interna; la comunicación con el cliente se realiza mediante llamada telefónica al número obligatorio provisto en el checkout.

### 2.9 Operaciones y Despacho en Sede
* **Panel de Despacho en Tiempo Real:** Actualización mediante polling corto (TanStack Query cada 5 a 10 segundos) con **alerta sonora auditiva (*chime*)** y visual ante nuevos pedidos entrantes.
* **Despacho Digital:** El operador lee el detalle del pedido en pantalla y empaqueta la orden (sin impresión de comandas térmicas desde la web). La facturación fiscal se procesa en el sistema de caja físico de la farmacia.
* **Repartidores:** Los motorizados no tienen usuario en el sistema. El operador les comunica la dirección y, tras la confirmación telefónica de entrega, el operador marca la orden como `Entregado`.

### 2.10 Cuentas de Usuario
* **Compra Rápida (Invitado):** Requiere Cédula/RIF, Nombre y Apellido, Teléfono y Dirección. Sin seguimiento web; la farmacia coordina por llamada telefónica.
* **Usuario Registrado:** Acceso mediante Cédula/Correo y Contraseña. Ofrece panel de seguimiento en vivo, libreta de direcciones, historial de órdenes y repetición rápida de tratamientos (*re-order*). Las compras previas hechas como invitado bajo la misma cédula se vinculan automáticamente a la nueva cuenta.

---

## 3. Arquitectura Técnica y Stack Tecnológico

### 3.1 Stack de Desarrollo
* **Framework:** Next.js (App Router, TypeScript en modo estricto).
* **Componentes UI:** Shadcn UI + Tailwind CSS.
* **Tablas de Datos:** TanStack Table (para paneles administrativos y catálogo).
* **ORM:** TypeORM con migraciones versionadas obligatorias (`synchronize: false`).
* **Base de Datos:** PostgreSQL.
* **Testing:** Vitest para pruebas unitarias de dominio y casos de uso.
* **Almacenamiento:** Azure Blob Storage (utilizando el emulador **Azurite** en desarrollo local).
* **Mapas:** Google Maps JavaScript API + Geometry Library.
* **Autenticación:** JWT personalizado en cookies `httpOnly` (`jose` + `bcryptjs`).

### 3.2 Puertos del Entorno Local (Docker)
* **`http://localhost:9251`**: Aplicación Web Next.js (E-commerce y Backoffice).
* **`localhost:9252`**: Base de Datos PostgreSQL.
* **`localhost:9253`**: Emulador Azurite (Azure Blob Storage).

---

## 4. Patrones de Diseño y Reglas de Código

### 4.1 Domain-Driven Design (DDD) - Monolito Modular
El código se organiza en Bounded Contexts independientes ubicados en `src/modules/`:
* `modules/catalog`: Productos, categorías, principios activos, inventario multisede.
* `modules/orders`: Carrito, checkout, cálculo de delivery, pedidos y transiciones de estado.
* `modules/branches`: Sucursales físicas, configuración bancaria, datos de contacto.
* `modules/currency`: Tasa BCV diaria, lógica de conversión y congelación monetaria.
* `modules/auth`: Clientes, roles administrativos (SuperAdmin, AdminSede, Operador), credenciales.
* `shared`: Núcleo compartido (Value Objects comunes, tipos de resultado, abstracciones de base de datos).

**Capas dentro de cada módulo:**
1. **`domain/`**: Entidades, Value Objects, interfaces de repositorio, errores de dominio. Sin dependencias de frameworks ni de TypeORM.
2. **`application/`**: Casos de uso (*Use Cases*), DTOs de entrada y salida.
3. **`infrastructure/`**: Entidades TypeORM, implementación de repositorios, clientes de storage y servicios externos.
4. **`presentation/`**: Server Actions, componentes visuales Shadcn, hooks y páginas de Next.js.

### 4.2 Patrón Repositorio
* La capa de dominio declara interfaces puras (ej. `IProductRepository`, `IOrderRepository`).
* La capa de infraestructura implementa dichas interfaces utilizando TypeORM.
* Los casos de uso dependen únicamente de las abstracciones de repositorio inyectadas.

### 4.3 Patrón de Resultado (Result Pattern)
Queda prohibido lanzar excepciones (`throw`) para controlar flujos de negocio previstos. Se utiliza una unión discriminada funcional serializable compatible con Server Actions:

```typescript
export type Result<T, E = AppError> =
  | { isSuccess: true; isFailure: false; value: T; error: null }
  | { isSuccess: false; isFailure: true; value: null; error: E };

export const Result = {
  ok: <T>(valor: T): Result<T, never> => ({
    isSuccess: true,
    isFailure: false,
    value: valor,
    error: null,
  }),
  fail: <E>(error: E): Result<never, E> => ({
    isSuccess: false,
    isFailure: true,
    value: null,
    error: error,
  }),
};
```

### 4.4 Documentación Obligatoria en Castellano
Todo método, función, clase, interfaz y tipo debe incluir comentarios **JSDoc/TSDoc en idioma castellano**, detallando su propósito, parámetros y resultado:

```typescript
/**
 * Actualiza el nivel de stock disponible de un producto para una sede específica.
 * @param productoId - Identificador único del producto.
 * @param sedeId - Identificador de la sucursal física.
 * @param nuevoStock - Cantidad numérica no negativa de inventario.
 * @returns Resultado con la entidad actualizada o error de dominio si los datos son inválidos.
 */
```
