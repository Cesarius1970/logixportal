# LogixPortal - Plan Maestro de Desarrollo
## ERP de Comercio Exterior End-to-End para Perú

---

## VISIÓN EJECUTIVA

**LogixPortal** es un ERP web especializado en comercio exterior peruano que gestiona el ciclo completo de importación/exportación con trazabilidad total, cumplimiento normativo SUNAT/ADUANAS, y estándares internacionales de calidad.

### Propuesta de Valor
- **Trazabilidad completa**: Desde cotización hasta entrega final
- **Cumplimiento automático**: SUNAT, ADUANAS, ISO 28000, SOX
- **Visibilidad real-time**: Dashboard ejecutivo y operativo
- **Integración nativa**: Bancos, SUNAT, agentes de aduana, transportistas

---

## ESTRATEGIA DE DESARROLLO INCREMENTAL

### Principio Rector: "Walking Skeleton" → MVP → Módulos Incrementales

**No construiremos todo de una vez.** Empezaremos con un sistema funcional mínimo (Walking Skeleton) que:
1. Tiene arquitectura completa pero funcionalidad mínima
2. Se puede mostrar, usar y validar con usuarios reales
3. Sirve de base sólida para todos los módulos futuros

---

## FASES DE DESARROLLO

### 📍 FASE 0: FUNDACIÓN (Semanas 1-4) - 1 MES
**Entregable**: Walking Skeleton funcional

#### Objetivos
- Arquitectura base desplegada
- Autenticación multi-tenant
- UI/UX base con Material Design 3
- CI/CD pipeline funcionando
- Base de datos core

#### Funcionalidades Mínimas
1. **Login/Registro** con validación RUC (SUNAT API)
2. **Dashboard básico** con métricas dummy
3. **Maestro de Productos** (CRUD simple)
4. **Maestro de Proveedores** (CRUD simple)
5. **Gestión de Usuarios y Roles**

#### Stack Técnico Confirmado
```yaml
Frontend:
  - Flutter Web (Material Design 3)
  - Riverpod 3 (state management)
  - Go Router (navegación)
  - Freezed (modelos inmutables)
  - Dio (HTTP client)

Backend:
  - Serverpod 3 (Dart backend framework)
  - PostgreSQL 15+ (base de datos principal)
  - Redis 7 (caché y sesiones)
  
Infraestructura:
  - Docker / Docker Compose
  - Nginx (reverse proxy)
  - GitHub Actions (CI/CD)
  - Railway/Render (hosting inicial)

Integraciones Iniciales:
  - SUNAT API (validación RUC)
  - Email (Resend/SendGrid)
```

#### Criterios de Éxito Fase 0
- [x] Usuario puede registrar empresa con RUC válido
- [x] Usuario puede hacer login y ver dashboard
- [x] Usuario puede crear/editar productos
- [x] Usuario puede crear/editar proveedores
- [x] Sistema desplegado en ambiente DEV accesible por URL

#### Riesgos y Mitigaciones
| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Curva de aprendizaje Serverpod | Media | Alto | Completar tutorial oficial, pair programming |
| Integraciones SUNAT inestables | Alta | Medio | Mocks para desarrollo, manejo de errores robusto |
| Problemas de deployment | Media | Medio | Usar Railway/Render con templates predefinidos |

---

### 📍 FASE 1: MVP - MÓDULO IMPORTACIONES (Semanas 5-12) - 2 MESES
**Entregable**: Sistema funcional para gestionar proceso completo de importación

#### Objetivos
- Proceso de importación end-to-end operativo
- Integración con agentes de aduana
- Generación de documentación básica
- Costeo de importación (CIF calculator)

#### Funcionalidades Core
1. **Cotizaciones de Importación**
   - Crear solicitud de cotización a proveedores
   - Recibir y comparar cotizaciones
   - Aprobar cotización

2. **Órdenes de Compra Internacional**
   - Generar OC desde cotización aprobada
   - Tracking de confirmación con proveedor
   - Gestión de pagos y cartas de crédito

3. **Gestión de Embarques**
   - Registro de embarque (FCL/LCL)
   - Tracking de contenedor (integración con navieras)
   - Documentos: BL, Invoice, Packing List

4. **Proceso Aduanero**
   - Generación de DUA (Declaración Única de Aduanas)
   - Integración con agente de aduana
   - Pago de aranceles e IGV
   - Levante de mercancía

