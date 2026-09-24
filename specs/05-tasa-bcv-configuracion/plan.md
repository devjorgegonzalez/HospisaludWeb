# Plan: Tasa de Cambio Oficial BCV y Configuración Monetaria

**Módulo:** 05 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/05-tasa-bcv-configuracion/spec.md)

---

## 1. Enfoque Arquitectónico
- **Next.js Cache Revalidation:** La tasa se consulta mediante una función con `unstable_cache` o tag de Next.js (`['bcv-rate']`). Al actualizarse en `updateExchangeRateAction`, se dispara `revalidateTag('bcv-rate')` y `revalidatePath('/')`, logrando que todos los Server Components reflejen el nuevo precio en Bolívares instantáneamente sin reiniciar el servidor.
- **Top Bar Badge:** Componente de cliente ligero `BcvRateBadge.tsx` que muestra en la barra superior: `BCV: 1 USD = XX.XX VES`.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/currency/types.ts`: Tipos `ExchangeRate`, `CurrencyConversionResult`.
- `src/domain/currency/currency-service.ts`: Funciones puras de formateo de moneda (USD con formato `$0.00` y VES con formato `Bs. 0,00`).
- `src/actions/exchange-rate-actions.ts`: Server Actions para obtener y actualizar la tasa.
- `src/components/layout/BcvRateBadge.tsx`: Badge visual en el navbar con la tasa del día.
- `src/components/admin/BcvRateManager.tsx`: Formulario de actualización e historial de tasas para el panel del Super Admin.
- `src/app/admin/exchange-rate/page.tsx`: Página del panel administrativo para gestión de tasas.

---

## 3. Dependencias Nuevas
- Ninguna dependencia adicional requerida (utiliza `Intl.NumberFormat` nativo de JavaScript).

---

## 4. Decisiones Técnicas
- **Precisión:** Almacenamiento en base de datos con `NUMERIC(12, 4)` para preservar los 4 decimales exactos publicados oficialmente por el BCV.
