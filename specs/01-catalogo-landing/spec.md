# Spec: Catálogo, Buscador Inteligente y Landing Page

**Módulo:** 01 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
Los clientes necesitan explorar el catálogo de medicamentos de forma ágil, buscar por nombre o principio activo, filtrar por sede y rango de precio, y visualizar rápidamente ofertas y productos destacados en una página de inicio estructurada, con precios convertidos a VES. Además, los administradores deben poder gestionar las imágenes de los productos cumpliendo estrictamente con los límites de tamaño (<1MB origen, <200kb webp destino).

---

## 2. Historias de Usuario
- **US-01.1:** Como visitante, quiero visualizar en la página de inicio (`/`) un buscador avanzado con filtros, una sección de Ofertas y una sección de Destacados en ese orden estricto.
- **US-01.2:** Como visitante, quiero buscar productos por nombre comercial, principio activo o marca, aplicando filtros de sede, rango de precio y componente.
- **US-01.3:** Como visitante, quiero ver en cada tarjeta de producto su precio en USD y su equivalente en VES calculado a la tasa oficial del día.
- **US-01.4:** Como administrador, quiero subir imágenes de productos de máximo 1MB, asegurando que el sistema las convierta a WebP con peso final menor a 200kb.
- **US-01.5:** Como administrador, quiero gestionar los principios activos (componentes) en un mantenedor CRUD dedicado.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-01.1:** El sistema deberá presentar en la página principal (`/`) el siguiente orden vertical: 1) Hero con Buscador y Filtros, 2) Carrusel/Grilla de Ofertas (`is_oferta = TRUE`), 3) Carrusel/Grilla de Destacados (`is_destacado = TRUE`).
- **AC-01.2:** El sistema deberá calcular y mostrar el precio en Bolívares (`VES = base_price_usd * tasa_bcv_vigente`) junto al precio en USD en todos los listados y fichas.

### Dirigido por Evento
- **CUANDO** el usuario ingresa un término de búsqueda o selecciona un principio activo, el sistema deberá filtrar reactivamente los productos coincidentes respetando la sede seleccionada.
- **CUANDO** el administrador sube una imagen de producto superior a 1MB en el formulario de administración, el sistema deberá rechazar la carga con el mensaje "El archivo excede el tamaño máximo permitido de 1MB".
- **CUANDO** el administrador sube una imagen válida (<= 1MB), el sistema deberá comprimirla a formato `.webp` garantizando un peso final inferior a 200kb antes de almacenarla.

### Dirigido por Estado
- **MIENTRAS** un producto tenga activo el flag `is_oferta = TRUE`, el sistema deberá incluirlo en la sección de Ofertas del landing page y mostrar el badge visual "Oferta".
- **MIENTRAS** un producto tenga activo el flag `is_destacado = TRUE`, el sistema deberá incluirlo en la sección de Destacados del landing page.

### Comportamiento no deseado
- **SI** un producto no cuenta con imagen cargada, **ENTONCES** el sistema deberá desplegar una imagen de placeholder médica por defecto sin romper el layout.

---

## 4. Modelo de Datos Relevante
- `products`: `id`, `barcode`, `commercial_name`, `brand_laboratory`, `presentation`, `concentration`, `administration_route`, `base_price_usd`, `image_url`, `is_oferta`, `is_receta_medica`, `is_venta_controlada`, `is_destacado`, `is_active`.
- `components`: `id`, `name`, `description`.
- `product_components`: `product_id`, `component_id`.
- `branches`: `id`, `code`, `name`.
- `branch_inventories`: `branch_id`, `product_id`, `stock_quantity`.

---

## 5. Contratos de Server Actions / Endpoints
- `getStorefrontData(branchId: string)`: Devuelve `{ offers: Product[], featured: Product[], bcvRate: number }`.
- `searchProducts(filters: { query?: string, branchId: string, minPrice?: number, maxPrice?: number, componentId?: string, page?: number })`: Devuelve `{ products: ProductWithStock[], total: number }`.
- `uploadProductImage(formData: FormData)`: Valida tamaño <= 1MB, comprime a `.webp` < 200kb y devuelve `{ imageUrl: string }`.
- `manageComponentAction(data: { id?: string, name: string, description?: string, action: 'CREATE' | 'UPDATE' | 'DELETE' })`.

---

## 6. Casos Límite
- Búsqueda con caracteres especiales o tildes (ej. "Ácido acetilsalicílico") → normalizar a minúsculas y sin acentos con `unaccent` o `ILIKE`.
- Imágenes PNG transparentes → mantener transparencia o aplicar fondo blanco al convertir a WebP.
- Sin conexión temporal al storage de imágenes → fallback graceful a imagen local `/images/placeholder-med.webp`.

---

## 7. Fuera de Alcance
- Procesamiento de videos o galerías 3D de productos.
- Múltiples imágenes por producto (se mantiene estrictamente una imagen principal por producto en MVP).
