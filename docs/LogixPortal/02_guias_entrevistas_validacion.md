# Guías de Entrevistas para Validación de LogixPortal

---

## OBJETIVOS DE LAS ENTREVISTAS

Las entrevistas tienen múltiples propósitos:
1. **Validar** que entendemos correctamente los procesos de comercio exterior
2. **Descubrir** pain points y necesidades no documentadas
3. **Priorizar** features según valor real para el usuario
4. **Iterar** diseño y funcionalidades basado en feedback
5. **Generar** buy-in y early adopters

---

## TIPOS DE ENTREVISTAS

### 1. Entrevistas de Descubrimiento (Pre-desarrollo)
**Objetivo**: Entender el estado actual y pain points

### 2. Entrevistas de Validación de Diseño (Durante Fase 0)
**Objetivo**: Validar wireframes, flujos y UX

### 3. Entrevistas de Testing de Usabilidad (Post cada fase)
**Objetivo**: Observar uso real y detectar fricciones

### 4. Entrevistas de Feedback Continuo (Durante uso piloto)
**Objetivo**: Mejora continua y priorización de backlog

---

## PERFIL DE ENTREVISTADOS

### Roles a Entrevistar (por empresa)

#### Nivel Estratégico
- **Gerente General / Owner** (1 entrevista)
- **Gerente de Comercio Exterior** (1 entrevista)

#### Nivel Operativo (Los más importantes para validación)
- **Jefe de Importaciones** (2-3 entrevistas)
- **Jefe de Exportaciones** (2-3 entrevistas)
- **Analista de Comercio Exterior** (3-4 entrevistas) ⭐ Usuario principal
- **Asistente de Operaciones** (2 entrevistas)

#### Nivel de Soporte
- **Contador / Jefe de Finanzas** (1 entrevista)
- **Jefe de Logística / Almacén** (1 entrevista)

### Empresas Objetivo

**Perfil Ideal de Empresa Piloto:**
- 20-500 trabajadores
- 10-200 operaciones de importación/exportación al año
- Ya usan algún software (Excel, ERP genérico, o sistema custom)
- Dispuestos a invertir tiempo en validación

**Empresas por Entrevistar (objetivo):**
- 3 empresas importadoras
- 2 empresas exportadoras
- 1 empresa mixta (import/export)

---

## GUÍA 1: ENTREVISTA DE DESCUBRIMIENTO INICIAL
### Duración: 60-90 minutos

### Preparación Previa
- [ ] Enviar NDA si es necesario
- [ ] Confirmar fecha/hora y asistentes
- [ ] Preparar grabadora/notas (pedir permiso)
- [ ] Revisar website de la empresa

### SECCIÓN A: Contexto de la Empresa (10 min)

**Objetivo**: Entender el negocio y volumen de operaciones

```
1. ¿Cuál es su core business? ¿Qué productos importan/exportan?

2. Volumen aproximado:
   - ¿Cuántas importaciones hacen al mes/año?
   - ¿Cuántas exportaciones hacen al mes/año?
   - ¿Valor promedio por operación?

3. ¿Con qué países/regiones trabajan principalmente?

4. ¿Qué INCOTERMS usan más frecuentemente?
   - Importaciones: ___________
   - Exportaciones: ___________

5. Tipo de transporte predominante:
   - [ ] Marítimo FCL
   - [ ] Marítimo LCL
   - [ ] Aéreo
   - [ ] Terrestre
   - [ ] Multimodal
```

### SECCIÓN B: Proceso Actual de Importación (25 min)

**Objetivo**: Mapear su proceso end-to-end

```
6. Descríbame paso a paso cómo gestionan una importación típica, 
   desde que identifican la necesidad hasta que la mercancía 
   está en su almacén.

   [DEJAR HABLAR - TOMAR NOTAS DETALLADAS]
   
   Preguntas de seguimiento:
   - ¿Quién inicia el proceso?
   - ¿Cómo seleccionan al proveedor?
   - ¿Cómo gestionan las cotizaciones?
   - ¿Cómo aprueban las compras?
   - ¿Cómo coordinan con el agente de aduana?
   - ¿Cómo calculan el costo final de importación?

7. ¿Qué herramientas usan actualmente?
   - [ ] Excel / Google Sheets
   - [ ] Software contable (¿cuál?)
   - [ ] ERP genérico (¿cuál?)
   - [ ] Sistema custom
   - [ ] Email
   - [ ] WhatsApp
   - [ ] Otro: ___________

8. ¿Qué documentos generan/gestionan en cada importación?
   [Hacer lista]
   
   Ejemplos a indagar:
   - Solicitud de cotización
   - Cotización de proveedor
   - Orden de compra (PO)
   - Proforma invoice
   - Commercial invoice
   - Packing list
   - Bill of Lading (BL)
   - DUA
   - Certificados de origen
   - Etc.

9. ¿Cómo hacen el seguimiento (tracking) del contenedor?
   - ¿Quién les provee la información?
   - ¿Con qué frecuencia actualizan el estado?

10. ¿Cómo calculan el costo final de la mercancía importada?
    - ¿Tienen una plantilla/fórmula?
    - ¿Incluyen todos los gastos (flete, seguro, aranceles, 
      agente, transporte local, almacenaje)?
```

