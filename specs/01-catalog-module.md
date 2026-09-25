# Especificación de Requerimientos: Módulo de Catálogo e Inventario Multisede

**Módulo:** `catalog`  
**Bounded Context:** Catálogo de Productos, Principios Activos e Inventario Multisede  
**Estándar:** Spec-Driven Development (SDD) / Kiro Style / Sintaxis EARS  

---

## 1. Resumen Ejecutivo
El módulo de catálogo gestiona el repositorio unificado de productos farmacéuticos y de cuidado personal de **Hospisalud**, sus clasificaciones taxonómicas, principios activos, laboratorios y el inventario físico independiente para las cuatro sucursales (**Hospital**, **Centro**, **Petrucci** y **Guanipa**).

Incorpora la lógica de búsqueda predictiva ultrarrápida, el motor de recomendaciones por principio activo ante productos agotados, y el tratamiento diferenciado entre medicamentos de **Venta Libre** y de **Venta Controlada**.

---

## 2. Requerimientos del Sistema (Sintaxis EARS)

### 2.1 Requerimientos Ubicuos (Ubiquitous)
1. `EARS-CAT-01`: El sistema debe mantener un catálogo general de productos y una estructura de precios base en USD idéntica y sincronizada para todas las sucursales.
2. `EARS-CAT-02`: El sistema debe almacenar y gestionar el nivel de stock físico de cada producto de manera numérica e independiente por cada una de las 4 sedes.
3. `EARS-CAT-03`: El sistema debe almacenar para cada producto los metadatos completos: código de barras, nombre comercial, principio activo, laboratorio fabricante, subcategoría, presentación, concentración, vía de administración, tipo de venta (`VENTA_LIBRE` o `VENTA_CONTROLADA`), URL de imagen y precio en USD.

### 2.2 Requerimientos Impulsados por Eventos (Event-Driven)
4. `EARS-CAT-04`: **Cuando** el usuario ingresa un término de búsqueda en el buscador inteligente, el sistema debe ejecutar una coincidencia predictiva sobre el nombre comercial, principio activo o laboratorio, retornando resultados en menos de 100 ms.
5. `EARS-CAT-05`: **Cuando** un usuario visualiza la ficha de un producto cuyo stock es igual a cero (0) en la sede activa, el sistema debe consultar la disponibilidad en las restantes sedes y mostrar el mensaje: *"Agotado en esta sede. Disponible en [Sede X]"*.
6. `EARS-CAT-06`: **Cuando** un producto esté agotado en la sede activa, el sistema debe consultar y recomendar automáticamente productos equivalentes que compartan el **mismo Principio Activo** y que posean stock disponible mayor a cero en la sede activa.

### 2.3 Requerimientos Impulsados por Estados (State-Driven)
7. `EARS-CAT-07`: **Mientras** la sede seleccionada mantenga stock mayor a cero de un producto, el sistema debe permitir agregar unidades al carrito hasta el límite del stock disponible de dicha sede.
8. `EARS-CAT-08`: **Mientras** el stock de un producto en la sede activa sea cero, el sistema debe deshabilitar el botón de compra para esa sede y desplegar las opciones de cambio de sede o medicamentos sustitutos.

### 2.4 Requerimientos de Comportamiento No Deseado / Errores (Unwanted Behavior)
9. `EARS-CAT-09`: **Si** un operador intenta registrar un stock numérico negativo en el panel administrativo, **entonces** el sistema debe rechazar la operación y retornar un error de validación de dominio.
10. `EARS-CAT-10`: **Si** se intenta crear un producto con un código de barras ya existente en el catálogo, **entonces** el sistema debe rechazar la creación indicando duplicidad de código.

### 2.5 Requerimientos Opcionales / Características Condicionales (Optional)
11. `EARS-CAT-11`: **Donde** un producto posea la clasificación `VENTA_CONTROLADA`, el sistema debe exhibir en su ficha de catálogo y en el carrito una etiqueta visual destacada de advertencia legal (*⚠️ Medicamento de Venta Controlada*).
12. `EARS-CAT-12`: **Donde** un producto esté marcado como `is_featured: true`, el sistema debe exhibirlo en la sección destacada de promociones y ofertas especiales de la página principal.

---

## 3. Arquitectura del Módulo (DDD)

