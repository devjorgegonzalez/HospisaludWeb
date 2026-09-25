# Plan: Carrito de Compras, Checkout y Pasarela Manual de Pagos

**Módulo:** 04 · **Fase:** Planificar  
**Referencia:** [spec.md](file:///h:/Repos/HospisaludWeb/specs/04-carrito-checkout-pagos/spec.md)

---

## 1. Enfoque Arquitectónico
- **Store del Carrito (Zustand + LocalStorage):** `useCartStore.ts` mantendrá los ítems del carrito localmente, recalculando reactivamente los flags de restricción (`requiresPrescription`, `isControlled`) para bloquear o desbloquear la opción de Delivery al vuelo.
- **Validación Robusta en el Servidor:** Todo lo calculado en el cliente (precios, delivery gratis si subtotal >= $25, inhabilitación de delivery) se recalcula y valida obligatoriamente en el Server Action `createOrderAction` antes del `INSERT` en PostgreSQL.
- **Generación de Código de Orden:** Formato serial legible: `ORD-YYYYMMDD-XXXX` (ej. `ORD-20260924-0042`).

---

## 2. Estructura de Archivos a Crear / Modificar
- `src/domain/orders/types.ts`: DTOs de orden, tipos de pago y estados.
- `src/domain/orders/order-calculator.ts`: Lógica matemática pura de subtotales, conversión VES y vuelto.
- `src/stores/useCartStore.ts`: Store del carrito con validación de restricciones médicas.
- `src/actions/order-actions.ts`: Server Actions para creación y consulta de órdenes.
- `src/app/cart/page.tsx`: Vista del carrito con selector de cantidades y desglose USD/VES.
- `src/app/checkout/page.tsx`: Pantalla de checkout con selector de Delivery/Pickup condicional y pasarela manual.
- `src/app/order-confirmation/[id]/page.tsx`: Pantalla de éxito con detalles del pedido y código de seguimiento.
- `src/components/checkout/PaymentMethodSelector.tsx`: Pestañas interactivas para Pago Móvil, Binance y Efectivo.

---

## 3. Dependencias Nuevas & Componentes shadcn/ui
- **Componentes shadcn/ui requeridos:** `Tabs`, `RadioGroup`, `Input`, `Label`, `Button`, `Alert`, `Card`, `Badge`, `Separator`.
- `zod`: Esquema de validación estricto en frontend y backend para el formulario de checkout.
- `react-hook-form` + `@hookform/resolvers`: Formularios de checkout con validación instantánea.

---

## 4. Decisiones Técnicas
- **Datos Bancarios Fijos / Configurables:** Los datos de Pago Móvil (Banco, Teléfono, RIF) y Binance Pay (Pay ID) se extraerán de variables de entorno configurables o de una tabla de parámetros.
