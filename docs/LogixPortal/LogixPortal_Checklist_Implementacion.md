# LogixPortal - Checklist de Implementación
## Control de Progreso por Fases

---

## 📌 Cómo Usar Este Checklist

- ✅ Marca con `[x]` las tareas completadas
- 🔄 En progreso: Agregar nota con fecha
- ⏸️ Bloqueado: Agregar razón y dependencia
- 📝 Agregar notas en la columna de Observaciones

---

## FASE 0: FUNDAMENTOS (Semanas 1-5)

### Setup Inicial

- [ ] **0.1** Reclutamiento de equipo completo
  - [ ] Backend Developer (Dart/Serverpod) #1
  - [ ] Backend Developer (Dart/Serverpod) #2
  - [ ] Frontend Developer (Flutter) #1
  - [ ] Frontend Developer (Flutter) #2
  - [ ] UI/UX Designer
  - [ ] QA Engineer
  - [ ] DevOps Engineer (part-time)
  - [ ] Product Owner
  
- [ ] **0.2** Infraestructura base
  - [ ] Servidor de desarrollo (AWS/GCP/Azure)
  - [ ] PostgreSQL 15+ con PostGIS instalado
  - [ ] Redis para cache
  - [ ] Docker + Docker Compose configurado
  - [ ] CI/CD pipeline (GitHub Actions/GitLab CI)
  - [ ] Monitoring (Prometheus + Grafana)
  
- [ ] **0.3** Proyecto Serverpod inicial
  - [ ] `serverpod create logixportal` ejecutado
  - [ ] Estructura de carpetas organizada (Feature-First)
  - [ ] Git repository creado y configurado
  - [ ] `.gitignore` configurado
  - [ ] README.md con instrucciones de setup

### Autenticación y Seguridad

- [ ] **0.4** Sistema de autenticación
  - [ ] Entidad `User` en serverpod.yaml
  - [ ] Entidad `Role` en serverpod.yaml
  - [ ] Entidad `Permission` en serverpod.yaml
  - [ ] JWT token generation
  - [ ] Refresh token logic
  - [ ] Password hashing (bcrypt/argon2)
  - [ ] Login endpoint
  - [ ] Logout endpoint
  - [ ] Refresh token endpoint
  
- [ ] **0.5** Autenticación de 2 factores (2FA)
  - [ ] Generación de secret TOTP
  - [ ] QR code para Google Authenticator
  - [ ] Verificación de código 2FA
  - [ ] Backup codes
  
- [ ] **0.6** RBAC (Role-Based Access Control)
  - [ ] Middleware de autorización
  - [ ] Decoradores para endpoints con permisos requeridos
  - [ ] Matriz de roles y permisos definida
  - [ ] Roles del sistema creados (Admin, Manager, User)

### Multi-Empresa

- [ ] **0.7** Entidades de empresa
  - [ ] Entidad `Company` en serverpod.yaml
  - [ ] Campo `company_id` en todas las tablas transaccionales
  - [ ] Row-Level Security (RLS) configurado en PostgreSQL
  - [ ] Middleware de validación de `company_id`
  
- [ ] **0.8** Gestión de empresas
  - [ ] CRUD de empresas (Create, Read, Update, Delete)
  - [ ] Validación de RUC con SUNAT API
  - [ ] Asignación de usuarios a empresas
  - [ ] Cambio de empresa activa por usuario

### Internacionalización

- [ ] **0.9** i18n configurado
  - [ ] `flutter_localizations` agregado
  - [ ] `app_es.arb` (Español - default)
  - [ ] `app_en.arb` (Inglés)
  - [ ] `intl_utils` configurado
  - [ ] Mensajes de error en español
  - [ ] Selector de idioma en UI

### UI/UX Base

- [ ] **0.10** Tema Material 3
  - [ ] Color scheme corporativo definido
  - [ ] `ThemeData` configurado
  - [ ] Tipografía personalizada
  - [ ] Componentes reutilizables (buttons, cards, inputs)
  
