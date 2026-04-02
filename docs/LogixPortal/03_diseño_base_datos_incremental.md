# Diseño de Base de Datos Incremental - LogixPortal

---

## FILOSOFÍA DEL DISEÑO INCREMENTAL

Este diseño de base de datos crece **orgánicamente** con cada fase del proyecto:

1. **Fase 0**: Tablas fundamentales (multi-tenancy, usuarios, maestros básicos)
2. **Fase 1**: Tablas de importaciones
3. **Fase 2**: Tablas de exportaciones
4. **Fase 3**: Tablas de inventario avanzado
5. **Fase 4**: Tablas de finanzas y facturación
6. **Fase 5-7**: Tablas de planificación, calidad e integraciones

**Principios:**
- Cada fase es **auto-contenida** pero **compatible** con futuras fases
- Usamos **foreign keys opcionales** para features futuras
- **No eliminamos** columnas, solo agregamos
- Usamos **migraciones versionadas** (Serverpod migrations)

---

## CONVENCIONES DE NOMENCLATURA

```sql
-- Nombres de tablas: snake_case, plural
users, purchase_orders, shipment_containers

-- Nombres de columnas: snake_case
created_at, tenant_id, po_number

-- Primary keys: "id" (SERIAL/BIGSERIAL)
id SERIAL PRIMARY KEY

-- Foreign keys: "{tabla_singular}_id"
tenant_id, user_id, supplier_id

-- Timestamps: 
created_at TIMESTAMP DEFAULT NOW()
updated_at TIMESTAMP DEFAULT NOW()

-- Soft deletes (cuando aplique):
deleted_at TIMESTAMP
```

---

## FASE 0: FUNDACIÓN
### Tablas Core del Sistema