5. **Costeo de Importación**
   - Cálculo automático: FOB + Flete + Seguro = CIF
   - Aranceles (% según partida arancelaria)
   - IGV (18%)
   - Gastos operativos (agente, almacén, transporte)
   - **Costo de importación unitario**

6. **Recepción de Mercancía**
   - Ingreso a almacén local
   - Control de calidad
   - Generación de lote/serie

#### Modelos de Datos Fase 1

```sql
-- IMPORTACIONES
quotation_requests
quotations
purchase_orders_international
shipments
containers
customs_declarations (DUA)
import_costs
landed_cost_breakdown

-- DOCUMENTOS
commercial_invoices
packing_lists
bills_of_lading
certificates_origin
```

#### Integraciones Fase 1
- API SUNAT (validaciones)
- Navieras (tracking básico - manual inicialmente)
- Agentes de aduana (email/PDF inicialmente)
- Bancos (registro manual de pagos)

#### Criterios de Éxito Fase 1
- [ ] Usuario puede crear cotización y recibir respuestas
- [ ] Usuario puede generar OC internacional
- [ ] Usuario puede registrar embarque y trackear contenedor
- [ ] Sistema calcula costo de importación automáticamente
- [ ] Usuario puede generar DUA con datos correctos
- [ ] Mercancía ingresada queda en inventario con costo correcto

---

### 📍 FASE 2: MÓDULO EXPORTACIONES (Semanas 13-18) - 1.5 MESES
**Entregable**: Proceso completo de exportación

#### Funcionalidades Core
1. Cotizaciones de Exportación (con INCOTERMS)
2. Órdenes de Venta Exportación
3. Producción/Preparación de mercancía
4. Booking con naviera/aerolínea
5. Generación de documentos: Factura Comercial, Packing List, Certificados
6. Declaración de Exportación (DSE)
7. Embarque y tracking
8. Cobranza internacional (cartas de crédito, cobranzas)

#### Modelos de Datos Fase 2
```sql
export_quotations
sales_orders_export
export_shipments
export_customs_declarations (DSE)
export_documents
letters_of_credit
```

---

### 📍 FASE 3: INVENTARIO AVANZADO Y LOGÍSTICA (Semanas 19-24) - 1.5 MESES

#### Funcionalidades Core
1. **Gestión Multi-Almacén**
   - Transferencias entre almacenes
   - Ubicaciones (pasillo-rack-nivel-bin)
   - Códigos de barra / QR

2. **Control de Lotes y Series**
   - Trazabilidad FEFO/FIFO
   - Fechas de vencimiento
   - Gestión de cuarentena

3. **Inventarios Cíclicos**
   - Programación ABC
   - Ajustes y conciliación
   - Auditoría de diferencias

4. **Distribución Local**
   - Planificación de rutas
   - Guías de remisión electrónicas (GRE)
   - Tracking GPS (integración básica)
   - POD (Proof of Delivery)

#### Modelos de Datos Fase 3
```sql
warehouse_locations
inventory_batches
inventory_serials
cycle_counts
stock_adjustments
delivery_routes
electronic_waybills (GRE)
proof_of_delivery
```

---

### 📍 FASE 4: FINANZAS Y FACTURACIÓN ELECTRÓNICA (Semanas 25-30) - 1.5 MESES

#### Funcionalidades Core
1. **Facturación Electrónica SUNAT**
   - Facturas (01)
   - Boletas (03)
   - Notas de Crédito/Débito (07/08)
   - Firmado digital con certificado
   - Envío a OSE/SUNAT
   - Validación CDR

2. **Cuentas por Cobrar**
   - Línea de crédito clientes
   - Seguimiento de cobranza
   - Estados de cuenta
   - Provisión de incobrables

3. **Cuentas por Pagar**
   - Programación de pagos
   - Conciliación con proveedores
   - Retenciones y percepciones

4. **Libros Electrónicos PLE**
   - Registro de Ventas
   - Registro de Compras
   - Formato 5.1 SUNAT

5. **Flujo de Caja**
   - Proyección semanal/mensual
   - Conciliación bancaria

#### Modelos de Datos Fase 4
```sql
invoices
credit_notes
debit_notes
accounts_receivable
payment_schedules
accounts_payable
ple_sales_register
ple_purchase_register
cash_flow_projections
bank_reconciliations
```

