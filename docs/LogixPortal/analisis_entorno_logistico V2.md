# Análisis Integral del Ecosistema Logístico y de Comercio Exterior (Versión Ampliada V2)

Este documento detalla el entorno de negocio que abarca desde la cadena de suministro internacional hasta la entrega al cliente final, enfocado al entorno regulatorio y geográfico del **Perú**, y enmarcado en los estándares internacionales de seguridad (**OEA, BASC**). El objetivo es estructurar este conocimiento para el diseño y arquitectura de un sistema ERP/Logístico web (B2B/B2C) desarrollado en **Serverpod** y **Dart/Flutter**.

---

## 1. El Core del Comercio Internacional

### 1.1 INCOTERMS 2020 (Términos de Comercio Internacional)
Definen el punto exacto donde se transfiere el riesgo, el costo y la responsabilidad entre comprador y vendedor. En la arquitectura de `Serverpod`, el Incoterm determinará qué servicios y costos (flete, seguro, aduana) se activan automáticamente dentro del *Expediente Operativo*.
*   **Origen/Salida:** 
    *   **EXW (Ex Works):** El comprador asume casi el 100% de la logística (recoge en fábrica del vendedor).
    *   **FCA (Free Carrier):** Vendedor solo entrega la carga al primer transportista en el país de origen.
*   **Transporte Principal No Pagado (Marítimos):**
    *   **FOB (Free On Board):** Vendedor asume costos hasta poner la carga *sobre el buque* en el puerto de embarque.
*   **Transporte Principal Pagado:**
    *   **CFR / CIF (Cost, Insurance & Freight):** Vendedor paga el flete (y el seguro en CIF) hasta el puerto de destino, pero el riesgo se transmitió al cargar el buque en origen.
*   **Llegada/Destino:**
    *   **DAP (Delivered at Place):** Vendedor asume flete y transporte hasta el lugar de entrega (ej. almacén del comprador), pero *sin* gastos de aduana.
    *   **DDP (Delivered Duty Paid):** Vendedor asume toda la cadena "Door to Door", pagando flete, aduana, y transporte local.

### 1.2 Documentos Requeridos y Digitalización (Gestor Documental)
El ERP debe servir como custodio digital de copias maestras inmutables.
*   **Comerciales:** `Commercial Invoice` (Factura comercial), `Packing List` (Lista de empaque, pesos y dimensiones).
*   **Transporte:** `Bill of Lading` (BL - Marítimo; incluye House BL emitido por Forwarder y Master BL emitido por Naviera), `Airway Bill` (AWB - Aéreo), `Carta Porte` (Terrestre internacional CRTM).
*   **Aduanas (Perú):** La **DAM** (Declaración Aduanera de Mercancías - formato DUA o Simplificada). `Volante de Despacho` (emitido al ingresar la carga al depósito temporal marítimo/aéreo).
*   **Certificaciones y Regulaciones:** `Certificados de Origen` (para acogerse a TLCs), `Póliza de Seguro`.

### 1.3 Modos de Transporte (Routing)
*   **Marítimo:** FCL (Full Container Load - de 20', 40', Reefer), LCL (Less than Container Load - carga suelta consolidada), Breakbulk / RoRo (maquinaria pesada rodante o fraccionada).
*   **Aéreo:** ULD (Pallets o contenedores aéreos especializados), Carga Suelta (*Loose Cargo*).
*   **Terrestre / Última Milla:** Trailers porta-contenedores al salir de aduana, camiones (FTL/LTL) refrigerados (para *cold chain*) o furgones ligeros de ciudad.

---

## 2. Marco Normativo y Aduanero en el Perú

Para operar una plataforma logística legal y eficiente en suelo peruano, se contemplan flujos obligatorios interconectados al Estado.

### 2.1 Regímenes Aduaneros (SUNAT)
Las operaciones de la DAM deben mapearse según estos flujos (Códigos de régimen de SUNAT):
1.  **Importación para el Consumo (Código 10):** Nacionalización definitiva, pago total de tributos (IGV 16%, IPM 2%, Ad/Valorem según partida, ISC en ciertos bienes, Percepción del IGV 3.5% o 10%).
2.  **Exportación Definitiva (Código 40):** Venta al extranjero. Trámite vital para gozar del Drawback o saldo a favor del IGV.
3.  **Admisión Temporal para Reexportación (Código 20):** Entra maquinaria (ej. minería) sin pagar impuestos por un tiempo definido, garantizado por una Carta Fianza. Control estricto de fechas de vencimiento en el sistema para evitar multas millonarias.
4.  **Admisión Temporal para Perfeccionamiento Activo (Código 21):** Ingresan insumos sin pagar tributos, se transforman (Ej. entran cierres y telas para fabricar un pantalón), y el producto final *se exporta obligatoriamente*.
5.  **Depósito Aduanero (Código 70):** Permite almacenar mercancía bajo control aduanero *sin* pagar impuestos aún, y nacionalizarla fraccionadamente según convenga mes a mes.
6.  **Tránsito Aduanero / Transbordo:** Cruce de mercancía por el país hacia otro destino.
7.  **Drawback:** Régimen promocional donde se pide la devolución parcial de aranceles al estado al haber exportado mercancía costeada con partes importadas.