- [ ] **0.11** Navegación
  - [ ] `go_router` configurado
  - [ ] Rutas principales definidas
  - [ ] Guards de autenticación
  - [ ] Deep linking configurado

### Testing Fase 0

- [ ] **0.12** Tests unitarios
  - [ ] Auth service tests
  - [ ] JWT token tests
  - [ ] Company CRUD tests
  - [ ] Cobertura >80%
  
- [ ] **0.13** Tests de integración
  - [ ] Login flow E2E
  - [ ] Company creation E2E
  - [ ] Multi-empresa E2E

**Entregables Fase 0:**
- [ ] Documento de Arquitectura Técnica
- [ ] Plan de Deployment
- [ ] Guía de Instalación Local
- [ ] Matriz de Roles y Permisos
- [ ] Diseño de Tema UI (Figma)
- [ ] Guía de Estilo de Código

---

## FASE 1: MÓDULO CORE (Semanas 6-12)

### Maestro de Productos

- [ ] **1.1** Entidades de productos
  - [ ] `Product` en serverpod.yaml
  - [ ] `ProductCategory` en serverpod.yaml
  - [ ] `UnitOfMeasure` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **1.2** CRUD de productos
  - [ ] Endpoint `createProduct`
  - [ ] Endpoint `updateProduct`
  - [ ] Endpoint `deleteProduct` (soft delete)
  - [ ] Endpoint `getProductById`
  - [ ] Endpoint `listProducts` con filtros
  - [ ] Endpoint `searchProducts` (full-text search)
  
- [ ] **1.3** UI de productos
  - [ ] Pantalla de lista de productos
  - [ ] Pantalla de detalle de producto
  - [ ] Formulario de creación/edición
  - [ ] Búsqueda con autocompletado
  - [ ] Importación masiva desde Excel
  - [ ] Exportación a Excel
  
- [ ] **1.4** Categorías de productos
  - [ ] CRUD de categorías
  - [ ] Árbol de categorías (padre-hijo)
  - [ ] UI de gestión de categorías

### Maestro de Partners

- [ ] **1.5** Entidades de partners
  - [ ] `BusinessPartner` en serverpod.yaml
  - [ ] Tipos: SUPPLIER, CUSTOMER, BOTH
  - [ ] Migraciones aplicadas
  
- [ ] **1.6** CRUD de partners
  - [ ] Endpoint `createPartner`
  - [ ] Endpoint `updatePartner`
  - [ ] Endpoint `deletePartner` (soft delete)
  - [ ] Endpoint `getPartnerById`
  - [ ] Endpoint `listPartners` con filtros
  
- [ ] **1.7** Validaciones
  - [ ] Validación de RUC con SUNAT API
  - [ ] Validación de DNI con RENIEC API
  - [ ] Detección de duplicados
  
- [ ] **1.8** UI de partners
  - [ ] Pantalla de lista de partners
  - [ ] Pantalla de detalle de partner
  - [ ] Formulario de creación/edición
  - [ ] Búsqueda avanzada
  - [ ] Importación desde Excel

### Almacenes

- [ ] **1.9** Entidades de almacenes
  - [ ] `Warehouse` en serverpod.yaml
  - [ ] `WarehouseLocation` en serverpod.yaml
  - [ ] Campo de geolocalización (PostGIS)
  - [ ] Migraciones aplicadas
  
- [ ] **1.10** CRUD de almacenes
  - [ ] Endpoint `createWarehouse`
  - [ ] Endpoint `updateWarehouse`
  - [ ] Endpoint `deleteWarehouse`
  - [ ] Endpoint `getWarehouseById`
  - [ ] Endpoint `listWarehouses`
  
- [ ] **1.11** Ubicaciones de almacén
  - [ ] CRUD de ubicaciones (rack/shelf/bin)
  - [ ] Jerarquía de ubicaciones
  - [ ] Búsqueda de ubicaciones
  