---

### 📍 FASE 5: PLANIFICACIÓN Y ANALYTICS (Semanas 31-36) - 1.5 MESES

#### Funcionalidades Core
1. **Forecasting con ML**
   - Predicción de demanda
   - Estacionalidad
   - Recomendaciones de reorden

2. **MRP (Material Requirement Planning)**
   - Explosión de BOM
   - Cálculo de necesidades
   - Sugerencias de compra

3. **Dashboards Ejecutivos**
   - KPIs financieros
   - KPIs operativos (OTD, fill rate, inventory turnover)
   - Análisis ABC/XYZ

4. **Reportería Avanzada**
   - Power BI embedded / Metabase
   - Reportes personalizables
   - Exportación Excel/PDF

---

### 📍 FASE 6: CALIDAD Y COMPLIANCE (Semanas 37-40) - 1 MES

#### Funcionalidades Core
1. Control de calidad de insumos/productos
2. NCR (No conformidades)
3. Auditorías internas
4. Trazabilidad sanitaria (DIGESA)
5. Certificaciones (ISO 9001, ISO 28000)

---

### 📍 FASE 7: INTEGRACIONES AVANZADAS (Semanas 41-44) - 1 MES

#### Integraciones
1. **Bancos**: BCP, Interbank, BBVA (APIs bancarias)
2. **Pasarelas de pago**: Niubiz, Mercado Pago
3. **WMS externos**: Integración bidireccional
4. **CRM**: Salesforce, Zoho
5. **Transportistas**: GPS tracking real-time
6. **Marketplaces**: Integración con canales de venta

---

## TIMELINE CONSOLIDADO

```
Total de Desarrollo: 44 semanas = ~11 meses

Fase 0: Fundación                    [███████] Mes 1
Fase 1: MVP Importaciones            [██████████████] Meses 2-3
Fase 2: Exportaciones                [███████████] Meses 4-5
Fase 3: Inventario Avanzado          [███████████] Meses 6-7
Fase 4: Finanzas y Facturación       [███████████] Meses 8-9
Fase 5: Planificación y Analytics    [███████████] Meses 10-11
Fase 6: Calidad y Compliance         [███████] Mes 11
Fase 7: Integraciones Avanzadas      [███████] Mes 11

Post-lanzamiento: Mantenimiento y nuevas features
```

### Timeline Realista con Buffer
- **Estimación optimista**: 11 meses
- **Estimación realista**: 14 meses (con 25% de buffer)
- **Estimación pesimista**: 18 meses (con imprevistos mayores)

**Recomendación**: Planificar para **14 meses** y apuntar a completar en 11-12 meses.

---

## EQUIPO NECESARIO POR FASE

### Fase 0 (Fundación)
- 1 Full-stack Developer (Flutter + Serverpod) - **Esencial**
- 1 DevOps/Infrastructure (part-time) - 50% dedicación

### Fase 1-2 (MVP + Exportaciones)
- 2 Full-stack Developers
- 1 UI/UX Designer (part-time) - 30% dedicación
- 1 QA Tester (desde Fase 1)
- 1 Product Owner / Business Analyst

### Fase 3-7 (Módulos Avanzados)
- 3 Full-stack Developers
- 1 Frontend Specialist (Flutter)
- 1 Backend Specialist (Serverpod/PostgreSQL)
- 1 QA Tester
- 1 DevOps Engineer (part-time aumenta a full-time)
- 1 Data Analyst / ML Engineer (Fase 5)

### Equipo de Validación
- 3-5 usuarios piloto (empresas importadoras/exportadoras reales)
- 1 Experto en comercio exterior (asesor)
- 1 Contador especializado en normativa SUNAT

---

## PRESUPUESTO ESTIMADO (REFERENCIAL)

### Costos de Personal (14 meses)
| Rol | Cantidad | Costo Mensual (USD) | Total (USD) |
|-----|----------|---------------------|-------------|
| Senior Full-stack Dev | 2 | $4,500 | $126,000 |
| Mid-level Full-stack Dev | 1 | $3,000 | $42,000 |
| UI/UX Designer | 1 (PT) | $1,500 | $21,000 |
| QA Tester | 1 | $2,500 | $35,000 |
| DevOps Engineer | 1 (PT) | $2,000 | $28,000 |
| Product Owner | 1 | $4,000 | $56,000 |
| **SUBTOTAL PERSONAL** | | | **$308,000** |

