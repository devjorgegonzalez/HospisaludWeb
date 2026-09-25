# Especificación del Modelo de Datos (PostgreSQL) — HospisaludWeb

**Versión:** 1.0  
**Fecha:** 2026-09-24  
**Motor:** PostgreSQL 16+  
**ORM:** TypeORM (`data-source.ts`, migraciones y entidades con decoradores)  
**Puerto Local (Docker):** `9242` (`postgresql://postgres:postgres@localhost:9242/hospisalud`)  
**Metodología:** Spec-Driven Development (SDD)  
**Documento maestro:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Diagrama Entidad-Relación (ERD)

```mermaid
erDiagram
    BRANCHES ||--o{ BRANCH_INVENTORIES : "tiene stock en"
    PRODUCTS ||--o{ BRANCH_INVENTORIES : "distribuido en"
    PRODUCTS ||--o{ PRODUCT_COMPONENTS : "contiene"
    COMPONENTS ||--o{ PRODUCT_COMPONENTS : "presente en"
    BRANCHES ||--o{ USERS : "administra / opera"
    USERS ||--o{ USER_ADDRESSES : "posee libretas de"
    BRANCHES ||--o{ ORDERS : "despacha"
    USERS ||--o{ ORDERS : "realiza (opcional)"
    ORDERS ||--o{ ORDER_ITEMS : "contiene"
    PRODUCTS ||--o{ ORDER_ITEMS : "referencia"
    ORDERS ||--o{ ORDER_STATUS_LOGS : "registra historial"
    USERS ||--o{ ORDER_STATUS_LOGS : "ejecuta cambio"
    BRANCHES ||--o{ STOCK_RESERVATIONS : "reserva en"
    PRODUCTS ||--o{ STOCK_RESERVATIONS : "bloquea unidades de"
    USERS ||--o{ EXCHANGE_RATES : "actualiza tasa"

    BRANCHES {
        uuid id PK
        varchar code UK "HOSPITAL, CENTRO, PETRUCCI, GUANIPA"
        varchar name
        text address
        decimal latitude
        decimal longitude
        varchar phone
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    COMPONENTS {
        uuid id PK
        varchar name UK "Principio activo (ej. Ibuprofeno)"
        text description
        timestamp created_at
        timestamp updated_at
    }

    PRODUCTS {
        uuid id PK
        varchar barcode UK
        varchar commercial_name
        text description
        varchar brand_laboratory
        varchar presentation
        varchar concentration
        varchar administration_route
        decimal base_price_usd
        text image_url
        boolean is_oferta
        boolean is_receta_medica
        boolean is_venta_controlada
        boolean is_destacado
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    PRODUCT_COMPONENTS {
        uuid product_id PK, FK
        uuid component_id PK, FK
    }

    BRANCH_INVENTORIES {
        uuid id PK
        uuid branch_id FK
        uuid product_id FK
        integer stock_quantity
        timestamp updated_at
    }

    STOCK_RESERVATIONS {
        uuid id PK
        uuid branch_id FK
        uuid product_id FK
        varchar session_or_user_id
        integer quantity
        timestamp expires_at
        varchar status "ACTIVE, RELEASED, COMMITTED"
        timestamp created_at
    }

    EXCHANGE_RATES {
        uuid id PK
        varchar currency_from "USD"
        varchar currency_to "VES"
        decimal rate
        date rate_date
        uuid updated_by FK
        timestamp created_at
    }

    USERS {
        uuid id PK
        varchar role "SUPER_ADMIN, BRANCH_ADMIN, OPERATOR, CLIENT"
        uuid branch_id FK "Nullable si es super_admin o client"
        varchar full_name
        varchar cedula
        varchar phone
        varchar email UK "Nullable para guests"
        varchar password_hash "Nullable para guests"
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    USER_ADDRESSES {
        uuid id PK
        uuid user_id FK
        varchar title "Casa, Trabajo, etc."
        text address_line
        varchar zone_type "URBANA, INTERURBANA"
        decimal latitude
        decimal longitude
        text reference_notes
        timestamp created_at
    }

    ORDERS {
        uuid id PK
        varchar order_number UK "ORD-YYYYMMDD-XXXX"
        uuid user_id FK "Nullable para guest"
        varchar guest_name
        varchar guest_cedula
        varchar guest_phone
        uuid branch_id FK "Sede asignada a preparar/despachar"
        varchar delivery_type "PICKUP, DELIVERY"
        varchar delivery_zone "URBANA, INTERURBANA"
        text delivery_address
        decimal delivery_fee_usd
        decimal subtotal_usd
        decimal total_usd
        decimal total_ves
        decimal bcv_rate_used
        varchar payment_method "PAGO_MOVIL, BINANCE_PAY, EFECTIVO_USD"
        varchar payment_reference
        decimal cash_denomination_usd
        decimal cash_change_usd
        varchar order_status "PENDIENTE_VERIFICACION, EN_ESPERA_GUANIPA, EN_PREPARACION, EN_CAMINO, LISTO_PICKUP, ENTREGADO, CANCELADO"
        boolean is_prescription_verified
        uuid approved_by FK "Operador que aprobó Guanipa o verificó pago"
        text notes
        timestamp created_at
        timestamp updated_at
    }

    ORDER_ITEMS {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        varchar product_name
        varchar presentation
        decimal unit_price_usd
        integer quantity
        decimal subtotal_usd
        timestamp created_at
    }

    ORDER_STATUS_LOGS {
        uuid id PK
        uuid order_id FK
        varchar previous_status
        varchar new_status
        uuid changed_by FK
        text notes
        timestamp created_at
    }
```

