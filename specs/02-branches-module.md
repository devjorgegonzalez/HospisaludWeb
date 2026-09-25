# Especificación de Requerimientos: Módulo de Sucursales y Selector de Sede

**Módulo:** `branches`  
**Bounded Context:** Sucursales Físicas, Cuentas Bancarias y Selección de Sede  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
El módulo de sucursales modela las cuatro sedes físicas de la red farmacéutica **Hospisalud** en El Tigre y San José de Guanipa. Gestiona la configuración bancaria independiente de cada tienda (datos de Pago Móvil, Binance Pay y efectivo), la geolocalización de las sedes físicas, la asignación automática por GPS en el primer ingreso y la revalidación estricta del carrito de compras ante cambios manuales de sucursal.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-BRN-01`: El sistema debe administrar cuatro sedes físicas fijas con sus coordenadas geográficas, teléfonos y direcciones:
   * **Sede 1:** Hospital (El Tigre)
   * **Sede 2:** Centro (El Tigre)
   * **Sede 3:** Petrucci (El Tigre)
   * **Sede 4:** Guanipa (San José de Guanipa)
2. `EARS-BRN-02`: El sistema debe mantener cuentas bancarias independientes por sede física para Pago Móvil (banco, código, cédula/RIF, teléfono) y Binance Pay (Pay ID, correo, QR).
3. `EARS-BRN-03`: El sistema debe proveer un selector manual de sede visible en el encabezado (*Header*) global en todo momento.

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
4. `EARS-BRN-04`: **Cuando** el usuario ingresa a la plataforma por primera vez, el sistema debe solicitar permiso de geolocalización GPS al navegador.
5. `EARS-BRN-05`: **Cuando** el usuario autoriza el permiso GPS, el sistema debe calcular las distancias euclidianas hacia las 4 sedes y asignar automáticamente como sede activa la sucursal más cercana.
6. `EARS-BRN-06`: **Cuando** el usuario cambia manualmente de sede en el selector superior teniendo productos en el carrito, el sistema debe conservar los ítems y revalidar sus cantidades contra el stock disponible en la nueva sede seleccionada.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
7. `EARS-BRN-07`: **Mientras** la sede seleccionada permanezca activa, todas las vistas de catálogo, búsquedas y checkout deben operar filtrando y reflejando el stock correspondiente a dicha sede.
8. `EARS-BRN-08`: **Mientras** el carrito contenga al menos un ítem cuya cantidad supere el stock de la nueva sede seleccionada, el sistema debe resaltar el ítem en rojo con la advertencia *"Sin stock en esta sede"* y mantener deshabilitado el botón de proceder al checkout.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
9. `EARS-BRN-09`: **Si** el usuario deniega el permiso de geolocalización GPS o el dispositivo no soporta geolocalización, **entonces** el sistema debe asignar por defecto la **Sede Hospital**.
10. `EARS-BRN-10`: **Si** una sucursal física es marcada como inactiva por la administración, **entonces** el sistema debe removerla del selector público y redirigir a los usuarios activos a la Sede Hospital.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/branches/
├── domain/
│   ├── entities/
│   │   ├── Branch.ts
│   │   └── BranchPaymentMethod.ts
│   ├── value-objects/
│   │   ├── GeoCoordinates.ts
│   │   ├── BranchCode.ts
│   │   └── BankAccountDetails.ts
│   ├── repositories/
│   │   └── IBranchRepository.ts
│   └── errors/
│       └── BranchNotFoundError.ts
├── application/
│   ├── dtos/
│   │   ├── BranchResponseDTO.ts
│   │   ├── BranchPaymentConfigDTO.ts
│   │   └── SelectNearestBranchDTO.ts
│   └── use-cases/
│       ├── ListActiveBranchesUseCase.ts
│       ├── GetBranchByIdUseCase.ts
│       ├── DetermineNearestBranchUseCase.ts
│       ├── GetBranchPaymentMethodsUseCase.ts
│       └── UpdateBranchPaymentMethodsUseCase.ts
├── infrastructure/
│   ├── entities/
│   │   ├── BranchEntity.ts
│   │   └── BranchPaymentMethodEntity.ts
│   ├── mappers/
│   │   └── BranchMapper.ts
│   └── repositories/
│       └── TypeOrmBranchRepository.ts
└── presentation/
    ├── actions/
    │   ├── getBranchesAction.ts
    │   ├── setSessionBranchAction.ts
    │   └── getBranchPaymentMethodsAction.ts
    └── components/
        ├── BranchSelector.tsx
        ├── BranchBadge.tsx
        └── BranchInfoCard.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `DetermineNearestBranchUseCase`
* **Entrada:** `latitude: number`, `longitude: number`.
* **Lógica:**
  1. Validar rangos de coordenadas con el Value Object `GeoCoordinates`.
  2. Obtener la lista de sedes activas desde `IBranchRepository`.
  3. Aplicar fórmula de Haversine para calcular la distancia en kilómetros entre el usuario y cada sede.
  4. Seleccionar la sede con menor distancia euclidiana.
* **Salida:** `Result<BranchResponseDTO, AppError>`.

### 4.2 `GetBranchPaymentMethodsUseCase`
* **Entrada:** `branchId: string`.
* **Lógica:**
  1. Consultar los medios de pago activos configurados para la sede solicitada.
  2. Retornar los datos bancarios para Pago Móvil (banco, RIF, teléfono) y Binance (QR y Pay ID).
* **Salida:** `Result<BranchPaymentConfigDTO[], AppError>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `GeoCoordinates.spec.ts`:
  * Debe rechazar latitudes fuera del rango `[-90, 90]` y longitudes fuera de `[-180, 180]`.
* `DetermineNearestBranchUseCase.spec.ts`:
  * Debe retornar 'Hospital' si las coordenadas del usuario están en la zona norte de El Tigre cercana al hospital.
  * Debe retornar 'Guanipa' si las coordenadas corresponden al municipio San José de Guanipa.
  * Si la lista de sedes está vacía o hay error, debe retornar Sede Hospital como fallback seguro.
