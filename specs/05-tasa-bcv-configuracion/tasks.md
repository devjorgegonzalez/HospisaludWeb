# Tasks: Tasa de Cambio Oficial BCV y Configuración Monetaria

**Módulo:** 05 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/05-tasa-bcv-configuracion/spec.md)

---

- [ ] **Task 05.1:** Implementar funciones de formateo y conversión monetaria en `currency-service.ts`.
- [ ] **Task 05.2:** Implementar Server Action `getCurrentExchangeRate` con cache tagging de Next.js.
- [ ] **Task 05.3:** Implementar Server Action `updateExchangeRateAction` con validación de rol `SUPER_ADMIN` e invalidación de caché.
- [ ] **Task 05.4:** Construir componente `BcvRateBadge.tsx` para exhibición pública de la tasa en el Navbar (satisface US-05.2).
- [ ] **Task 05.5:** Construir formulario y tabla histórica en `src/app/admin/exchange-rate/page.tsx` para el Super Admin.
- [ ] **Task 05.6:** Escribir tests unitarios para las funciones de conversión USD a VES y formateo contable.
