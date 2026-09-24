# Tasks: Carrito de Compras, Checkout y Pasarela Manual de Pagos

**Módulo:** 04 · **Fase:** Tareas Ejecutables  
**Trazabilidad:** Mapeo 1:1 a Criterios de Aceptación de [spec.md](file:///h:/Repos/HospisaludWeb/specs/04-carrito-checkout-pagos/spec.md)

---

- [ ] **Task 04.1:** Definir tipos DTOs y esquemas Zod en `src/domain/orders/types.ts`.
- [ ] **Task 04.2:** Implementar calculadora de orden en `order-calculator.ts` (subtotales, delivery gratis >= $25 USD, cálculo de vuelto para efectivo).
- [ ] **Task 04.3:** Crear store Zustand `useCartStore.ts` con persistencia y detección de productos controlados/receta médica.
- [ ] **Task 04.4:** Construir página del carrito `src/app/cart/page.tsx` con soporte de doble moneda USD/VES.
- [ ] **Task 04.5:** Construir formulario de checkout en `src/app/checkout/page.tsx` con bloqueo reactivo de Delivery ante productos controlados (satisface EARS estado).
- [ ] **Task 04.6:** Construir componente `PaymentMethodSelector.tsx` con campos para Pago Móvil (Ref), Binance (TxID) y Efectivo (Denominación).
- [ ] **Task 04.7:** Implementar Server Action `createOrderAction` con validaciones de seguridad e inserción en PostgreSQL.
- [ ] **Task 04.8:** Crear pantalla de confirmación `src/app/order-confirmation/[id]/page.tsx` con número de orden y resumen.
- [ ] **Task 04.9:** Escribir tests unitarios para las reglas de delivery gratis, cálculo de vuelto y bloqueo de delivery por venta controlada.