- [ ] **1.12** Mapas con PostGIS
  - [ ] Integración de `flutter_map`
  - [ ] Visualización de almacenes en mapa
  - [ ] Cálculo de distancias
  - [ ] Rutas entre almacenes

### Inventarios

- [ ] **1.13** Entidades de inventario
  - [ ] `InventoryBalance` en serverpod.yaml
  - [ ] `InventoryMovement` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **1.14** Movimientos de inventario
  - [ ] Endpoint `createMovement` (Receipt, Shipment, Adjustment, Transfer)
  - [ ] Endpoint `listMovements` con filtros
  - [ ] Endpoint `getMovementById`
  - [ ] Actualización automática de balances
  
- [ ] **1.15** Consultas de inventario
  - [ ] Endpoint `getInventoryBalance` por producto/almacén
  - [ ] Endpoint `getInventoryByWarehouse`
  - [ ] Endpoint `getInventoryByProduct`
  - [ ] Endpoint `getLowStockProducts`
  
- [ ] **1.16** UI de inventarios
  - [ ] Pantalla de consulta de stock
  - [ ] Kardex de movimientos
  - [ ] Registro de ajustes de inventario
  - [ ] Registro de transferencias
  - [ ] Alertas de stock mínimo

### Tipo de Cambio

- [ ] **1.17** Integración SBS
  - [ ] Endpoint para consultar tipo de cambio del día
  - [ ] Caché en Redis de tipos de cambio
  - [ ] Historial de tipos de cambio
  - [ ] UI de consulta de tipo de cambio

**Entregables Fase 1:**
- [ ] Modelo de Datos Core (DER)
- [ ] API Spec - Productos
- [ ] API Spec - Partners
- [ ] API Spec - Almacenes e Inventarios
- [ ] Manual de Usuario - Maestros
- [ ] Wireframes de Pantallas Core
- [ ] Reporte de Testing E2E Fase 1
- [ ] Plan de Migración de Datos

---

## FASE 2: MÓDULO DE COMPRAS (Semanas 13-19)

### Órdenes de Compra

- [ ] **2.1** Entidades de compras
  - [ ] `PurchaseOrder` en serverpod.yaml
  - [ ] `PurchaseOrderItem` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **2.2** CRUD de POs
  - [ ] Endpoint `createPurchaseOrder`
  - [ ] Endpoint `updatePurchaseOrder`
  - [ ] Endpoint `approvePurchaseOrder`
  - [ ] Endpoint `cancelPurchaseOrder`
  - [ ] Endpoint `listPurchaseOrders` con filtros
  
- [ ] **2.3** Flujo de aprobación
  - [ ] Aprobación multi-nivel (requester → supervisor → manager)
  - [ ] Endpoint `submitForApproval`
  - [ ] Endpoint `approve`
  - [ ] Endpoint `reject`
  - [ ] Notificaciones por email
  
- [ ] **2.4** UI de POs
  - [ ] Pantalla de lista de POs
  - [ ] Formulario de creación de PO
  - [ ] Vista de detalle de PO
  - [ ] Workflow de aprobación
  - [ ] Dashboard de POs pendientes

### Recepciones de Mercancía

- [ ] **2.5** Entidades de recepciones
  - [ ] `GoodsReceipt` en serverpod.yaml
  - [ ] `GoodsReceiptItem` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **2.6** CRUD de recepciones
  - [ ] Endpoint `createGoodsReceipt`
  - [ ] Endpoint `completeGoodsReceipt`
  - [ ] Endpoint `cancelGoodsReceipt`
  - [ ] Endpoint `listGoodsReceipts`
  - [ ] Generación automática de movimientos de inventario
  
- [ ] **2.7** Control de calidad
  - [ ] Estados de calidad (APPROVED, REJECTED, QUARANTINE)
  - [ ] Registro de rechazos
  - [ ] Generación de NCRs (No Conformidades)
  
