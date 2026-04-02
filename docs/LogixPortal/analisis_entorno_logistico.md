# Análisis Integral del Ecosistema Logístico y de Comercio Exterior

Este documento estandariza y detalla el entorno de negocio que abarca desde la cadena de suministro internacional hasta la entrega al cliente final. El objetivo es proporcionar una base sólida para el diseño y arquitectura de un sistema ERP/Logístico web (B2B/B2C) desarrollado en **Serverpod** como backend y **Dart/Flutter** como cliente web/móvil.

---

## 1. El Flujo de la Cadena de Suministro (Visión End-to-End)

En una operación internacional *Door to Door* (Ej. un incoterm EXW o DDP), interactúan típicamente los siguientes ejes:

1. **Comercio Exterior:** El importador negocia la compra en el extranjero y solicita el servicio.
2. **Forwarder (Origen y Destino):** Coordina el recojo de la carga en fábrica del proveedor, y la monta en un barco/avión.
3. **Agente de Aduana:** Al llegar al país, se encarga del cumplimiento regulatorio (pago de tributos, inspecciones).
4. **Transporte Primario:** Mueve el/los contenedor(es) del puerto extra-portuario hacia los propios almacenes.
5. **WMS:** Recepciona la carga (desconsolidación), la codifica, y la pone en *stock* físico y virtual.
6. **Última Milla:** Con base a pedidos *e-commerce* o empresariales B2B, recoge del almacén y despacha con flota pesada y ligera mediante ruteos optimizados y app de conductor.

---

## 2. Dominios de Negocio a Sistematizar

### 2.1 Comercio Exterior (El Cliente Final)
Es la génesis de la necesidad logística. Este módulo permite al Importador / Exportador administrar su compraventa.
*   **Procesos Clave:**
    *   Emisión de Órdenes de Compra (Purchase Orders).
    *   Cotización Internacional de fletes y simulación de Costeo (Landed Cost).
*   **Datos y Entidades Clave:** INCOTERMs (FOB, CIF, EXW), Partidas Arancelarias (HS Codes), Commercial Invoice, Packing List y Certificados de Origen.

### 2.2 Agente de Carga Internacional (Freight Forwarder)
Es el "Arquitecto del Transporte". Coordina con líneas navieras y aerolíneas; no son dueños de los activos, compran espacios.
*   **Procesos Clave:**
    *   Gestión de Tarifas (Pricing) y Recargos.
    *   Agrupación y Desagrupación (Consolidación LCL / Desconsolidación).
    *   Emisión de Documentos de Transporte (Bill of Lading - B/L, Airway Bill - AWB).
*   **Datos y Entidades Clave:** Rutas (Port of Loading - POL, Port of Discharge - POD), Fechas (ETA, ETD, Cut-off), Conceptos de Facturación de Flete.

### 2.3 Agente de Aduanas (Customs Broker)
Es el ente fiscal y aduanero que vela por la nacionalización de la carga. Intermediario regulado por el estado (Ej. SUNAT Aduanas).
*   **Procesos Clave:**
    *   Confección y numeración de la DAM (Declaración Aduanera de Mercancías).
    *   Despacho Aduanero, Reconocimientos Previos o Aforos Físicos/Documentales.
    *   Gestión de Levante Aduanero.
*   **Datos y Entidades Clave:** Regímenes aduaneros, Canales de control (Verde, Naranja, Rojo), Impuestos (IGV, ISC, Ad Valorem, Percepciones), Multas, Garantías. 

### 2.4 Operador Logístico Integral (3PL / 4PL)
Integra todo lo anterior prestando a su cliente una visión unificada, usualmente asumiendo responsabilidad de extremo a extremo.
*   **Procesos Clave:**
    *   Generación del **Expediente o "Carpeta"**: Agrupa el Forwarding, Aduanas, y Transporte.
    *   Portal B2B: Dar visibilidad al cliente final en un solo dashboard.
    *   Facturación Unificada integrando todas las unidades de negocio.

### 2.5 Manejo de Almacenes (WMS - Warehouse Management System)
Administra los recintos físicos de almacenaje, maximizando espacios y minimizando errores.
*   **Procesos Clave:**
    *   **Inbound:** Cita de recepción, descarga, inspección de calidad y *Putaway* (acomodo en estantería).
    *   **Almacenamiento y Control:** Mantenimiento (Conteos cíclicos), Reposición (Replenishment).
    *   **Outbound:** Creación de olas de pedido, Picking (selección manual/PDA), Packing (etiquetado), y Staging para despacho.
*   **Datos y Entidades Clave:** Maestro de SKUs, Lotes y Caducidades, Ubicaciones lógicas (Centro > Almacén > Zona > Pasillo > Rack > Posición), Regímenes de salida (FIFO, FEFO, LIFO), Trazabilidad de Movimientos.

### 2.6 Delivery de Última Milla (Last Mile / TMS Ligero)
Distribución capilar al cliente B2C o Puntos de Venta B2B. Aporta un contacto directo e intenso seguimiento.
*   **Procesos Clave:**
    *   Gestión de Flota e integración de órdenes.
    *   Planificación de Rutas (dinámicas y maestras).
    *   Liquidación de Viaje y Logística Inversa (Devoluciones).