```sql
-- =====================================================
-- MULTI-TENANCY
-- =====================================================

CREATE TABLE tenants (
    id BIGSERIAL PRIMARY KEY,
    ruc VARCHAR(11) UNIQUE NOT NULL,
    company_name VARCHAR(200) NOT NULL,
    trade_name VARCHAR(200),
    address TEXT,
    phone VARCHAR(20),
    email VARCHAR(100),
    logo_url TEXT,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT chk_ruc_format CHECK (ruc ~ '^[0-9]{11}$')
);

COMMENT ON TABLE tenants IS 'Empresas clientes del sistema (multi-tenant)';

-- Índices
CREATE INDEX idx_tenants_active ON tenants(active) WHERE active = true;

-- =====================================================
-- USUARIOS Y AUTENTICACIÓN
-- =====================================================

CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    full_name VARCHAR(200) GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,
    document_type VARCHAR(10), -- DNI, CE, PASAPORTE
    document_number VARCHAR(20),
    phone VARCHAR(20),
    avatar_url TEXT,
    two_factor_enabled BOOLEAN DEFAULT false,
    two_factor_secret VARCHAR(100),
    active BOOLEAN DEFAULT true,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT chk_document_type CHECK (document_type IN ('DNI', 'CE', 'PASAPORTE'))
);

COMMENT ON TABLE users IS 'Usuarios del sistema';

-- Índices
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_active ON users(tenant_id, active) WHERE active = true;

-- =====================================================
-- ROLES Y PERMISOS (RBAC)
-- =====================================================

CREATE TABLE roles (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_system_role BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, name)
);

CREATE TABLE permissions (
    id BIGSERIAL PRIMARY KEY,
    module VARCHAR(50) NOT NULL,
    action VARCHAR(50) NOT NULL,
    resource VARCHAR(100) NOT NULL,
    description TEXT,
    
    UNIQUE(module, action, resource)
);

-- Permisos iniciales
INSERT INTO permissions (module, action, resource, description) VALUES
    ('procurement', 'create', 'quotation_requests', 'Crear solicitudes de cotización'),
    ('procurement', 'read', 'quotation_requests', 'Ver solicitudes de cotización'),
    ('procurement', 'update', 'quotation_requests', 'Editar solicitudes de cotización'),
    ('procurement', 'delete', 'quotation_requests', 'Eliminar solicitudes de cotización'),
    ('procurement', 'approve', 'quotation_requests', 'Aprobar solicitudes de cotización'),
    ('procurement', 'create', 'purchase_orders', 'Crear órdenes de compra'),
    ('procurement', 'read', 'purchase_orders', 'Ver órdenes de compra'),
    ('procurement', 'approve', 'purchase_orders', 'Aprobar órdenes de compra');

CREATE TABLE role_permissions (
    role_id BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id BIGINT NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    granted_at TIMESTAMP DEFAULT NOW(),
    
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP DEFAULT NOW(),
    assigned_by BIGINT REFERENCES users(id),
    
    PRIMARY KEY (user_id, role_id)
);

-- =====================================================
-- MAESTROS: MONEDAS Y TIPOS DE CAMBIO
-- =====================================================

CREATE TABLE currencies (
    code VARCHAR(3) PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    symbol VARCHAR(5) NOT NULL,
    active BOOLEAN DEFAULT true
);

INSERT INTO currencies (code, name, symbol) VALUES
    ('PEN', 'Soles', 'S/'),
    ('USD', 'Dólares Americanos', '$'),
    ('EUR', 'Euros', '€');

CREATE TABLE exchange_rates (
    id BIGSERIAL PRIMARY KEY,
    date DATE NOT NULL,
    from_currency VARCHAR(3) NOT NULL REFERENCES currencies(code),
    to_currency VARCHAR(3) NOT NULL REFERENCES currencies(code),
    buy_rate DECIMAL(10, 6) NOT NULL,
    sell_rate DECIMAL(10, 6) NOT NULL,
    source VARCHAR(20) DEFAULT 'SBS',
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(date, from_currency, to_currency)
);

CREATE INDEX idx_exchange_rates_date ON exchange_rates(date DESC);

-- =====================================================
-- MAESTROS: UBIGEOS (Departamento/Provincia/Distrito)
-- =====================================================

CREATE TABLE ubigeos (
    code VARCHAR(6) PRIMARY KEY,
    department VARCHAR(100) NOT NULL,
    province VARCHAR(100) NOT NULL,
    district VARCHAR(100) NOT NULL,
    full_name VARCHAR(300) GENERATED ALWAYS AS (
        department || ' / ' || province || ' / ' || district
    ) STORED
);

-- Se carga desde archivo CSV del INEI
-- https://www.inei.gob.pe/media/MenuRecursivo/publicaciones_digitales/Est/Lib1140/cap02.pdf

-- =====================================================
-- MAESTROS: UNIDADES DE MEDIDA
-- =====================================================

CREATE TABLE units_of_measure (
    id BIGSERIAL PRIMARY KEY,
    code VARCHAR(10) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    symbol VARCHAR(10),
    sunat_code VARCHAR(10),
    active BOOLEAN DEFAULT true
);

-- Unidades comunes según SUNAT
INSERT INTO units_of_measure (code, name, symbol, sunat_code) VALUES
    ('NIU', 'Unidad', 'UND', 'NIU'),
    ('KGM', 'Kilogramo', 'KG', 'KGM'),
    ('TNE', 'Tonelada', 'TON', 'TNE'),
    ('MTR', 'Metro', 'M', 'MTR'),
    ('MTK', 'Metro cuadrado', 'M2', 'MTK'),
    ('MTQ', 'Metro cúbico', 'M3', 'MTQ'),
    ('LTR', 'Litro', 'L', 'LTR'),
    ('DZN', 'Docena', 'DOC', 'DZN'),
    ('GRM', 'Gramo', 'G', 'GRM');

-- =====================================================
-- MAESTROS: PRODUCTOS
-- =====================================================

CREATE TABLE product_categories (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    parent_id BIGINT REFERENCES product_categories(id),
    code VARCHAR(20) NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, code)
);

CREATE INDEX idx_product_categories_tenant ON product_categories(tenant_id);

CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    product_code VARCHAR(50) NOT NULL,
    barcode VARCHAR(50),
    name VARCHAR(300) NOT NULL,
    description TEXT,
    category_id BIGINT REFERENCES product_categories(id),
    unit_of_measure_id BIGINT NOT NULL REFERENCES units_of_measure(id),
    
    -- Características
    product_type VARCHAR(20) DEFAULT 'FINISHED_GOOD',
    is_purchasable BOOLEAN DEFAULT true,
    is_sellable BOOLEAN DEFAULT true,
    
    -- Costos y precios
    standard_cost DECIMAL(15, 4),
    sale_price DECIMAL(15, 4),
    
    -- Impuestos
    tax_type VARCHAR(20) DEFAULT 'IGV',
    igv_rate DECIMAL(5, 2) DEFAULT 18.00,
    
    -- Dimensiones
    weight_kg DECIMAL(10, 4),
    volume_m3 DECIMAL(10, 6),
    
    -- Control
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, product_code),
    CONSTRAINT chk_product_type CHECK (product_type IN ('RAW_MATERIAL', 'FINISHED_GOOD', 'SERVICE')),
    CONSTRAINT chk_tax_type CHECK (tax_type IN ('IGV', 'EXONERATED', 'EXEMPT'))
);

COMMENT ON TABLE products IS 'Catálogo maestro de productos';

CREATE INDEX idx_products_tenant ON products(tenant_id);
CREATE INDEX idx_products_tenant_code ON products(tenant_id, product_code);
CREATE INDEX idx_products_active ON products(tenant_id, active) WHERE active = true;

-- =====================================================
-- MAESTROS: PROVEEDORES
-- =====================================================

CREATE TABLE suppliers (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    supplier_code VARCHAR(20) NOT NULL,
    document_type VARCHAR(10) NOT NULL,
    document_number VARCHAR(20) NOT NULL,
    business_name VARCHAR(300) NOT NULL,
    trade_name VARCHAR(300),
    address TEXT,
    country_code VARCHAR(2) DEFAULT 'PE', -- ISO 3166-1 alpha-2
    phone VARCHAR(20),
    email VARCHAR(100),
    contact_person VARCHAR(200),
    contact_email VARCHAR(100),
    supplier_type VARCHAR(20) DEFAULT 'LOCAL',
    payment_term_days INTEGER DEFAULT 30,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, supplier_code),
    CONSTRAINT chk_document_type_supplier CHECK (document_type IN ('RUC', 'DNI', 'TAX_ID', 'PASSPORT')),
    CONSTRAINT chk_supplier_type CHECK (supplier_type IN ('LOCAL', 'INTERNATIONAL'))
);

COMMENT ON TABLE suppliers IS 'Proveedores (locales e internacionales)';

CREATE INDEX idx_suppliers_tenant ON suppliers(tenant_id);
CREATE INDEX idx_suppliers_tenant_code ON suppliers(tenant_id, supplier_code);
CREATE INDEX idx_suppliers_active ON suppliers(tenant_id, active) WHERE active = true;

-- =====================================================
-- MAESTROS: CLIENTES
-- =====================================================

CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    customer_code VARCHAR(20) NOT NULL,
    document_type VARCHAR(10) NOT NULL,
    document_number VARCHAR(20) NOT NULL,
    business_name VARCHAR(300),
    trade_name VARCHAR(300),
    address TEXT,
    country_code VARCHAR(2) DEFAULT 'PE',
    phone VARCHAR(20),
    email VARCHAR(100),
    customer_type VARCHAR(20) DEFAULT 'B2B',
    payment_term_days INTEGER DEFAULT 0,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, customer_code),
    CONSTRAINT chk_document_type_customer CHECK (document_type IN ('RUC', 'DNI', 'TAX_ID', 'PASSPORT')),
    CONSTRAINT chk_customer_type CHECK (customer_type IN ('B2B', 'B2C'))
);

COMMENT ON TABLE customers IS 'Clientes (locales e internacionales)';

CREATE INDEX idx_customers_tenant ON customers(tenant_id);
CREATE INDEX idx_customers_tenant_code ON customers(tenant_id, customer_code);

-- =====================================================
-- AUDITORÍA
-- =====================================================

CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    user_id BIGINT REFERENCES users(id),
    action VARCHAR(50) NOT NULL,
    entity_type VARCHAR(100) NOT NULL,
    entity_id BIGINT NOT NULL,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_tenant ON audit_logs(tenant_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);

-- =====================================================
-- FUNCIONES Y TRIGGERS
-- =====================================================

-- Función para actualizar updated_at automáticamente
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Aplicar trigger a tablas con updated_at
CREATE TRIGGER update_tenants_updated_at BEFORE UPDATE ON tenants
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_products_updated_at BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_suppliers_updated_at BEFORE UPDATE ON suppliers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_customers_updated_at BEFORE UPDATE ON customers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

---

## FASE 1: MÓDULO DE IMPORTACIONES

```sql
-- =====================================================
-- SOLICITUDES DE COTIZACIÓN (RFQ)
-- =====================================================

