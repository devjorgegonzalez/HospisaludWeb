# Plan: Catálogo, Buscador Inteligente y Landing Page

**Módulo:** 01 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/01-catalogo-landing/spec.md)

---

## 1. Enfoque Arquitectónico
- **Next.js App Router (Server Components):** La página raíz (`app/page.tsx`) cargará los datos de Ofertas, Destacados y Componentes en el servidor para máxima velocidad de renderizado (SEO y carga inicial instantánea).
- **Client Components para Interactividad:** El buscador predictivo con filtros (`app/_components/SearchFilters.tsx`) y los carruseles se implementarán como Client Components optimizados con `useTransition` o `nuqs` (URL query state).
- **Pipeline de Imágenes:** 
  - Validación cliente con `File.size <= 1048576` (1MB).
  - Conversión/compresión servidor mediante `sharp` en un Server Action de Next.js (`sharp(buffer).webp({ quality: 80 }).toBuffer()`), verificando que el output sea `<= 200 * 1024` bytes.
  - Almacenamiento en directorio público `/public/uploads/products/` para desarrollo local, o compatible con S3/Supabase Storage.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/catalog/types.ts`: Interfaces TypeScript de `Product`, `Component`, `ProductFilter`.
- `src/actions/catalog-actions.ts`: Server Actions para búsqueda con filtros y catálogo de storefront.
- `src/actions/upload-actions.ts`: Pipeline de validación y compresión de imagen WebP con `sharp`.
- `src/actions/component-actions.ts`: CRUD de principios activos.
- `src/app/page.tsx`: Landing Page estructurada: Hero/Buscador -> Ofertas -> Destacados.
- `src/app/products/[id]/page.tsx`: Ficha técnica de detalle del producto.
- `src/components/storefront/HeroSearch.tsx`: Hero con buscador predictivo y drawer de filtros.
- `src/components/storefront/OfferSection.tsx`: Carrusel de ofertas.
- `src/components/storefront/FeaturedSection.tsx`: Grilla de destacados.
- `src/components/storefront/ProductCard.tsx`: Tarjeta con precio USD/VES y badges de flags.
- `src/components/admin/ImageUploadDropzone.tsx`: Dropzone con preview y validación de 1MB.

---

## 3. Dependencias Nuevas & Componentes shadcn/ui
- **Componentes shadcn/ui requeridos:** `Card`, `Badge`, `Button`, `Input`, `Select`, `Dialog`, `Slider` (rango de precios).
- `sharp`: Procesamiento, redimensión y compresión a WebP en Node.js.
- `lucide-react`: Iconografía para el buscador, badges médicos y filtros.
- `embla-carousel-react`: Carrusel accesible y ligero para la sección de Ofertas.

---

## 4. Decisiones Técnicas
- **Filtros en URL:** Los parámetros de búsqueda (`q`, `branch`, `minPrice`, `maxPrice`, `component`) se sincronizarán en la URL (`/search?q=...`) permitiendo compartir enlaces y navegación con historial.
- **Normalización de Texto:** Búsquedas PostgreSQL usando `ILIKE` y normalización de mayúsculas/minúsculas.