*   **Datos y Entidades Clave:** Manifiestos/Hoja de ruta, Geocercas, PRUEBA DE ENTREGA (POD - *Proof of Delivery*: Firma digitalizada, Fotografías, Geolocalización, sellos), Control de Rechazos.

---

## 3. Arquitectura y Modelado en Serverpod

Implementar esta vasta solución web en **Serverpod (Dart Backend)** aporta grandes beneficios, por su fuerte tipado de extremo a extremo y soporte robusto a la comunicación en tiempo real. 

### 3.1 Agrupación de Módulos (Archivos .yaml base de Serverpod)

El sistema debe tener un diseño modular a nivel de capa de datos. Ejemplos de modelos a definir:

**A. Core, Entidades e Infraestructura (`/models/core/`)**
*   `company.yaml`: Clientes, Agentes, Proveedores, Navieras, Aerolíneas.
*   `person_contact.yaml`: Contactos B2B o Consignatarios.
*   `location_master.yaml`: Puertos (códigos UNLOCODE), Aeropuertos, Códigos Postales, Direcciones Fiscales.

**B. Operaciones (Forwarding, Aduana & 3PL) (`/models/operations/`)**
*   `operation_file.yaml`: La entidad central *(El Expediente)* de la que cuelga el resto.
*   `routing.yaml`: Segmentos del transporte internacional, ETA/ETD.
*   `bl_document.yaml`: Datos del B/L (peso bruto, volumen, contenedores).
*   `customs_dam.yaml`: Datos puntuales regulatorios (Números de orden, canales, levantes).

**C. WMS (Módulo de Almacenes) (`/models/wms/`)**
*   `product_sku.yaml`: Ficha técnica del producto (Dimensiones, código de barras corporativo).
*   `warehouse_location.yaml`: Mapeo jerárquico del almacén con tipo de rack.
*   `inventory.yaml`: La foto actual del stock.
*   `stock_movement_ledger.yaml`: Importante para bases de datos transaccionales, cada alta, baja y transferencia de stock, dejada como un registro inmutable.

**D. Última Milla (Last Mile & Flota) (`/models/last_mile/`)**
*   `vehicle.yaml` / `driver.yaml`: Gestión del activo móvil y la persona a cargo.
*   `route_manifest.yaml`: Hoja de ruta generada.
*   `delivery_stop.yaml`: Puntos secuenciales de la hoja de ruta.
*   `proof_of_delivery.yaml`: Tabla para almacenar metadatos de las fotos de entrega y estatus finales (Entregado, Parcial, Fallido).

### 3.2 Recomendaciones Funcionales al Aprovechar Serverpod

1.  **Eventos en Tiempo Real con Streams:** 
    Serverpod permite configurar `Streaming`. Esto es CRÍTICO para el módulo de Última Milla. Cuando los conductores usen el App de reparto, enviarán su posición (Lat/Lng) vía WebSocket de Serverpod a la base de datos, y el administrador logístico verá la ubicación de la flota moviéndose en un mapa web en vivo.
2.  **Manejo de Pruebas y Evidencias (Files / S3):** 
    Desde la DAM de aduanas enviada en PDF, hasta la Foto de Entrega tomada por el chófer. En Serverpod se debe integrar fuertemente `CloudStorageManager`. Sugerimos conectar un Bucket de Amazon S3 con Serverpod para administrar el altísimo volumen de documentos y facturas que produce la logística.
3.  **End-Points Personalizados y Webhooks:** 
    El comercio exterior necesita comunicarse con terceros (Sistemas de GPS como Wisetrack, portales de Aduanas VUCE, sistemas contables como Odoo o SAP). Serverpod te permite no solo crear APIs REST en caso de requerirse para la ingesta externa de información, sino conectar tareas cronometradas (`FutureCall`) para consultar la tasa de cambio de la SBS al día automáticamente.
4.  **Caché Avanzado (Redis):** Si tu aplicación web mostrará el stock de cientos de miles de SKUs de un almacén en tiempo real, usar Redis provisto por Serverpod permite que las consultas principales del cliente (B2B Dashboard) sean ultra-rápidas y se aligere la carga en PostgreSQL.

---

## 4. Definición del Producto Fase 1 (MVP)

La enorme amplitud del ecosistema requiere estructurar las entregas:
*   **Fase 1 (Core + Expediente Visibilidad):** Alta de clientes, maestro de proveedores, y creación del "File" (Expediente) donde se registre que un contenedor partió, en qué fecha llega y si ya ingresó al puerto.
*   **Fase 2 (WMS Local):** Ingreso de mercancía al almacén (lectura por lote/serie), control de ubicación centralizado en terminal y egreso de carga.
*   **Fase 3 (Última milla & Mobile):** Trazabilidad de reparto y carga de evidencias (Delivery App para choferes).
*   **Fase 4 (Aduana & Cobranza Integrada):** Algoritmos de provisión de gastos de despacho aduanero y cierres financieros de expedientes.