### 2.2 Entidades Reguladoras Locales y Trámites
Tu plataforma web integrará notificaciones, aprobaciones o subirá los permisos emitidos por estas entidades (muchos a través del sistema del estado **VUCE - Ventanilla Única de Comercio Exterior**).
*   **Ministerio de Transporte y Comunicaciones (MTC):** Permisos para equipos de telecomunicaciones (Homologaciones).
*   **SENASA:** Sanidad Agraria (Plantas, frutas, verduras, cueros sin curtir, animales). Emiten certificados fito/zoosanitarios.
*   **DIGESA:** Salud Ambiental. Permisos para importar alimentos industrializados, juguetes (muy común en campañas navideñas) y artículos del hogar.
*   **DIGEMID:** Dirección de Medicamentos. Cosméticos, reactivos bioquímicos y medicinas.
*   **SANIPES:** Órgano regulador sanitario para productos de origen pesquero.

---

## 3. Mejores Prácticas de Mercado Internacional (BASC, OEA)

Para que el diseño contenga **BASC** y **OEA**, el esquema de bases de datos debe garantizar la trazabilidad extrema y resguardos anti-fraude, inmutabilidad de movimientos e integraciones de seguridad en transportes.

### 3.1 Operador Económico Autorizado (OEA)
El OEA (gestionado por SUNAT en Perú pero de carácter la Organización Mundial de Aduanas) premia a empresas con historial intachable de cumplimiento aduanero, fiscal y de seguridad de la cadena de suministro.
*   **Beneficio principal para el negocio:** Garantiza **Canal Verde** (sin revisión física) en los despachos o atenciones prioritarias. 
*   **Requisito Tecnológico que te pide OEA:**
    *   Trazabilidad Documentaria Intachable: Cada DAM y BL debe registrar *quién*, *cuándo* y *cómo* los modificó en la base de datos de Serverpod.
    *   Custodia y permisos segregados (RBAC) basados en roles estrictos en Serverpod (ej. Solo liquidador A ve expedientes B).
    *   Un sólido sistema financiero incorporado que permita cruzar facturas (Commercial Invoices) contra transferencias SWIFT reales, mitigando el lavado de activos.

### 3.2 Business Alliance for Secure Commerce (BASC)
La norma prioriza *Seguridad Preventiva* contra narcotráfico, terrorismo y soborno que utilicen los contenedores del negocio de tu cliente como transporte.
*   **Requisito Tecnológico para que tu cliente renueve BASC anualmente:**
    *   Registro irrefutable del GPS de los camiones integrados al Módulo de Última Milla/Transporte. Desviaciones de ruta (Ruta Maestra vs Ruta Ejecutada) deben mandar una alerta instantánea (Vía WebSocket en Serverpod + Envío de SMS/Mail al cliente).
    *   **Trazabilidad de Precintos (Sellos de Seguridad):** El sistema debe guardar el # de Precinto de origen y exigir una doble validación visual y confirmación de que el precinto en destino fue el mismo.
    *   Upload obligatorio de Fotografías de 7 Puntos (Las 7 revisiones al contenedor vacío: Pisos, techos, paredes) en la App Flutter.

---

## 4. Evolución de la Arquitectura en Serverpod

Al sumar regulación, el diseño propuesto de nuestros modelos `YAML` en Serverpod debe especializarse:

### Módulo Principal Extendído (Master Data)
*   **`incoterm.yaml`:** Tabla catálogo maestral (FOB, CIF, etc.) con banderas sobre cargos (`incluyeFlete`, `incluyeSeguro`).
*   **`sunat_hs_code.yaml`:** La Partida Arancelaria. Esta tabla debe estar pre-poblada con los ad-valorem actuales y tasas de recargo impositivo. 
*   **`hs_code_restriction.yaml`:** Interseccional ("Many to Many"). Indica que la partida regulada "Z" exige pedir un trámite en `DIGEMID`. Bloqueando al operador logístico a numerar DAM si la fecha del permiso ha expirado.

### Manejo de Regímenes y Trámites
*   **`customs_dam_regime.yaml`:** Registro inmutable del cambio de estatus de la mercancía. Ejemplo: *Admisión temporal -> Vence en Día X -> Cambió a régimen de Importación a consumo el Día X-1*. Un *CronJob* (`FutureCall` en Serverpod) todos los días recorrerá la BBDD a medianoche detectando regímenes temporales próximos a vencer y enviando alertas automáticas a los clientes B2B sobre confiscación por SUNAT inminente.
*   **`vuce_procedure.yaml`:** Tabla que asocia un expediente a un nro. de trámite iniciado en el portal VUCE del estado (Fecha de vigencia, Fecha de vencimiento, PDF de la resolución adjunto).

### Módulo de Transporte Seguro (Compliance OEA/BASC)
*   **`security_seal.yaml`: (Manejo de Precintos).** Tabla para llevar la vida de un precinto de botella satelital o físico amarrado a `Vehicle` o `Container`.
*   **`gps_route_deviation.yaml`: (Alertas)** Un registro de todas las veces que un camión se desvió más de *X* kilómetros o permaneció detenido más de *N* minutos en la autopista (sospecha de contaminación/robo de carga).
*   **`audit_log.yaml`:** Vital para OEA. Es el "Historial inmutable" o log de acciones interceptadas por un middleware de seguridad a nivel Serverpod. ¿Cambiaron la Commercial Invoice el 22 de abril? Queda la evidencia forense grabada por quién y qué IP lo hizo.
