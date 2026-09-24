# Plan: Geolocalización, Contexto de Sede y Enrutamiento Inteligente

**Módulo:** 03 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/03-geolocalizacion-enrutamiento/spec.md)

---

## 1. Enfoque Arquitectónico
- **Gestión de Contexto Global:** Se utilizará un store de Zustand (`src/stores/useBranchStore.ts`) sincronizado con cookies HTTP (`hospisalud_branch_id`). Esto permite que los Server Components lean la sede directamente desde los headers/cookies sin provocar un parpadeo (*flash of unstyled content*).
- **Cálculo de Distancia Haversine:** Módulo matemático puro en `src/domain/geo/haversine.ts` para calcular la distancia en kilómetros entre el GPS del usuario y las coordenadas fijas de cada una de las 4 sedes.
- **Motor de Enrutamiento (Routing Engine):** Lógica desacoplada en `src/domain/routing/order-router.ts` que evalúa:
  1. Si el destino es Guanipa.
  2. Si la Sede Guanipa tiene 100% de stock de los items del pedido.
  3. Si no, busca entre Hospital, Centro y Petrucci aquella con stock completo, fijando la orden en `EN_ESPERA_GUANIPA` y el delivery en $5.00 USD.

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/geo/haversine.ts`: Algoritmo de cálculo de distancia geográfica.
- `src/domain/routing/order-router.ts`: Motor de decisión de asignación de sede para órdenes.
- `src/stores/useBranchStore.ts`: Store Zustand para gestión de sede activa en el cliente.
- `src/actions/branch-actions.ts`: Server Actions para obtener sedes activas y fijar cookie de sede.
- `src/components/layout/BranchSelector.tsx`: Dropdown en el Navbar para cambiar de sede manualmente.
- `src/components/layout/GeolocationModal.tsx`: Modal inicial amigable para solicitar permiso GPS con botón "Usar mi ubicación" y "Continuar con Sede Hospital".

---

## 3. Dependencias Nuevas
- `zustand`: Manejo de estado liviano para la sede y el carrito.
- `cookies-next`: Sincronización transparente de cookies entre cliente y servidor.

---

## 4. Decisiones Técnicas
- **Cookie First:** El valor de la sede se guarda en cookies con `SameSite=Lax` y `Path=/`, de modo que los Server Actions y Server Components siempre tienen acceso inmediato a `cookies().get('hospisalud_branch_id')`.