- [ ] **2.8** UI de recepciones
  - [ ] Pantalla de registro de recepción
  - [ ] Asociación con PO
  - [ ] Registro de discrepancias
  - [ ] Reporte de recepciones

### Costos de Importación

- [ ] **2.9** Cálculo de costos
  - [ ] Endpoint `calculateImportCosts`
  - [ ] FOB + Freight + Insurance = CIF
  - [ ] Derechos arancelarios
  - [ ] IGV de importación
  - [ ] Otros gastos (handling, storage)
  
- [ ] **2.10** UI de costos
  - [ ] Calculadora de costos de importación
  - [ ] Desglose detallado
  - [ ] Comparación con presupuesto

**Entregables Fase 2:**
- [ ] Modelo de Datos - Procurement
- [ ] API Spec - Purchase Orders
- [ ] API Spec - Goods Receipts
- [ ] Flujo de Aprobación de POs (Diagrama)
- [ ] Manual de Usuario - Compras
- [ ] Template PDF - Orden de Compra
- [ ] Reporte de Performance Testing

---

## FASE 3: VENTAS Y FACTURACIÓN (Semanas 20-28)

### Órdenes de Venta

- [ ] **3.1** Entidades de ventas
  - [ ] `SalesOrder` en serverpod.yaml
  - [ ] `SalesOrderItem` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **3.2** CRUD de SOs
  - [ ] Endpoint `createSalesOrder`
  - [ ] Endpoint `updateSalesOrder`
  - [ ] Endpoint `confirmSalesOrder`
  - [ ] Endpoint `cancelSalesOrder`
  - [ ] Endpoint `listSalesOrders`
  
- [ ] **3.3** Reserva de inventario
  - [ ] Reserva automática al confirmar SO
  - [ ] Liberación de reserva al cancelar
  - [ ] Verificación de disponibilidad

### Facturación Electrónica SUNAT

- [ ] **3.4** Entidades de facturación
  - [ ] `ElectronicDocument` en serverpod.yaml
  - [ ] `ElectronicDocumentItem` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **3.5** Generación de XML UBL 2.1
  - [ ] Librería para generar XML UBL
  - [ ] Template de Factura (01)
  - [ ] Template de Boleta (03)
  - [ ] Template de Nota de Crédito (07)
  - [ ] Template de Nota de Débito (08)
  
- [ ] **3.6** Firma digital
  - [ ] Integración con certificado digital (.pfx)
  - [ ] Firmado de XML
  - [ ] Generación de hash
  
- [ ] **3.7** Integración OSE/SUNAT
  - [ ] Endpoint `sendToSunat`
  - [ ] Procesamiento de CDR
  - [ ] Manejo de errores SUNAT
  - [ ] Reintentos automáticos
  
- [ ] **3.8** PDF con QR
  - [ ] Generación de PDF
  - [ ] QR Code estándar SUNAT
  - [ ] Código de barras
  - [ ] Template profesional
  
- [ ] **3.9** UI de facturación
  - [ ] Formulario de emisión de factura
  - [ ] Vista de detalle de comprobante
  - [ ] Envío por email al cliente
  - [ ] Reenvío a SUNAT
  - [ ] Reporte de comprobantes emitidos

### Libros Electrónicos PLE

- [ ] **3.10** Generación de PLE
  - [ ] Registro de Ventas (14.1)
  - [ ] Registro de Compras (8.1)
  - [ ] Formato .txt según SUNAT
  - [ ] Nombrado de archivos correcto
  
- [ ] **3.11** UI de libros electrónicos
  - [ ] Generación de PLE por período
  - [ ] Descarga de archivos
  - [ ] Validación de formato