CREATE TABLE quotation_requests (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    request_number VARCHAR(50) NOT NULL,
    request_date DATE NOT NULL DEFAULT CURRENT_DATE,
    requested_by BIGINT NOT NULL REFERENCES users(id),
    required_date DATE,
    priority VARCHAR(20) DEFAULT 'NORMAL',
    status VARCHAR(20) DEFAULT 'DRAFT',
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, request_number),
    CONSTRAINT chk_qr_priority CHECK (priority IN ('LOW', 'NORMAL', 'HIGH', 'URGENT')),
    CONSTRAINT chk_qr_status CHECK (status IN ('DRAFT', 'SENT', 'PARTIAL_RESPONSE', 'CLOSED', 'CANCELLED'))
);

CREATE INDEX idx_quotation_requests_tenant ON quotation_requests(tenant_id);
CREATE INDEX idx_quotation_requests_status ON quotation_requests(tenant_id, status);

CREATE TABLE quotation_request_items (
    id BIGSERIAL PRIMARY KEY,
    quotation_request_id BIGINT NOT NULL REFERENCES quotation_requests(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    product_id BIGINT NOT NULL REFERENCES products(id),
    description TEXT,
    quantity DECIMAL(15, 4) NOT NULL,
    unit_of_measure_id BIGINT NOT NULL REFERENCES units_of_measure(id),
    notes TEXT,
    
    UNIQUE(quotation_request_id, line_number),
    CONSTRAINT chk_qri_quantity CHECK (quantity > 0)
);

CREATE TABLE quotation_request_suppliers (
    id BIGSERIAL PRIMARY KEY,
    quotation_request_id BIGINT NOT NULL REFERENCES quotation_requests(id) ON DELETE CASCADE,
    supplier_id BIGINT NOT NULL REFERENCES suppliers(id),
    sent_at TIMESTAMP,
    responded_at TIMESTAMP,
    
    UNIQUE(quotation_request_id, supplier_id)
);

-- =====================================================
-- COTIZACIONES RECIBIDAS
-- =====================================================

CREATE TABLE quotations (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    quotation_number VARCHAR(50) NOT NULL,
    quotation_request_id BIGINT REFERENCES quotation_requests(id),
    supplier_id BIGINT NOT NULL REFERENCES suppliers(id),
    quotation_date DATE NOT NULL,
    valid_until DATE,
    currency_code VARCHAR(3) NOT NULL REFERENCES currencies(code),
    exchange_rate DECIMAL(10, 6) DEFAULT 1.000000,
    incoterm VARCHAR(10), -- EXW, FOB, CIF, etc.
    delivery_port VARCHAR(100),
    delivery_time_days INTEGER,
    payment_terms TEXT,
    status VARCHAR(20) DEFAULT 'RECEIVED',
    subtotal DECIMAL(15, 2) NOT NULL,
    total_amount DECIMAL(15, 2) NOT NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, quotation_number),
    CONSTRAINT chk_quotation_status CHECK (status IN ('RECEIVED', 'UNDER_REVIEW', 'APPROVED', 'REJECTED'))
);