---

## 2. Definición Detallada de Tablas y Restricciones (DDL)

### 2.1 Tabla `branches` (Sedes)
Representa las cuatro sucursales físicas operativas.

```sql
CREATE TABLE branches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(30) NOT NULL UNIQUE, -- 'HOSPITAL', 'CENTRO', 'PETRUCCI', 'GUANIPA'
    name VARCHAR(100) NOT NULL,
    address TEXT NOT NULL,
    latitude DECIMAL(10, 7) NOT NULL,
    longitude DECIMAL(10, 7) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- Datos semilla fijos obligatorios:
INSERT INTO branches (code, name, address, latitude, longitude, phone) VALUES
('HOSPITAL', 'Sede Hospital', 'Sector Hospital, El Tigre, Edo. Anzoátegui', 8.8872000, -64.2456000, '+584140000001'),
('CENTRO', 'Sede Centro', 'Av. Francisco de Miranda, Centro, El Tigre', 8.8950000, -64.2500000, '+584140000002'),
('PETRUCCI', 'Sede Petrucci', 'Sector Petrucci, El Tigre', 8.8780000, -64.2600000, '+584140000003'),
('GUANIPA', 'Sede Guanipa', 'Av. Fernández Padilla, San José de Guanipa', 8.8820000, -64.1650000, '+584140000004');
```

---

### 2.2 Tabla `components` (Principios Activos)
Catálogo normalizado de componentes activos para el buscador inteligente y filtros.

```sql
CREATE TABLE components (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(150) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_components_name ON components (LOWER(name));
```

---

### 2.3 Tabla `products` (Catálogo Unificado)
Catálogo unificado compartido por todas las sedes con precios en USD y las cuatro banderas de negocio fijas.

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    barcode VARCHAR(64) NOT NULL UNIQUE,
    commercial_name VARCHAR(200) NOT NULL,
    description TEXT,
    brand_laboratory VARCHAR(150) NOT NULL,
    presentation VARCHAR(120) NOT NULL,       -- Ej: 'Caja x 10 Comprimidos'
    concentration VARCHAR(80) NOT NULL,      -- Ej: '500 mg'
    administration_route VARCHAR(80) NOT NULL, -- Ej: 'Oral', 'Tópica', 'Oftálmica'
    base_price_usd NUMERIC(10, 2) NOT NULL CHECK (base_price_usd >= 0),
    image_url TEXT,                          -- Máx 200kb webp
    
    -- Los 4 flags fijos de negocio:
    is_oferta BOOLEAN NOT NULL DEFAULT FALSE,
    is_receta_medica BOOLEAN NOT NULL DEFAULT FALSE,
    is_venta_controlada BOOLEAN NOT NULL DEFAULT FALSE,
    is_destacado BOOLEAN NOT NULL DEFAULT FALSE,
    
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_products_barcode ON products (barcode);
CREATE INDEX idx_products_commercial_name ON products (LOWER(commercial_name));
CREATE INDEX idx_products_flags ON products (is_oferta, is_destacado, is_venta_controlada, is_receta_medica);
CREATE INDEX idx_products_price ON products (base_price_usd);
```

---

### 2.4 Tabla `product_components` (Relación Producto - Principio Activo)
Permite que un medicamento tenga uno o múltiples componentes asociados.

```sql
CREATE TABLE product_components (
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    component_id UUID NOT NULL REFERENCES components(id) ON DELETE RESTRICT,
    PRIMARY KEY (product_id, component_id)
);