**Entregables Fase 3:**
- [ ] Modelo de Datos - Ventas e Invoicing
- [ ] API Spec - Sales Orders
- [ ] API Spec - Electronic Documents
- [ ] Guía de Integración SUNAT
- [ ] Especificación XML UBL 2.1
- [ ] Manual de Facturación Electrónica
- [ ] Template PDF - Factura con QR
- [ ] Certificación OSE (si aplica)

---

## FASE 4: COMERCIO EXTERIOR (Semanas 29-37)

### Despachos Aduaneros

- [ ] **4.1** Entidades de customs
  - [ ] `CustomsDeclaration` en serverpod.yaml
  - [ ] Migraciones aplicadas
  
- [ ] **4.2** DUA (Importación)
  - [ ] Endpoint `createImportDeclaration`
  - [ ] Cálculo de derechos e impuestos
  - [ ] Integración VUCE (si disponible)
  
- [ ] **4.3** DSI (Exportación)
  - [ ] Endpoint `createExportDeclaration`
  - [ ] Registro en VUCE
  
- [ ] **4.4** UI de despachos
  - [ ] Formulario de DUA/DSI
  - [ ] Gestión de documentos adjuntos
  - [ ] Seguimiento de estado

### Tracking de Contenedores

- [ ] **4.5** Integración con navieras
  - [ ] API de tracking genérica
  - [ ] Parseo de datos de tracking
  - [ ] Alertas de llegada
  
- [ ] **4.6** UI de tracking
  - [ ] Mapa de ruta de contenedor
  - [ ] Timeline de eventos
  - [ ] Notificaciones push

**Entregables Fase 4:**
- [ ] Modelo de Datos - Customs
- [ ] API Spec - Import/Export Operations
- [ ] Guía de Integración VUCE
- [ ] Catálogo de Partidas Arancelarias
- [ ] Manual de Despachos Aduaneros
- [ ] Calculadora de Costos de Importación
- [ ] Reporte de Integración con APIs Navieras

---

## FASE 5: LOGÍSTICA Y TRANSPORTE (Semanas 38-45)

### Rutas y Transporte

- [ ] **5.1** Entidades de logística
  - [ ] `Route` en serverpod.yaml
  - [ ] `Shipment` en serverpod.yaml
  - [ ] `Vehicle` en serverpod.yaml
  - [ ] `Driver` en serverpod.yaml
  
- [ ] **5.2** Planificación de rutas
  - [ ] Algoritmo de optimización (PostGIS)
  - [ ] Endpoint `createRoute`
  - [ ] Endpoint `optimizeRoute`
  
- [ ] **5.3** GPS Tracking
  - [ ] Integración con dispositivos GPS
  - [ ] Actualización de posición en tiempo real
  - [ ] Mapa de seguimiento

### Guías de Remisión

- [ ] **5.4** Entidades de guías
  - [ ] `DeliveryGuide` en serverpod.yaml
  - [ ] Generación de XML para guía electrónica
  
- [ ] **5.5** UI de transporte
  - [ ] Creación de rutas
  - [ ] Asignación de vehículos y choferes
  - [ ] Dashboard de operaciones logísticas

**Entregables Fase 5:**
- [ ] Modelo de Datos - Logistics
- [ ] API Spec - Routes & Shipments
- [ ] Guía de Configuración PostGIS
- [ ] Algoritmo de Optimización de Rutas
- [ ] Manual de Gestión de Transporte
- [ ] App Móvil - Guía del Chofer

---

## FASE 6: ANALYTICS Y FORECASTING (Semanas 46-54)

### Dashboards

- [ ] **6.1** Dashboard ejecutivo
  - [ ] KPIs generales (ventas, compras, inventario)
  - [ ] Gráficos en tiempo real
  - [ ] Filtros por período
  
- [ ] **6.2** Dashboard de inventarios
  - [ ] Rotación de inventario
  - [ ] Productos obsoletos
  - [ ] Alertas de vencimiento
  