### SECCIÓN C: Pain Points (15 min)

**Objetivo**: Identificar los mayores dolores

```
11. ¿Cuáles son los 3 mayores problemas/frustraciones que 
    enfrentan en su proceso de importaciones?
    
    1. _________________________________________________
    2. _________________________________________________
    3. _________________________________________________

12. ¿Qué información es difícil de obtener o toma mucho tiempo?

13. ¿Han tenido problemas con costos inesperados en importaciones?
    - ¿Qué tipo de costos?
    - ¿Cómo los manejan?

14. ¿Qué tan fácil/difícil es tener visibilidad del estado actual 
    de todas sus importaciones en curso?

15. ¿Han tenido multas o problemas con SUNAT/ADUANAS?
    - ¿De qué tipo?
    - ¿Cómo podrían haberse evitado?

16. ¿Qué reportes o métricas les gustaría tener que hoy no tienen?
```

### SECCIÓN D: Herramientas Actuales (10 min)

**Objetivo**: Entender qué funciona y qué no de sus herramientas

```
17. De las herramientas que usan hoy, ¿qué es lo que MÁS les gusta?

18. De las herramientas que usan hoy, ¿qué es lo que MÁS odian?

19. Si pudieran cambiar UNA cosa de su software/proceso actual, 
    ¿qué sería?

20. ¿Qué features/funcionalidades son absolutamente CRÍTICAS 
    (no pueden vivir sin ellas)?

21. ¿Qué features les gustaría tener pero no tienen hoy?
```

### SECCIÓN E: Exportaciones (si aplica) (15 min)

**Si la empresa también exporta, repetir preguntas similares:**

```
22. Proceso de exportación paso a paso

23. Documentos que generan/gestionan

24. Pain points específicos de exportación

25. ¿Cómo gestionan las cartas de crédito / cobranzas internacionales?

26. ¿Cómo calculan pricing de exportación (considerando 
    INCOTERMS, comisiones, etc.)?
```

### SECCIÓN F: Cierre (5 min)

```
27. ¿Estarían interesados en probar un nuevo sistema si resuelve 
    estos problemas?
    
    [ ] Definitivamente sí
    [ ] Probablemente sí
    [ ] No estoy seguro
    [ ] Probablemente no
    [ ] Definitivamente no

28. ¿Estarían dispuestos a participar como empresa piloto?
    (Esto implicaría feedback continuo, reuniones cada 2 semanas)

29. ¿Hay alguien más en su equipo con quien deberíamos hablar?

30. ¿Podemos volver a contactarlos para mostrarles prototipos?

31. ¿Alguna otra cosa que quiera compartir?
```

### Post-Entrevista
- [ ] Enviar email de agradecimiento
- [ ] Procesar notas y extraer insights
- [ ] Identificar patterns entre entrevistas
- [ ] Actualizar backlog de features basado en feedback

---

## GUÍA 2: VALIDACIÓN DE WIREFRAMES/DISEÑO
### Duración: 45-60 minutos

**Momento**: Durante Fase 0, antes de programar funcionalidades

### Preparación
- [ ] Tener wireframes en Figma listos para compartir pantalla
- [ ] Preparar flujo específico a validar
- [ ] Tener cuenta de "demo" lista si es prototipo interactivo

### ESTRUCTURA

#### Introducción (5 min)
```
"Hoy queremos mostrarle los diseños iniciales del sistema que 
estamos construyendo. No es funcional todavía, solo diseños.

Queremos su feedback honesto: si algo no tiene sentido, está 
confuso, o falta algo, díganoslo.

No hay respuestas correctas o incorrectas. Ustedes son los expertos."
```

#### Flujo 1: Crear Cotización de Importación (15 min)

**Mostrar pantalla por pantalla**

```
[MOSTRAR PANTALLA 1: Dashboard]

1. ¿Qué le parece esta pantalla inicial? ¿Entiende qué muestra?

2. ¿La información que muestra es útil para usted?

3. Si tuviera que crear una nueva cotización, ¿dónde haría click?

[MOSTRAR PANTALLA 2: Formulario de Cotización]

4. ¿Los campos que pedimos tienen sentido?

5. ¿Falta algo importante?

6. ¿Hay algo innecesario o confuso?

7. ¿El flujo es lógico o esperaría algo diferente?

[Continuar con cada pantalla del flujo]
```

