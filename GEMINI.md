# Contexto del Proyecto: HospisaludWeb

Este documento sirve como la única fuente de verdad (Single Source of Truth) para el desarrollo de la plataforma web de la farmacia multisede, aplicando los principios de **Spec-Driven Design (SDD)** y la estrategia **Kiro**.

## 1. Visión General
Plataforma e-commerce para una red de farmacias con 4 sucursales (Hospital, Centro, Petrucci, Guanipa). La plataforma permite a los usuarios buscar medicamentos, verificar disponibilidad en tiempo real por sede y realizar pedidos para Pickup o Delivery, gestionando inventarios separados pero un catálogo unificado.

## 2. Stack Tecnológico & Infraestructura Local
- **Framework:** Next.js (App Router, Server Actions)
- **Base de Datos:** PostgreSQL 16+
- **Contenedores & Despliegue Local (Docker):**
  - **Servicio Web (Next.js):** Puerto expuesto **`9241`** (`http://localhost:9241`)
  - **Servicio Base de Datos (PostgreSQL):** Puerto expuesto **`9242`** (`localhost:9242`)
- **Autenticación:** 📌 [PENDIENTE DEFINICIÓN]
- **Procesamiento de Imágenes:** Conversión a WebP en frontend/backend (max 200kb de peso final).

## 3. Arquitectura del Landing Page
La página de inicio (`/`) debe estructurarse estrictamente en el siguiente orden:
1. **Hero / Buscador Avanzado:** Permite buscar texto y filtrar por: Sede, Rango de Precio y Principio Activo.
2. **Sección Ofertas:** Carrusel o grilla filtrada por el flag `Oferta`.
3. **Sección Destacados:** Carrusel o grilla filtrada por el flag `Destacado`.

## 4. Reglas de Negocio Core

### 4.1 Inventario y Sedes
- **Catálogo Unificado:** Las 4 sedes comparten los mismos productos, metadatos y precios.
- **Inventario Independiente:** El stock se gestiona manualmente desde el panel administrativo de cada sede; no hay conexión automática con ERPs externos.
- **Geolocalización:** El sistema pedirá GPS. Si se aprueba, asigna la sede más cercana. Si se deniega, el fallback por defecto es **Sede Hospital**.
- **Reserva de Stock:** Al entrar al checkout, el stock se bloquea por 15 minutos. Si el pago no se confirma, se libera.
- **Disponibilidad Cruzada:** Si un producto está en 0, la UI debe indicar: *"Agotado en esta sede. Disponible en [Sede X]"*.

### 4.2 Catálogo y Productos
- **Metadatos Básicos:** Nombre comercial, código de barras, principio activo, marca/laboratorio.
- **Atributos Técnicos:** Presentación, concentración, vía de administración.
- **Gestión de Imágenes:** 
  - El administrador puede editar/subir imágenes.
  - Límite de carga: 1MB máximo.
  - Compresión obligatoria en el sistema: formato `.webp`, peso final < 200kb.
- **Flags Fijos (Banderas booleanas):**
  1. `Oferta`
  2. `Receta Médica`
  3. `Venta Controlada`
  4. `Destacado`

### 4.3 Precios, Pagos y Checkout
- **Moneda:** Precios base fijados en Dólares (USD).
- **Tasa BCV:** La conversión a Bolívares (VES) la dicta la tasa BCV, la cual es actualizada **manualmente** cada día a las 7:00 AM por el Super Administrador en el panel.
- **Pasarela de Pago Manual:**
  - Pago Móvil (requiere ingreso de Referencia).
  - Binance Pay (requiere ID de Transacción).
  - Efectivo (requiere especificar la denominación del billete para calcular el vuelto).
- **Venta Controlada / Receta Médica:** Si el carrito contiene un producto con los flags de restricción, **el Delivery se deshabilita automáticamente**. El usuario solo podrá escoger "Retiro en Tienda" (Pickup) para validación física del récipe.

### 4.4 Logística y Delivery
- **Tarifas (Valores duros para MVP):**
  - Urbana: $2.00 USD.
  - Interurbana: $5.00 USD.
  - Delivery Gratis: Subtotal de productos ≥ $25.00 USD.
- **Regla Especial Guanipa:** Si el pedido es para Guanipa pero la sucursal local no tiene el stock completo, el sistema pondrá el pedido en estado **"En Espera"**. Un operador de El Tigre debe aprobar manualmente el despacho hacia Guanipa (Tarifa fijada a $5 USD si se aprueba).

## 5. Módulos Administrativos (CRUDs)
- **Sedes:** Mantenedor de información de las sucursales.
- **Componentes:** Mantenedor de principios activos.
- **Productos:** Mantenedor central del catálogo (manejo de imágenes y 4 flags).
- **Tasa de Cambio:** Panel exclusivo de Super Admin.
- **Kanban de Pedidos:** Tablero con estados (Pendiente Verificación ➔ En Espera (Guanipa) ➔ En Preparación ➔ En Camino/Listo ➔ Entregado).
- **Usuarios:** Gestión de cuentas (Guest checkout vs Usuarios registrados con historial y libretas de direcciones).

## 6. Notificaciones
Todas las alertas y notificaciones de nuevos pedidos o cambios de estado operan exclusivamente dentro del panel administrativo web en tiempo real (sin integraciones con WhatsApp u otros servicios externos por el momento).