CREATE INDEX idx_quotations_tenant ON quotations(tenant_id);
CREATE INDEX idx_quotations_supplier ON quotations(supplier_id);

CREATE TABLE quotation_items (
    id BIGSERIAL PRIMARY KEY,
    quotation_id BIGINT NOT NULL REFERENCES quotations(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    product_id BIGINT NOT NULL REFERENCES products(id),
    description TEXT,
    quantity DECIMAL(15, 4) NOT NULL,
    unit_price DECIMAL(15, 4) NOT NULL,
    subtotal DECIMAL(15, 2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    
    UNIQUE(quotation_id, line_number)
);

-- =====================================================
-- ÓRDENES DE COMPRA INTERNACIONALES
-- =====================================================

CREATE TABLE purchase_orders (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    po_number VARCHAR(50) NOT NULL,
    po_date DATE NOT NULL DEFAULT CURRENT_DATE,
    quotation_id BIGINT REFERENCES quotations(id),
    supplier_id BIGINT NOT NULL REFERENCES suppliers(id),
    currency_code VARCHAR(3) NOT NULL REFERENCES currencies(code),
    exchange_rate DECIMAL(10, 6) DEFAULT 1.000000,
    incoterm VARCHAR(10) NOT NULL,
    port_of_loading VARCHAR(100),
    port_of_discharge VARCHAR(100),
    delivery_time_days INTEGER,
    payment_method VARCHAR(50),
    status VARCHAR(20) DEFAULT 'DRAFT',
    subtotal DECIMAL(15, 2) NOT NULL,
    freight DECIMAL(15, 2) DEFAULT 0,
    insurance DECIMAL(15, 2) DEFAULT 0,
    other_charges DECIMAL(15, 2) DEFAULT 0,
    total_amount DECIMAL(15, 2) NOT NULL,
    notes TEXT,
    created_by BIGINT NOT NULL REFERENCES users(id),
    approved_by BIGINT REFERENCES users(id),
    approved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, po_number),
    CONSTRAINT chk_po_status CHECK (status IN ('DRAFT', 'SENT', 'CONFIRMED', 'IN_TRANSIT', 'COMPLETED', 'CANCELLED')),
    CONSTRAINT chk_po_incoterm CHECK (incoterm IN ('EXW', 'FCA', 'FAS', 'FOB', 'CFR', 'CIF', 'CPT', 'CIP', 'DAP', 'DPU', 'DDP'))
);

CREATE INDEX idx_purchase_orders_tenant ON purchase_orders(tenant_id);
CREATE INDEX idx_purchase_orders_status ON purchase_orders(tenant_id, status);

CREATE TABLE purchase_order_items (
    id BIGSERIAL PRIMARY KEY,
    po_id BIGINT NOT NULL REFERENCES purchase_orders(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    product_id BIGINT NOT NULL REFERENCES products(id),
    description TEXT,
    quantity DECIMAL(15, 4) NOT NULL,
    unit_price DECIMAL(15, 4) NOT NULL,
    subtotal DECIMAL(15, 2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    hs_code VARCHAR(20), -- Partida arancelaria
    received_quantity DECIMAL(15, 4) DEFAULT 0,
    
    UNIQUE(po_id, line_number),
    CONSTRAINT chk_poi_quantity CHECK (quantity > 0)
);

-- =====================================================
-- EMBARQUES
-- =====================================================

CREATE TABLE shipments (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    shipment_number VARCHAR(50) NOT NULL,
    po_id BIGINT NOT NULL REFERENCES purchase_orders(id),
    shipment_type VARCHAR(20) DEFAULT 'SEA_FCL',
    carrier_name VARCHAR(200),
    vessel_name VARCHAR(200),
    voyage_number VARCHAR(50),
    booking_number VARCHAR(50),
    bill_of_lading VARCHAR(50),
    container_number VARCHAR(50),
    seal_number VARCHAR(50),
    port_of_loading VARCHAR(100),
    port_of_discharge VARCHAR(100),
    etd DATE, -- Estimated Time of Departure
    eta DATE, -- Estimated Time of Arrival
    atd DATE, -- Actual Time of Departure
    ata DATE, -- Actual Time of Arrival
    status VARCHAR(20) DEFAULT 'BOOKED',
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, shipment_number),
    CONSTRAINT chk_shipment_type CHECK (shipment_type IN ('SEA_FCL', 'SEA_LCL', 'AIR', 'LAND')),
    CONSTRAINT chk_shipment_status CHECK (status IN ('BOOKED', 'IN_TRANSIT', 'ARRIVED', 'CLEARED', 'DELIVERED'))
);

CREATE INDEX idx_shipments_tenant ON shipments(tenant_id);
CREATE INDEX idx_shipments_po ON shipments(po_id);

-- =====================================================
-- DECLARACIÓN ÚNICA DE ADUANAS (DUA)
-- =====================================================

CREATE TABLE customs_declarations (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    dua_number VARCHAR(50),
    shipment_id BIGINT NOT NULL REFERENCES shipments(id),
    customs_broker VARCHAR(200),
    declaration_date DATE,
    customs_value_usd DECIMAL(15, 2),
    customs_value_pen DECIMAL(15, 2),
    tariff_amount DECIMAL(15, 2),
    igv_amount DECIMAL(15, 2),
    other_taxes DECIMAL(15, 2),
    total_duties_taxes DECIMAL(15, 2),
    status VARCHAR(20) DEFAULT 'DRAFT',
    clearance_date DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT chk_dua_status CHECK (status IN ('DRAFT', 'SUBMITTED', 'UNDER_REVIEW', 'CLEARED', 'HELD'))
);

CREATE INDEX idx_customs_declarations_tenant ON customs_declarations(tenant_id);
CREATE INDEX idx_customs_declarations_shipment ON customs_declarations(shipment_id);

-- =====================================================
-- COSTEO DE IMPORTACIÓN (LANDED COST)
-- =====================================================

CREATE TABLE landed_costs (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    po_id BIGINT NOT NULL REFERENCES purchase_orders(id),
    shipment_id BIGINT REFERENCES shipments(id),
    customs_declaration_id BIGINT REFERENCES customs_declarations(id),
    
    -- Costos
    fob_amount DECIMAL(15, 2),
    freight_amount DECIMAL(15, 2),
    insurance_amount DECIMAL(15, 2),
    cif_amount DECIMAL(15, 2) GENERATED ALWAYS AS (
        COALESCE(fob_amount, 0) + COALESCE(freight_amount, 0) + COALESCE(insurance_amount, 0)
    ) STORED,
    
    tariff_amount DECIMAL(15, 2),
    igv_amount DECIMAL(15, 2),
    customs_broker_fee DECIMAL(15, 2),
    local_transport DECIMAL(15, 2),
    warehouse_fees DECIMAL(15, 2),
    other_costs DECIMAL(15, 2),
    
    total_landed_cost DECIMAL(15, 2),
    
    calculated_at TIMESTAMP DEFAULT NOW(),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_landed_costs_po ON landed_costs(po_id);
```

---

## MIGRACIÓN DE FASE 0 A FASE 1

```sql
-- Script de migración: 001_fase_0_to_fase_1.sql

BEGIN;

-- Agregar nuevas tablas de Fase 1
-- (incluir todos los CREATE TABLE de arriba)

-- Agregar permisos para módulo de importaciones
INSERT INTO permissions (module, action, resource, description) VALUES
    ('imports', 'create', 'quotation_requests', 'Crear solicitudes de cotización'),
    ('imports', 'read', 'quotation_requests', 'Ver solicitudes de cotización'),
    ('imports', 'create', 'quotations', 'Registrar cotizaciones recibidas'),
    ('imports', 'create', 'purchase_orders', 'Crear órdenes de compra internacionales'),
    ('imports', 'approve', 'purchase_orders', 'Aprobar órdenes de compra'),
    ('imports', 'create', 'shipments', 'Registrar embarques'),
    ('imports', 'read', 'shipments', 'Ver tracking de embarques'),
    ('imports', 'create', 'customs_declarations', 'Crear DUA'),
    ('imports', 'read', 'landed_costs', 'Ver costeo de importaciones');

COMMIT;
```

---

## FASE 2: MÓDULO DE EXPORTACIONES

```sql
-- =====================================================
-- COTIZACIONES DE EXPORTACIÓN
-- =====================================================

CREATE TABLE export_quotations (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    quotation_number VARCHAR(50) NOT NULL,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    quotation_date DATE NOT NULL,
    valid_until DATE,
    currency_code VARCHAR(3) NOT NULL REFERENCES currencies(code),
    incoterm VARCHAR(10) NOT NULL,
    port_of_loading VARCHAR(100),
    destination_port VARCHAR(100),
    payment_terms TEXT,
    status VARCHAR(20) DEFAULT 'DRAFT',
    subtotal DECIMAL(15, 2) NOT NULL,
    total_amount DECIMAL(15, 2) NOT NULL,
    notes TEXT,
    created_by BIGINT NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, quotation_number),
    CONSTRAINT chk_export_quotation_status CHECK (status IN ('DRAFT', 'SENT', 'ACCEPTED', 'REJECTED', 'EXPIRED'))
);

-- =====================================================
-- ÓRDENES DE VENTA EXPORTACIÓN
-- =====================================================

CREATE TABLE sales_orders_export (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    so_number VARCHAR(50) NOT NULL,
    export_quotation_id BIGINT REFERENCES export_quotations(id),
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    so_date DATE NOT NULL,
    currency_code VARCHAR(3) NOT NULL REFERENCES currencies(code),
    incoterm VARCHAR(10) NOT NULL,
    port_of_loading VARCHAR(100),
    destination_port VARCHAR(100),
    payment_method VARCHAR(50),
    status VARCHAR(20) DEFAULT 'CONFIRMED',
    subtotal DECIMAL(15, 2) NOT NULL,
    total_amount DECIMAL(15, 2) NOT NULL,
    created_by BIGINT NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, so_number),
    CONSTRAINT chk_so_export_status CHECK (status IN ('CONFIRMED', 'IN_PRODUCTION', 'READY_TO_SHIP', 'SHIPPED', 'DELIVERED'))
);

-- =====================================================
-- DECLARACIÓN SIMPLIFICADA DE EXPORTACIÓN (DSE)
-- =====================================================

CREATE TABLE export_customs_declarations (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    dse_number VARCHAR(50),
    so_export_id BIGINT NOT NULL REFERENCES sales_orders_export(id),
    declaration_date DATE,
    fob_value_usd DECIMAL(15, 2),
    status VARCHAR(20) DEFAULT 'DRAFT',
    clearance_date DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT chk_dse_status CHECK (status IN ('DRAFT', 'SUBMITTED', 'CLEARED', 'REJECTED'))
);
```

---

## FASE 3: INVENTARIO AVANZADO

```sql
-- =====================================================
-- ALMACENES
-- =====================================================

CREATE TABLE warehouses (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    warehouse_code VARCHAR(20) NOT NULL,
    name VARCHAR(200) NOT NULL,
    warehouse_type VARCHAR(30) DEFAULT 'PRINCIPAL',
    address TEXT,
    manager_id BIGINT REFERENCES users(id),
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, warehouse_code),
    CONSTRAINT chk_warehouse_type CHECK (warehouse_type IN ('PRINCIPAL', 'TRANSIT', 'EXTERNAL'))
);