#### Flujo 2: Ver Estado de Importaciones en Curso (10 min)

```
[MOSTRAR Lista/Dashboard de Importaciones]

8. Si quisiera ver todas sus importaciones en curso, ¿esperaría 
   verlas aquí?

9. ¿Qué información esperaría ver en esta lista?
   - ¿Está todo lo que necesita?
   - ¿Hay algo que sobre?

10. ¿Cómo esperaría filtrar o buscar una importación específica?

11. Si hace click en una importación, ¿qué esperaría ver?
```

#### Validación de Terminología (10 min)

```
12. Los nombres que usamos (ej: "Cotización", "Orden de Compra", 
    "Embarque"), ¿son los que usan en su empresa?
    
    Si no, ¿qué términos usan ustedes?

13. [Mostrar íconos] ¿Los íconos son claros o confusos?

14. ¿Los colores/diseño visual les parece profesional?
```

#### Preguntas Abiertas (10 min)

```
15. De todo lo que vio, ¿qué es lo que MÁS le gustó?

16. ¿Qué es lo que MÁS le preocupó o confundió?

17. ¿Usaría este sistema si estuviera disponible mañana?
    ¿Por qué sí o por qué no?

18. En una escala de 1-10, ¿qué tan probable es que use este sistema?
    ¿Por qué ese número?

19. ¿Qué tendríamos que cambiar para que sea un 10?
```

---

## GUÍA 3: TESTING DE USABILIDAD (Sistema Funcional)
### Duración: 60-90 minutos

**Momento**: Al final de Fase 1 (MVP), Fase 2, etc.

### Preparación
- [ ] Sistema funcional en ambiente de STAGING
- [ ] Datos de prueba pre-cargados
- [ ] Usuario y contraseña listos
- [ ] Grabar pantalla (con permiso)
- [ ] Observar, NO guiar

### FORMATO: "Think Aloud"

**Instrucción al usuario:**
```
"Le voy a pedir que haga algunas tareas en el sistema.
Por favor, piense en voz alta mientras lo hace:
- Diga lo que está buscando
- Diga si algo lo confunde
- Diga qué espera que pase cuando hace click

Recuerde: estamos probando el SISTEMA, no a usted.
Si algo no funciona o es confuso, es culpa nuestra, no suya."
```

### TAREAS A REALIZAR

#### Tarea 1: Crear una Nueva Cotización de Importación
```
Escenario:
"Imagine que necesita importar 1000 unidades de Product X desde 
China. Cree una solicitud de cotización para 2 proveedores."

Observar:
- ¿Encuentra dónde empezar?
- ¿Completa los campos sin ayuda?
- ¿El flujo es intuitivo?
- ¿Qué lo frustra?
- ¿Qué le sorprende positivamente?

Preguntas post-tarea:
1. En una escala de 1-5 (muy difícil a muy fácil), ¿qué tan 
   fácil fue completar esta tarea?
   
2. ¿Algo lo confundió?

3. ¿Cambiaría algo?
```

#### Tarea 2: Comparar Cotizaciones y Aprobar una
```
Escenario:
"Han llegado las respuestas de los 2 proveedores. Compare las 
cotizaciones y apruebe la mejor."

Observar:
- ¿Encuentra las cotizaciones recibidas?
- ¿Puede compararlas fácilmente?
- ¿Entiende cómo aprobar?

Preguntas post-tarea:
[Mismas que Tarea 1]
```

#### Tarea 3: Ver el Estado de Todas sus Importaciones
```
Escenario:
"Su jefe le pregunta cuántas importaciones tienen en curso y 
en qué etapa están. Encuentre esa información."

Observar:
- ¿Sabe dónde buscar?
- ¿La información mostrada es clara?
- ¿Puede responder la pregunta del jefe?
```

#### Tarea 4: Calcular Costo Final de una Importación
```
Escenario:
"Necesita saber cuánto le costó finalmente el producto importado 
(incluyendo todos los gastos). Encuentre esa información para 
la importación #12345."

Observar:
- ¿Encuentra el reporte/cálculo?
- ¿Confía en los números?
- ¿Falta algún costo?
```

### PREGUNTAS FINALES

```
1. En general, ¿qué tan fácil o difícil fue usar el sistema?

2. ¿Qué le gustó más?

3. ¿Qué le gustó menos?

4. ¿Hay algo que esperaba encontrar pero no estaba?

5. ¿Hay algo que estaba pero no necesita?

6. Si tuviera una varita mágica, ¿qué cambiaría?

7. ¿Usaría este sistema en su día a día? ¿Por qué sí/no?

8. ¿Qué tendría que mejorar para que lo use?

9. En comparación con su sistema actual, ¿este es mejor, peor, 
   o similar? ¿En qué aspectos?

10. ¿Pagaría por este sistema? ¿Cuánto estaría dispuesto a pagar 
    mensualmente?
```