CREATE INDEX idx_product_components_comp ON product_components (component_id);
```

---

### 2.5 Tabla `branch_inventories` (Inventario Independiente por Sede)
Maneja el stock numérico exacto por cada sucursal física.

```sql
CREATE TABLE branch_inventories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    CONSTRAINT uq_branch_product UNIQUE (branch_id, product_id)
);

CREATE INDEX idx_branch_inventory_lookup ON branch_inventories (branch_id, product_id);
```

---

### 2.6 Tabla `stock_reservations` (Bloqueo Temporal de 15 Minutos)
Gestiona la retención temporal de unidades durante el checkout antes de confirmar pago.

```sql
CREATE TYPE reservation_status AS ENUM ('ACTIVE', 'RELEASED', 'COMMITTED');

CREATE TABLE stock_reservations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    session_or_user_id VARCHAR(100) NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    status reservation_status NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_reservations_check ON stock_reservations (branch_id, product_id, status, expires_at);
```

---

### 2.7 Tabla `exchange_rates` (Tasa BCV del Día)
Registro histórico y tasa vigente en Bolívares (VES) por Dólar (USD).

```sql
CREATE TABLE exchange_rates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    currency_from VARCHAR(3) NOT NULL DEFAULT 'USD',
    currency_to VARCHAR(3) NOT NULL DEFAULT 'VES',
    rate NUMERIC(12, 4) NOT NULL CHECK (rate > 0),
    rate_date DATE NOT NULL UNIQUE,
    updated_by UUID, -- Referencia a tabla users
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_exchange_rates_date ON exchange_rates (rate_date DESC);
```

---

### 2.8 Tabla `users` (Usuarios y Roles de Sistema)
Soporta clientes invitados (guest), clientes registrados y los tres roles de empleados.

```sql
CREATE TYPE user_role AS ENUM (
    'SUPER_ADMIN',     -- Acceso global, BCV, catálogo maestro
    'BRANCH_ADMIN',    -- Gestión de stock e incidencias de su sede
    'OPERATOR',        -- Kanban de preparación, verificación de pagos
    'CLIENT'           -- Cliente registrado (opcional)
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    role user_role NOT NULL DEFAULT 'CLIENT',
    branch_id UUID REFERENCES branches(id) ON DELETE SET NULL, -- Asignado si es BRANCH_ADMIN u OPERATOR
    full_name VARCHAR(150) NOT NULL,
    cedula VARCHAR(30) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    email VARCHAR(180) UNIQUE,
    password_hash VARCHAR(255),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_users_role_branch ON users (role, branch_id);
CREATE INDEX idx_users_email ON users (email);
```

---

### 2.9 Tabla `user_addresses` (Libreta de Direcciones para Clientes)

```sql
CREATE TABLE user_addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(60) NOT NULL DEFAULT 'Casa', -- 'Casa', 'Trabajo', 'Mamá'
    address_line TEXT NOT NULL,
    zone_type VARCHAR(30) NOT NULL CHECK (zone_type IN ('URBANA', 'INTERURBANA')),
    latitude DECIMAL(10, 7),
    longitude DECIMAL(10, 7),
    reference_notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_user_addresses_user ON user_addresses (user_id);
```

---

### 2.10 Tabla `orders` (Cabecera de Pedidos)
Almacena órdenes tanto de invitados como de usuarios registrados con todos los controles de negocio.

```sql
CREATE TYPE delivery_mode AS ENUM ('PICKUP', 'DELIVERY');
CREATE TYPE order_status_type AS ENUM (
    'PENDIENTE_VERIFICACION',
    'EN_ESPERA_GUANIPA',
    'EN_PREPARACION',
    'EN_CAMINO',
    'LISTO_PICKUP',
    'ENTREGADO',
    'CANCELADO'
);
CREATE TYPE payment_method_type AS ENUM ('PAGO_MOVIL', 'BINANCE_PAY', 'EFECTIVO_USD');

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_number VARCHAR(30) NOT NULL UNIQUE,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL, -- Nullable para Checkout Invitado
    
    -- Datos del comprador
    guest_name VARCHAR(150) NOT NULL,
    guest_cedula VARCHAR(30) NOT NULL,
    guest_phone VARCHAR(30) NOT NULL,
    
    -- Sede de despacho / retiro
    branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE RESTRICT,
    
    -- Modalidad y Logística
    delivery_type delivery_mode NOT NULL,
    delivery_zone VARCHAR(30) CHECK (delivery_zone IN ('URBANA', 'INTERURBANA')),
    delivery_address TEXT,
    delivery_fee_usd NUMERIC(10, 2) NOT NULL DEFAULT 0.00,
    
    -- Montos y Tasa
    subtotal_usd NUMERIC(10, 2) NOT NULL CHECK (subtotal_usd >= 0),
    total_usd NUMERIC(10, 2) NOT NULL CHECK (total_usd >= 0),
    bcv_rate_used NUMERIC(12, 4) NOT NULL,
    total_ves NUMERIC(14, 2) NOT NULL CHECK (total_ves >= 0),
    
    -- Pago
    payment_method payment_method_type NOT NULL,
    payment_reference VARCHAR(120),          -- Ref bancaria o Binance Tx ID
    cash_denomination_usd NUMERIC(10, 2),    -- Billete con que pagará (ej. 20.00)
    cash_change_usd NUMERIC(10, 2),          -- Vuelto a entregar al cliente
    
    -- Estado y Validaciones
    order_status order_status_type NOT NULL DEFAULT 'PENDIENTE_VERIFICACION',
    is_prescription_verified BOOLEAN NOT NULL DEFAULT FALSE,
    special_approval_notes TEXT,             -- Justificación de despacho El Tigre -> Guanipa
    approved_by UUID REFERENCES users(id) ON DELETE SET NULL,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_orders_status ON orders (order_status);
CREATE INDEX idx_orders_branch ON orders (branch_id);
CREATE INDEX idx_orders_created ON orders (created_at DESC);
```

---

### 2.11 Tabla `order_items` (Detalle de Productos por Orden)

```sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    product_name VARCHAR(200) NOT NULL,
    presentation VARCHAR(120) NOT NULL,
    unit_price_usd NUMERIC(10, 2) NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    subtotal_usd NUMERIC(10, 2) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_order_items_order ON order_items (order_id);
```

---

### 2.12 Tabla `order_status_logs` (Trazabilidad y Auditoría)

```sql
CREATE TABLE order_status_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    previous_status order_status_type,
    new_status order_status_type NOT NULL,
    changed_by UUID REFERENCES users(id) ON DELETE SET NULL,
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_order_logs_order ON order_status_logs (order_id);
```

---

## 3. Invariantes y Reglas de Negocio en Base de Datos

1. **Bloqueo de Delivery por Venta Controlada / Receta Médica:**
   - Si una orden contiene al menos un producto donde `is_venta_controlada = TRUE` o `is_receta_medica = TRUE`, `orders.delivery_type` **DEBE ser forzosamente `PICKUP`**.
2. **Delivery Gratis:**
   - Si `orders.delivery_type = 'DELIVERY'` y `orders.subtotal_usd >= 25.00`, entonces `orders.delivery_fee_usd` **DEBE ser 0.00**.
3. **Cálculo de Existencia Disponible:**
   - `Stock Disponible = branch_inventories.stock_quantity - SUM(stock_reservations.quantity WHERE status = 'ACTIVE' AND expires_at > NOW())`.
4. **Regla Guanipa ($5 Delivery):**
   - Si el pedido se enruta a Guanipa pero es despachado desde una sede de El Tigre, la orden entra en `EN_ESPERA_GUANIPA` y al aprobarse se fija `delivery_fee_usd = 5.00`.