-- =====================================================
-- UBICACIONES DE ALMACÉN
-- =====================================================

CREATE TABLE warehouse_locations (
    id BIGSERIAL PRIMARY KEY,
    warehouse_id BIGINT NOT NULL REFERENCES warehouses(id) ON DELETE CASCADE,
    location_code VARCHAR(50) NOT NULL,
    aisle VARCHAR(10),
    rack VARCHAR(10),
    level VARCHAR(10),
    bin VARCHAR(10),
    active BOOLEAN DEFAULT true,
    
    UNIQUE(warehouse_id, location_code)
);

-- =====================================================
-- INVENTARIO
-- =====================================================

CREATE TABLE inventory (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    product_id BIGINT NOT NULL REFERENCES products(id),
    warehouse_id BIGINT NOT NULL REFERENCES warehouses(id),
    location_id BIGINT REFERENCES warehouse_locations(id),
    batch_number VARCHAR(50),
    serial_number VARCHAR(100),
    quantity DECIMAL(15, 4) NOT NULL DEFAULT 0,
    reserved_quantity DECIMAL(15, 4) DEFAULT 0,
    unit_cost DECIMAL(15, 4),
    manufacturing_date DATE,
    expiration_date DATE,
    last_movement_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, product_id, warehouse_id, COALESCE(batch_number, ''), COALESCE(serial_number, '')),
    CONSTRAINT chk_inventory_quantity CHECK (quantity >= 0)
);

