# Tasks: Catálogo, Buscador Inteligente y Landing Page

**Módulo:** 01 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/01-catalogo-landing/spec.md)

---

- [ ] **Task 01.1:** Definir tipos e interfaces TypeScript en `src/domain/catalog/types.ts` (`Product`, `Component`, `ProductFilter`).
- [ ] **Task 01.2:** Implementar Server Action `manageComponentAction` en `src/actions/component-actions.ts` para CRUD de principios activos.
- [ ] **Task 01.3:** Implementar Server Action `uploadProductImage` en `src/actions/upload-actions.ts` con validación de <=1MB y compresión a WebP <200kb usando `sharp` (satisface EARS evento compresión).
- [ ] **Task 01.4:** Crear componente `ImageUploadDropzone.tsx` para el mantenedor con feedback visual de compresión.
- [ ] **Task 01.5:** Implementar Server Action `searchProducts` en `src/actions/catalog-actions.ts` con filtros combinados por sede, rango de precio y componente (satisface AC-01.2).
- [ ] **Task 01.6:** Construir `ProductCard.tsx` con soporte de doble moneda (USD/VES) y renderizado de los 4 flags fijos (Oferta, Destacado, Receta Médica, Venta Controlada).
- [ ] **Task 01.7:** Construir `HeroSearch.tsx` con selector de sede, input predictivo y dropdowns de filtros.
- [ ] **Task 01.8:** Construir `OfferSection.tsx` con carrusel para productos con `is_oferta = TRUE` (satisface AC-01.1).
- [ ] **Task 01.9:** Construir `FeaturedSection.tsx` con grilla para productos con `is_destacado = TRUE` (satisface AC-01.1).
- [ ] **Task 01.10:** Ensamblar Landing Page en `src/app/page.tsx` siguiendo estrictamente el orden: Hero/Buscador -> Ofertas -> Destacados.
- [ ] **Task 01.11:** Escribir tests unitarios y de integración para búsqueda con filtros y compresión de imagen.