---

## GUÍA 4: ENTREVISTA DE FEEDBACK CONTINUO (Usuarios Piloto)
### Duración: 30 minutos, cada 2 semanas

**Objetivo**: Mejora continua durante uso real

### ESTRUCTURA

#### Check-in Rápido (5 min)
```
1. ¿Cómo les ha ido con el sistema en las últimas 2 semanas?

2. ¿Lo han usado? ¿Con qué frecuencia?

3. ¿Han encontrado bugs o problemas técnicos?
```

#### Feature Específico (10 min)
```
[Enfocarse en la última feature lanzada]

4. ¿Han probado [Feature X]?

5. ¿Qué opinan de [Feature X]?

6. ¿Les resuelve el problema para el que fue diseñada?

7. ¿Cambiarían algo?
```

#### Priorización de Backlog (10 min)
```
[Mostrar lista de próximas features]

8. De estas 5 features que pensamos desarrollar próximamente, 
   ¿cuáles son las 3 más importantes para ustedes?
   
   Ordénenlas de mayor a menor prioridad:
   
   [ ] Feature A: _______
   [ ] Feature B: _______
   [ ] Feature C: _______
   [ ] Feature D: _______
   [ ] Feature E: _______

9. ¿Por qué eligieron esas 3?

10. ¿Hay algo que NO está en la lista pero que necesitan urgentemente?
```

#### Cierre (5 min)
```
11. ¿Algo más que quieran compartir?

12. ¿Vamos bien? ¿Alguna preocupación?
```

---

## PROCESAMIENTO DE ENTREVISTAS

### Después de Cada Entrevista

1. **Transcribir notas** (mismo día)
2. **Extraer quotes relevantes**
3. **Identificar pain points** específicos
4. **Identificar feature requests**
5. **Clasificar por prioridad**:
   - 🔴 Critical (blocker)
   - 🟡 High (needed soon)
   - 🟢 Medium (nice to have)
   - ⚪ Low (future)

### Después de 3-5 Entrevistas

1. **Buscar patterns**:
   - ¿Qué pain points se repiten?
   - ¿Qué features se mencionan múltiples veces?
   
2. **Crear affinity diagram** (agrupar insights similares)

3. **Actualizar roadmap** basado en findings

4. **Crear user stories** para backlog

5. **Presentar findings** al equipo de desarrollo

---

## TEMPLATES DE DOCUMENTACIÓN

### Template: Resumen de Entrevista

```markdown
# Entrevista con [Empresa] - [Rol del Entrevistado]

**Fecha**: YYYY-MM-DD
**Duración**: XX minutos
**Entrevistador**: [Nombre]
**Tipo**: Descubrimiento / Validación / Usabilidad / Feedback

## Perfil de la Empresa
- Industria: _______
- Tamaño: _______
- Volumen operaciones: _______
- Software actual: _______

## Key Insights
1. [Insight 1]
2. [Insight 2]
3. [Insight 3]

## Pain Points Identificados
| Pain Point | Severity | Quote |
|------------|----------|-------|
| [Problema 1] | 🔴 Critical | "..." |
| [Problema 2] | 🟡 High | "..." |

## Feature Requests
| Feature | Priority | Notes |
|---------|----------|-------|
| [Feature 1] | High | ... |
| [Feature 2] | Medium | ... |

## Quotes Destacados
> "[Quote relevante]" - [Entrevistado]

## Acciones a Tomar
- [ ] Acción 1
- [ ] Acción 2

## Próximos Pasos
- Fecha siguiente entrevista: _______
```

---

## BUENAS PRÁCTICAS

### DO ✅
- Grabar (con permiso) para no perderte nada
- Hacer preguntas abiertas ("¿Cómo...?", "¿Por qué...?")
- Escuchar MÁS de lo que hablas (80/20 rule)
- Indagar en pain points específicos
- Pedir ejemplos concretos
- Observar lenguaje corporal (en video calls)
- Tomar notas de quotes textuales
- Agradecer su tiempo

### DON'T ❌
- Defender tu diseño o discutir
- Llenar silencios rápidamente (dejar pensar)
- Hacer preguntas leading ("¿No crees que esto es mejor?")
- Asumir que entiendes (pedir aclaración)
- Prometer features que no vas a construir
- Interrumpir
- Tomar feedback negativo personal
- Olvidar hacer follow-up

---

**Estas guías son documentos vivos. Ajusta según lo que aprendas en cada entrevista.**