CREATE INDEX idx_inventory_tenant_product ON inventory(tenant_id, product_id);
CREATE INDEX idx_inventory_warehouse ON inventory(warehouse_id);
CREATE INDEX idx_inventory_expiration ON inventory(expiration_date) WHERE expiration_date IS NOT NULL;

-- =====================================================
-- MOVIMIENTOS DE INVENTARIO
-- =====================================================

CREATE TABLE inventory_movements (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL REFERENCES tenants(id),
    movement_number VARCHAR(50) NOT NULL,
    movement_type VARCHAR(30) NOT NULL,
    movement_date TIMESTAMP NOT NULL DEFAULT NOW(),
    product_id BIGINT NOT NULL REFERENCES products(id),
    warehouse_id BIGINT NOT NULL REFERENCES warehouses(id),
    batch_number VARCHAR(50),
    serial_number VARCHAR(100),
    quantity DECIMAL(15, 4) NOT NULL,
    unit_cost DECIMAL(15, 4),
    reference_type VARCHAR(30),
    reference_id BIGINT,
    notes TEXT,
    created_by BIGINT NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(tenant_id, movement_number),
    CONSTRAINT chk_movement_type CHECK (movement_type IN ('RECEIPT', 'SHIPMENT', 'ADJUSTMENT', 'TRANSFER'))
);