### Costos de Infraestructura (14 meses)
| Servicio | Costo Mensual (USD) | Total (USD) |
|----------|---------------------|-------------|
| Hosting (Railway/AWS) | $200 | $2,800 |
| Database (PostgreSQL managed) | $100 | $1,400 |
| Redis Cache | $50 | $700 |
| S3 Storage | $50 | $700 |
| CI/CD (GitHub Actions) | $50 | $700 |
| Monitoring (Sentry, Datadog) | $100 | $1,400 |
| Email Service | $30 | $420 |
| **SUBTOTAL INFRA** | | **$8,120** |

### Costos de Licencias y Servicios
| Ítem | Costo (USD) |
|------|-------------|
| Certificado Digital SUNAT | $150/año |
| APIs de Integración | $2,000 |
| Diseño UI/UX (Figma Pro) | $300 |
| Testing Tools | $500 |
| **SUBTOTAL SERVICIOS** | **$2,950** |

### **PRESUPUESTO TOTAL: ~$320,000 USD** (14 meses)

**Nota**: Esta es una estimación para un equipo externo. Costos varían según:
- Si desarrollas in-house vs outsourcing
- Ubicación geográfica del equipo
- Nivel de seniority requerido
- Uso de cloud managed services vs self-hosted

---

## ESTRATEGIA DE VALIDACIÓN CON USUARIOS

### Momentos de Validación

1. **Post Fase 0**: Validar UX base, navegación, maestros
2. **Post Fase 1 (MVP)**: Validación crítica del flujo de importaciones
3. **Post Fase 2**: Validación de exportaciones
4. **Cada nueva feature**: Testing con usuario piloto antes de release

### Empresas Piloto Ideales
- 1 importadora pequeña (10-50 contenedores/año)
- 1 importadora mediana (50-200 contenedores/año)
- 1 empresa mixta (importa y exporta)

---

## CRITERIOS DE ÉXITO DEL PROYECTO

### Técnicos
- ✅ Uptime > 99.5%
- ✅ Tiempo de respuesta < 2 segundos (p95)
- ✅ 0 vulnerabilidades críticas (OWASP Top 10)
- ✅ Test coverage > 80%

### Funcionales
- ✅ 100% de comprobantes electrónicos aceptados por SUNAT
- ✅ Trazabilidad completa de todas las operaciones
- ✅ Auditoría completa (quién, qué, cuándo)
- ✅ Cálculo correcto de costos de importación/exportación

### Negocio
- ✅ Tiempo de cierre mensual < 5 días
- ✅ Reducción de 50% en tiempo de procesamiento de importaciones
- ✅ 95% de satisfacción de usuarios piloto
- ✅ ROI positivo en 18 meses

---

## SIGUIENTES PASOS INMEDIATOS

### Semana 1-2: Preparación
1. ✅ **Definir alcance exacto de Fase 0** (este documento)
2. ⬜ Crear guías de entrevistas para usuarios
3. ⬜ Diseñar esquema de base de datos incremental
4. ⬜ Configurar repositorio y estructura de proyecto
5. ⬜ Configurar ambientes (DEV, STAGING, PROD)

### Semana 3-4: Desarrollo Fase 0
6. ⬜ Implementar autenticación y multi-tenancy
7. ⬜ Implementar CRUD de productos y proveedores
8. ⬜ Implementar dashboard base
9. ⬜ Primera validación con usuario

---

## DOCUMENTOS COMPLEMENTARIOS

Los siguientes documentos detallan cada aspecto:

1. **`02_guias_entrevistas.md`** - Guías estructuradas para entrevistar usuarios
2. **`03_diseño_base_datos_incremental.md`** - Esquema de BD por fases
3. **`04_arquitectura_tecnica.md`** - Detalles técnicos de la arquitectura
4. **`05_manual_desarrollo.md`** - Guía para desarrolladores
5. **`06_plan_testing_qa.md`** - Estrategia de testing y QA

---

**Versión**: 1.0  
**Fecha**: Marzo 2026  
**Autor**: Equipo LogixPortal  
**Estado**: Aprobado para Ejecución
