# Spec: Carrito de Compras, Checkout y Pasarela Manual de Pagos

**Módulo:** 04 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
El cliente necesita gestionar su carrito de compras y realizar el checkout en Bolívares (VES) y Dólares (USD). Se debe aplicar rigurosamente la regla legal que inhabilita el servicio de Delivery para medicamentos controlados o con receta médica (forzando Pickup para validación física). Además, la pasarela de pagos debe capturar los comprobantes manuales (Pago Móvil, Binance Pay, o cálculo de vuelto para Efectivo en divisas) y registrar el pedido en estado inicial `PENDIENTE_VERIFICACION`.

---

## 2. Historias de Usuario
- **US-04.1:** Como cliente, quiero agregar productos al carrito y ver el desglose en USD y en VES calculado con la tasa BCV del día.
- **US-04.2:** Como cliente, si mi carrito contiene un medicamento de Venta Controlada o Receta Médica, quiero ser advertido y que solo se me permita elegir la opción "Retiro en Tienda (Pickup)".
- **US-04.3:** Como cliente, si elijo Delivery para productos regulares, quiero ver las tarifas fijas de Zona Urbana ($2.00) e Interurbana ($5.00), o $0.00 si mi compra es igual o superior a $25.00 USD.
- **US-04.4:** Como cliente, quiero pagar mediante Pago Móvil (ingresando referencia), Binance Pay (ingresando ID de transacción) o Efectivo en Dólares (indicando con qué billete pagaré para recibir vuelto exacto).
- **US-04.5:** Como cliente invitado, quiero completar mi pedido ingresando solo nombre, cédula, teléfono y dirección, sin tener que crear una contraseña obligatoria.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-04.1:** El sistema deberá calcular el total de la orden sumando el subtotal de productos más la tarifa de entrega aplicable.
- **AC-04.2:** El sistema deberá crear todo nuevo pedido con el estado inicial `PENDIENTE_VERIFICACION`.

### Dirigido por Estado
- **MIENTRAS** el carrito contenga al menos un producto con `is_venta_controlada = TRUE` o `is_receta_medica = TRUE`, el sistema deberá deshabilitar la opción de "Delivery", seleccionar obligatoriamente "Retiro en Tienda (Pickup)" y mostrar el aviso: *"Este producto requiere presentación de receta física. Solo disponible para Retiro en Tienda"*.
- **MIENTRAS** el subtotal de productos sea `>= 25.00 USD` y la modalidad seleccionada sea "Delivery", el sistema deberá fijar automáticamente el costo de entrega en `$0.00 USD`.

### Dirigido por Evento
- **CUANDO** el cliente selecciona "Pago Móvil", el sistema deberá mostrar los datos de la cuenta bancaria de la farmacia, la tasa BCV oficial y exigir el campo "Número de Referencia" antes de confirmar.
- **CUANDO** el cliente selecciona "Binance Pay", el sistema deberá mostrar el Pay ID / código QR y exigir el "Transaction ID".
- **CUANDO** el cliente selecciona "Efectivo en Divisas" con Delivery, el sistema deberá exigir la denominación del billete (ej. $20) y calcular el vuelto exacto a entregar: `vuelto = billete - total_usd`.
- **CUANDO** el usuario hace clic en "Confirmar Pedido", el sistema deberá crear la orden en PostgreSQL, asociar los ítems y confirmar la reserva de stock.

### Comportamiento no deseado
- **SI** en el pago con efectivo el cliente ingresa una denominación menor al total de la orden, **ENTONCES** el sistema deberá rechazar la confirmación indicando: *"La denominación del billete debe ser mayor o igual al total a pagar"*.

---

## 4. Modelo de Datos Relevante
- `orders`: `id`, `order_number`, `user_id`, `guest_name`, `guest_cedula`, `guest_phone`, `branch_id`, `delivery_type`, `delivery_zone`, `delivery_fee_usd`, `subtotal_usd`, `total_usd`, `total_ves`, `bcv_rate_used`, `payment_method`, `payment_reference`, `cash_denomination_usd`, `cash_change_usd`, `order_status`.
- `order_items`: `order_id`, `product_id`, `product_name`, `unit_price_usd`, `quantity`, `subtotal_usd`.

---

## 5. Contratos de Server Actions
- `createOrderAction(orderData: CreateOrderDTO)`: Valida stock, reglas de venta controlada, cálculos y genera el registro en BD. Retorna `{ success: boolean, orderNumber: string, orderId: string }`.
- `validateCartRestrictions(cartItems: CartItem[])`: Retorna `{ allowDelivery: boolean, requiresPrescription: boolean, warnings: string[] }`.

---

## 6. Casos Límite
- Cambio de tasa BCV en el instante en que el usuario está pagando: La orden se congela con la tasa `bcv_rate_used` guardada al momento de la creación de la orden.
- Desconexión del usuario tras confirmar: El pedido ya quedó registrado en `PENDIENTE_VERIFICACION` con su número de orden para seguimiento.

---

## 7. Fuera de Alcance
- Integración con pasarelas automáticas de tarjeta de crédito internacional (Stripe).