CREATE INDEX idx_inventory_movements_tenant ON inventory_movements(tenant_id);
CREATE INDEX idx_inventory_movements_product ON inventory_movements(product_id);
CREATE INDEX idx_inventory_movements_date ON inventory_movements(movement_date DESC);
```

---

## RESUMEN DE CRECIMIENTO DE BD

| Fase | Nuevas Tablas | Total Acumulado | Complejidad |
|------|---------------|-----------------|-------------|
| Fase 0 | 16 tablas | 16 | Baja |
| Fase 1 | +12 tablas | 28 | Media |
| Fase 2 | +4 tablas | 32 | Media |
| Fase 3 | +5 tablas | 37 | Media-Alta |
| Fase 4 | +8 tablas | 45 | Alta |
| Fase 5-7 | +10 tablas | 55 | Alta |

---

## ESTRATEGIA DE MIGRACIONES (Serverpod)

```dart
// migrations/001_initial_schema.dart
import 'package:serverpod/serverpod.dart';

class Migration001 extends Migration {
  @override
  Future<void> upgrade(Database db) async {
    // Ejecutar SQL de Fase 0
    await db.execute('''
      -- Contenido del script de Fase 0
    ''');
  }
  
  @override
  Future<void> downgrade(Database db) async {
    // Rollback si es necesario
  }
}

// migrations/002_add_import_module.dart
class Migration002 extends Migration {
  @override
  Future<void> upgrade(Database db) async {
    // Ejecutar SQL de Fase 1
  }
}
```

---

**Este diseño incremental permite:**
- ✅ Empezar simple y crecer orgánicamente
- ✅ Cada fase es funcional y auto-contenida
- ✅ Migrar datos sin romper funcionalidad existente
- ✅ Testing más fácil (menos complejidad por fase)
- ✅ Deployment más seguro
