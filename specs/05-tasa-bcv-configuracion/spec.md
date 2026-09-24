# Spec: Tasa de Cambio Oficial BCV y Configuración Monetaria

**Módulo:** 05 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
Los precios base de todos los medicamentos están fijados en dólares americanos (USD), pero las operaciones comerciales en Venezuela exigen liquidación y visualización del equivalente legal en bolívares (VES) según la tasa oficial del Banco Central de Venezuela (BCV). El Super Administrador debe disponer de una interfaz segura para actualizar manualmente la tasa cada día a las 7:00 AM (o cuando sea necesario), conservando un histórico auditable.

---

## 2. Historias de Usuario
- **US-05.1:** Como Super Administrador, quiero acceder a un panel exclusivo para ingresar la tasa oficial BCV del día.
- **US-05.2:** Como visitante o cliente, quiero ver en la cabecera del sitio la tasa BCV oficial vigente utilizada para las conversiones.
- **US-05.3:** Como Super Administrador, quiero visualizar un historial de las tasas registradas anteriormente con fecha y hora de actualización.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-05.1:** El sistema deberá utilizar la tasa de cambio más reciente registrada en `exchange_rates` para cualquier conversión matemática de USD a VES en la plataforma.
- **AC-05.2:** El sistema deberá restringir la actualización de la tasa BCV estrictamente a usuarios con el rol `SUPER_ADMIN`.

### Dirigido por Evento
- **CUANDO** el Super Administrador ingresa una nueva tasa válida (ej. `45.8500`) y confirma el formulario, el sistema deberá registrar la nueva tasa en la base de datos e invalidar la caché global de Next.js (`revalidatePath('/')`).
- **CUANDO** se actualiza la tasa, el sistema deberá reflejar el nuevo valor en la barra superior pública inmediatamente.

### Comportamiento no deseado
- **SI** un usuario intenta ingresar una tasa menor o igual a cero (`rate <= 0`), **ENTONCES** el sistema deberá rechazar la operación con el mensaje: *"El valor de la tasa debe ser un número positivo válido"*.

---

## 4. Modelo de Datos Relevante
- `exchange_rates`: `id`, `currency_from` (`USD`), `currency_to` (`VES`), `rate` (NUMERIC(12,4)), `rate_date` (DATE), `updated_by` (UUID), `created_at`.

---

## 5. Contratos de Server Actions
- `getCurrentExchangeRate()`: Devuelve `{ rate: number, rateDate: string, updatedAt: string }`.
- `updateExchangeRateAction(newRate: number)`: Solo ejecutable por `SUPER_ADMIN`. Actualiza e invalida tags de caché.
- `getExchangeRateHistory(limit?: number)`: Devuelve lista histórica de tasas.

---

## 6. Casos Límite
- Primer inicio de la aplicación sin tasa registrada: Cargar un valor de fallback predeterminado desde variables de entorno (`DEFAULT_BCV_RATE`) hasta que el Super Admin registre la primera tasa.

---

## 7. Fuera de Alcance
- Integración automática vía web scraping del portal del BCV (acordado explícitamente como manual para evitar bloqueos por Cloudflare/Captcha).
