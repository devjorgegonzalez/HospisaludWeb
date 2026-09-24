# Tasks: Geolocalización, Contexto de Sede y Enrutamiento Inteligente

**Módulo:** 03 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/03-geolocalizacion-enrutamiento/spec.md)

---

- [ ] **Task 03.1:** Implementar función Haversine en `src/domain/geo/haversine.ts` para cálculo de distancia por coordenadas.
- [ ] **Task 03.2:** Definir store de Zustand `useBranchStore.ts` con persistencia en cookie para la sede activa.
- [ ] **Task 03.3:** Crear Server Action `setBranchCookie` y `getActiveBranches` en `branch-actions.ts`.
- [ ] **Task 03.4:** Construir componente `BranchSelector.tsx` para la barra de navegación (satisface US-03.3).
- [ ] **Task 03.5:** Construir modal `GeolocationModal.tsx` con soporte para Geolocation API y fallback a Hospital si se deniega (satisface AC-03.2).
- [ ] **Task 03.6:** Implementar lógica de enrutamiento en `order-router.ts` con soporte para la Regla Especial Guanipa ($5 USD y estado `EN_ESPERA_GUANIPA`).
- [ ] **Task 03.7:** Escribir tests unitarios para el enrutador de pedidos cubriendo casos estándar y caso especial Guanipa.