- [ ] **6.3** Dashboard de ventas
  - [ ] Análisis de tendencias
  - [ ] Productos más vendidos
  - [ ] Clientes top

### Reportes

- [ ] **6.4** Motor de reportes
  - [ ] Generador de reportes dinámico
  - [ ] Filtros personalizables
  - [ ] Exportación a Excel/PDF
  
- [ ] **6.5** Reportes predefinidos
  - [ ] Reporte de compras por proveedor
  - [ ] Reporte de ventas por cliente
  - [ ] Reporte de inventario valorizado
  - [ ] Estado de cuenta de clientes/proveedores

### Forecasting con ML

- [ ] **6.6** Modelo de pronóstico
  - [ ] Integración con Prophet/ARIMA
  - [ ] Entrenamiento con datos históricos
  - [ ] Endpoint `getForecast`
  
- [ ] **6.7** UI de forecasting
  - [ ] Visualización de pronósticos
  - [ ] Ajuste de parámetros
  - [ ] Comparación real vs forecast

**Entregables Fase 6:**
- [ ] Modelo de Datos - Analytics
- [ ] Catálogo de KPIs
- [ ] Especificación de Dashboards
- [ ] Documentación Técnica - ML Forecasting
- [ ] Manual de Reportería Avanzada
- [ ] Guía de Exportación de Datos

---

## 🎯 HITOS DE VALIDACIÓN

### Hito 1: MVP Fase 0 (Semana 5)
- [ ] Demo con stakeholders
- [ ] Login funcional con 2FA
- [ ] Multi-empresa funcionando
- [ ] Aprobación para continuar a Fase 1

### Hito 2: MVP Fase 1 (Semana 12)
- [ ] Demo de maestros
- [ ] Productos, partners, almacenes, inventarios funcionando
- [ ] Usuarios piloto (3-5) probando el sistema
- [ ] Feedback recopilado y priorizado

### Hito 3: MVP Fase 2 (Semana 19)
- [ ] Demo de compras
- [ ] Flujo completo PO → Recepción → Inventario
- [ ] Primeras POs reales procesadas
- [ ] Aprobación para avanzar a ventas

### Hito 4: MVP Fase 3 (Semana 28)
- [ ] Demo de facturación
- [ ] Primera factura electrónica aceptada por SUNAT
- [ ] PLE generado correctamente
- [ ] Certificación OSE obtenida (si aplica)

### Hito 5: MVP Fase 4 (Semana 37)
- [ ] Demo de comercio exterior
- [ ] Primera DUA registrada
- [ ] Tracking de contenedor funcionando
- [ ] Integración VUCE validada

### Hito 6: MVP Fase 5 (Semana 45)
- [ ] Demo de logística
- [ ] Rutas optimizadas con PostGIS
- [ ] GPS tracking en tiempo real
- [ ] Guías de remisión electrónicas

### Hito 7: Lanzamiento Fase 6 (Semana 54)
- [ ] Dashboards completos
- [ ] Forecasting con ML funcionando
- [ ] Lanzamiento oficial
- [ ] Plan de soporte post-lanzamiento

---

## 📊 MÉTRICAS DE ÉXITO

### Objetivos de Negocio
- [ ] 95%+ de facturas aceptadas por SUNAT en primer envío
- [ ] <3 días para cierre contable mensual
- [ ] 99.5%+ precisión de inventario
- [ ] <2 segundos de tiempo de respuesta promedio

### Objetivos Técnicos
- [ ] Cobertura de tests >80%
- [ ] Zero critical bugs en producción por 30 días
- [ ] 99.5% uptime
- [ ] 500+ usuarios concurrentes soportados

### Objetivos de Producto
- [ ] 20+ empresas usando el sistema en producción
- [ ] NPS >50
- [ ] <10% churn mensual
- [ ] $500K+ ARR (Annual Recurring Revenue)

---

**Última actualización:** [Fecha]  
**Responsable:** [Nombre del PM/Scrum Master]