```text
src/modules/catalog/
├── domain/
│   ├── entities/
│   │   ├── Product.ts
│   │   ├── Category.ts
│   │   ├── Subcategory.ts
│   │   ├── ActiveIngredient.ts
│   │   └── Laboratory.ts
│   ├── value-objects/
│   │   ├── Barcode.ts
│   │   ├── MoneyUSD.ts
│   │   ├── StockQuantity.ts
│   │   └── SaleType.ts
│   ├── repositories/
│   │   ├── IProductRepository.ts
│   │   ├── ICategoryRepository.ts
│   │   └── IActiveIngredientRepository.ts
│   └── errors/
│       ├── ProductNotFoundError.ts
│       └── InsufficientStockError.ts
├── application/
│   ├── dtos/
│   │   ├── ProductResponseDTO.ts
│   │   ├── CreateProductDTO.ts
│   │   ├── SearchProductsQueryDTO.ts
│   │   └── AdjustStockDTO.ts
│   └── use-cases/
│       ├── SearchProductsUseCase.ts
│       ├── GetProductDetailsUseCase.ts
│       ├── GetSubstitutesByActiveIngredientUseCase.ts
│       ├── CreateProductUseCase.ts
│       ├── UpdateProductUseCase.ts
│       └── AdjustBranchStockUseCase.ts
├── infrastructure/
│   ├── entities/
│   │   ├── ProductEntity.ts
│   │   ├── CategoryEntity.ts
│   │   ├── SubcategoryEntity.ts
│   │   ├── ActiveIngredientEntity.ts
│   │   ├── LaboratoryEntity.ts
│   │   └── BranchInventoryEntity.ts
│   ├── mappers/
│   │   └── ProductMapper.ts
│   ├── repositories/
│   │   ├── TypeOrmProductRepository.ts
│   │   └── TypeOrmCategoryRepository.ts
│   └── storage/
│       └── AzureBlobStorageClient.ts
└── presentation/
    ├── actions/
    │   ├── searchProductsAction.ts
    │   ├── getProductDetailsAction.ts
    │   └── adjustStockAction.ts
    └── components/
        ├── ProductCard.tsx
        ├── ProductGrid.tsx
        ├── SubstituteRecommendations.tsx
        ├── ControlledMedBadge.tsx
        └── AdminProductForm.tsx
```

---

## 4. Casos de Uso Principales

### 4.1 `SearchProductsUseCase`
* **Entrada:** `SearchProductsQueryDTO` (`searchTerm: string`, `branchId: string`, `categoryId?: string`, `page: number`, `limit: number`).
* **Lógica:**
  1. Limpiar y normalizar el término de búsqueda.
  2. Consultar repositorio mediante coincidencia de texto en `name`, `activeIngredient.name` y `laboratory.name`.
  3. Cruzar con el inventario de la `branchId` solicitada.
  4. Retornar DTO con lista paginada de productos, indicando disponibilidad inmediata en la sede y disponibilidad cruzada en otras sedes.
* **Salida:** `Result<PaginatedProductsDTO, AppError>`.

### 4.2 `GetSubstitutesByActiveIngredientUseCase`
* **Entrada:** `productId: string`, `branchId: string`.
* **Lógica:**
  1. Obtener el producto origen y su `activeIngredientId`.
  2. Buscar todos los productos activos con el mismo `activeIngredientId` distintos al producto origen.
  3. Filtrar aquellos cuyo stock en `branchId` sea mayor a 0.
  4. Retornar la lista ordenada por precio o relevancia.
* **Salida:** `Result<ProductResponseDTO[], AppError>`.

### 4.3 `AdjustBranchStockUseCase`
* **Entrada:** `AdjustStockDTO` (`branchId: string`, `productId: string`, `newStock: number`, `operatorId: string`).
* **Lógica:**
  1. Validar que `newStock >= 0`.
  2. Cargar la entidad de inventario de la sede bajo transacción.
  3. Asignar el nuevo valor de stock.
  4. Guardar y retornar resultado exitoso.
* **Salida:** `Result<void, AppError>`.

---

## 5. Criterios de Aceptación y Pruebas Unitarias (Vitest)

* `Product.spec.ts`:
  * Debe validar correctamente que un código de barras no esté vacío.
  * Debe rechazar precios en USD negativos.
  * Debe identificar correctamente si un medicamento es de venta controlada.
* `SearchProductsUseCase.spec.ts`:
  * Debe retornar productos cuando la búsqueda coincide con el principio activo.
  * Debe marcar `isAvailableInBranch: false` si el stock en la sede es 0.
* `GetSubstitutesByActiveIngredientUseCase.spec.ts`:
  * Debe retornar únicamente sustitutos que tengan stock > 0 en la sede especificada.
  * No debe incluir el mismo producto agotado dentro de la lista de sustitutos.
