# Especificación de Requerimientos: Módulo de Moneda, Tasa BCV y Control Horario

**Módulo:** `currency`  
**Bounded Context:** Conversión Monetaria, Tasa Oficial BCV y Ventana Operativa  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
El módulo de moneda administra la conversión financiera bimonetaria de **Hospisalud**. El catálogo opera con precios base en dólares estadounidenses (**USD**), requiriendo la conversión precisa a bolívares soberanos (**VES**) basada en la tasa diaria del Banco Central de Venezuela (**BCV**).

Adicionalmente, este módulo centraliza la invariante de tiempo que controla la **Ventana Operativa de Compras (8:00 AM a 9:00 PM)**, protegiendo las transacciones y congelando los importes monetarios al momento del checkout.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-CUR-01`: El sistema debe almacenar todos los precios base de los productos y tarifas de delivery expresados exclusivamente en USD.
2. `EARS-CUR-02`: El sistema debe calcular el equivalente en VES multiplicando el monto en USD por la tasa BCV oficial vigente (`total_ves = round(total_usd * bcv_rate, 2)`).
3. `EARS-CUR-03`: El sistema debe proveer al Super Administrador una interfaz en su panel de control para actualizar y verificar la tasa oficial BCV diariamente antes de las 8:00 AM.

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
4. `EARS-CUR-04`: **Cuando** el Super Administrador ingresa una nueva tasa oficial y confirma la actualización, el sistema debe registrar la fecha de vigencia, el usuario responsable y activar la nueva tasa para todas las operaciones subsiguientes.
5. `EARS-CUR-05`: **Cuando** un cliente confirma un pedido, el sistema debe registrar y congelar de forma inmutable en la orden: la tasa BCV aplicada, el subtotal en USD, el subtotal en VES, el delivery en USD/VES y los totales finales.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
6. `EARS-CUR-06`: **Mientras** la hora local de Venezuela (`America/Caracas`) se encuentre dentro del rango operativo de **8:00 AM a 9:00 PM**, el sistema debe permitir compras, adición al carrito y finalización de checkout.
7. `EARS-CUR-07`: **Mientras** la hora local se encuentre fuera del rango operativo (**9:01 PM a 7:59 AM**), el sistema debe deshabilitar los botones de compra, activar el modo de sólo consulta y exhibir un banner informando el horario de apertura.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
8. `EARS-CUR-08`: **Si** un usuario intenta enviar una solicitud de confirmación de pedido fuera del horario de 8:00 AM a 9:00 PM (ej. mediante manipulación de cliente o script), **entonces** el servidor debe rechazar la orden retornando el error de dominio `StoreClosedError`.
9. `EARS-CUR-09`: **Si** el Super Administrador intenta registrar una tasa BCV menor o igual a cero, **entonces** el sistema debe rechazar el valor retornando un error de validación.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/currency/
├── domain/
│   ├── entities/
│   │   └── ExchangeRate.ts
│   ├── value-objects/
│   │   ├── MoneyVES.ts
│   │   ├── BCVRateValue.ts
│   │   └── OperatingHoursWindow.ts
│   ├── repositories/
│   │   └── IExchangeRateRepository.ts
│   ├── services/
│   │   └── CurrencyConverterService.ts
│   └── errors/
│       ├── StoreClosedError.ts
│       └── InvalidExchangeRateError.ts
├── application/
│   ├── dtos/
│   │   ├── CurrentRateResponseDTO.ts
│   │   ├── UpdateRateCommandDTO.ts
│   │   └── OperationalStatusDTO.ts
│   └── use-cases/
│       ├── GetCurrentBCVRateUseCase.ts
│       ├── UpdateBCVRateUseCase.ts
│       └── CheckOperatingHoursUseCase.ts
├── infrastructure/
│   ├── entities/
│   │   └── BCVExchangeRateEntity.ts
│   ├── mappers/
│   │   └── ExchangeRateMapper.ts
│   └── repositories/
│       └── TypeOrmExchangeRateRepository.ts
└── presentation/
    ├── actions/
    │   ├── updateRateAction.ts
    │   └── getOperatingStatusAction.ts
    └── components/
        ├── CurrencyToggleDisplay.tsx
        ├── ClosedStoreBanner.tsx
        └── AdminRateUpdateCard.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `CheckOperatingHoursUseCase`
* **Entrada:** `currentTime?: Date`.
* **Lógica:**
  1. Obtener la hora actual en zona horaria `America/Caracas`.
  2. Evaluar si la hora se encuentra entre las 08:00:00 y las 21:00:00.
  3. Retornar DTO con `isOpen: boolean`, mensaje informativo y tiempo restante para apertura/cierre.
* **Salida:** `Result<OperationalStatusDTO, never>`.

### 4.2 `UpdateBCVRateUseCase`
* **Entrada:** `UpdateRateCommandDTO` (`rate: number`, `effectiveDate: string`, `adminUserId: string`).
* **Lógica:**
  1. Validar que `rate > 0`.
  2. Crear entidad `ExchangeRate`.
  3. Persistir en base de datos mediante `IExchangeRateRepository`.
  4. Retornar tasa registrada.
* **Salida:** `Result<CurrentRateResponseDTO, AppError>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `OperatingHoursWindow.spec.ts`:
  * A las 07:59:59 AM debe retornar `isOpen: false`.
  * A las 08:00:00 AM debe retornar `isOpen: true`.
  * A las 21:00:00 PM debe retornar `isOpen: true`.
  * A las 21:00:01 PM debe retornar `isOpen: false`.
* `CurrencyConverterService.spec.ts`:
  * Para $10.00 USD con tasa 36.50 debe retornar exactamente 365.00 VES.
  * Debe redondear matemáticamente a dos (2) decimales según estándar bancario.
