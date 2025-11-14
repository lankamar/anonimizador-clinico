# Marco Normativo y Regulatorio

## Fundamentación Legal Argentina

### Ley 25.326 - Protección de los Datos Personales

**Artículos clave:**

- **Art. 2**: Define "datos personales" como información de cualquier tipo referida a personas físicas o de existencia ideal determinadas o determinables.
- **Art. 7**: Establece que los **datos disociados** (aquellos que no pueden vincularse a una persona determinada o determinable) **quedan fuera del ámbito de aplicación** de la ley.
- **Art. 9**: Los datos sensibles (salud, origen étnico, genética) requieren consentimiento expreso y nivel máximo de protección.

**Implicancia para este proyecto:**  
El objetivo central es lograr la **disociación irreversible** de los datos clínicos, transformando datos personales en datos disociados mediante técnicas de supresión, generalización, agrupamiento y hash irreversible.

---

### Disposición 11/2006 - Dirección Nacional de Protección de Datos Personales

**Objeto:** Establece criterios técnicos para la disociación de datos personales.

**Requisitos técnicos:**
- El proceso de disociación debe ser **irreversible**
- No debe existir posibilidad razonable de reconstruir la identidad del titular
- Deben eliminarse identificadores directos e indirectos que permitan la reidentificación
- Se recomienda revisión por expertos técnicos y éticos

**Implicancia para este proyecto:**  
La aplicación implementa múltiples capas de anonimización (supresión + generalización + agrupamiento + hash) para asegurar irreversibilidad conforme a estos criterios.

---

### Resolución 1480/2011 - Ministerio de Salud

**Objeto:** Guía Nacional para Investigaciones con Seres Humanos.

**Requisitos:**
- Toda investigación con datos de salud requiere aprobación de **Comités de Ética en Investigación** (CEI)
- Los datos deben protegerse mediante medidas técnicas y organizativas
- Se debe garantizar confidencialidad, incluso en datos anonimizados
- Trazabilidad y auditoría del proceso

**Implicancia para este proyecto:**  
El sistema genera documentación exhaustiva (logs, reportes de auditoría) que facilita la revisión por CEI y demuestra cumplimiento de salvaguardas técnicas.

---

## Metodología Internacional: Sondeck-Laurent (2025)

**Referencia:** Sondeck, LP & Laurent, M. (2025). "Practical and Ready-to-Use Methodology to Assess the Re-Identification Risk in Anonymized Datasets." *Scientific Reports*, 15(1):23223.

### Principios Clave

1. **Evaluación granular de riesgo:** Analiza atributo por atributo, no solo globalmente
2. **Clasificación de atributos:**
   - **Identificadores Directos (IDD):** Nombre, DNI, dirección → Supresión obligatoria
   - **Cuasi-Identificadores (QI):** Edad, fecha, código postal → Evaluación caso por caso
   - **Atributos Sensibles (AS):** Diagnóstico, tratamiento → Protección reforzada

3. **Matriz de Riesgo:** Cada QI se califica por:
   - **Severidad (S):** Impacto del daño si ocurre reidentificación (Alta/Media/Baja)
   - **Exposición (E):** Probabilidad de vinculación con fuentes externas (Alta/Media/Baja)

4. **Escenarios de Ataque:**
   - **Prosecutor's attack:** Atacante con conocimiento previo del individuo
   - **Journalist's attack:** Búsqueda de individuos específicos notables
   - **Marketer's attack:** Identificación de grupos demográficos

5. **Clases de Equivalencia:** Agrupamiento para k-anonimato (mínimo k individuos comparten combinación de atributos)

### Adaptación al Contexto Argentino

| Aspecto | Metodología Sondeck-Laurent | Adaptación Argentina |
|---------|----------------------------|---------------------|
| **Modelos de privacidad** | k-anonimato, l-diversidad, t-closeness | Incorporado en motor de reglas configurable |
| **Evaluación de riesgo** | Métricas cuantitativas | Estimación cualitativa (Bajo/Medio/Alto) con justificación |
| **Documentación** | Criterios y resultados formales | Tabla exhaustiva de intervenciones + log SQLite |
| **Auditoría** | Revisión periódica recomendada | Integración con revisión ética institucional (CEI) |

---

## Modelos de Privacidad Implementados

### k-Anonimato
**Definición:** Cada combinación única de cuasi-identificadores debe aparecer en al menos k registros.

**Implementación:**
- Agrupamiento de edad en rangos (5 años)
- Generalización geográfica (provincia vs. ciudad)
- Generalización temporal (año vs. fecha exacta)

**Ejemplo:**
- (Edad: 37, Localidad: La Plata, Fecha: 15/03/2021) 
- → (Edad: 35-39, Provincia: Buenos Aires, Año: 2021)

### l-Diversidad
**Definición:** Para cada clase de equivalencia, debe haber al menos l valores distintos del atributo sensible.

**Aplicación:**
- Protección de diagnósticos raros mediante agrupamiento en categorías CIE-10 de nivel superior
- Verificación de diversidad en tratamientos dentro de cada grupo demográfico

### t-Closeness
**Definición:** La distribución del atributo sensible en cada clase de equivalencia debe ser similar a la distribución global.

**Aplicación:**
- Monitoreo de distribuciones de diagnósticos/tratamientos post-anonimización
- Alertas cuando agrupamientos generan sesgos evidentes

---

## Brechas Regulatorias y Soluciones Técnicas

### Brecha 1: Ausencia de Herramientas Formales de Evaluación de Riesgo

**Problema:** La normativa argentina define "disociación" pero no especifica metodologías cuantitativas para medir riesgo residual.

**Solución en este proyecto:**
- Adaptación de Sondeck-Laurent para cuantificar riesgo
- Clasificación automática de atributos (IDD/QI/AS)
- Matriz de Severidad × Exposición por variable
- Estimación de riesgo residual documentada

### Brecha 2: Fragmentación Provincial

**Problema:** Diferentes jurisdicciones pueden tener requisitos específicos.

**Solución:**
- Motor de reglas configurable (YAML/JSON)
- Perfiles por provincia/institución
- Versionado de reglas para trazabilidad

### Brecha 3: Evaluación de Utilidad Post-Anonimización

**Problema:** La ley no especifica cómo confirmar que los datos conservan "utilidad científica".

**Solución:**
- Sección obligatoria "Riqueza Diagnóstica" en reporte de auditoría
- Validación caso por caso de preservación de información clínica relevante
- Métricas de completitud pre/post anonimización

---

## Principio de Minimización de Datos

**Origen:** GDPR (Art. 5.1.c) y mejores prácticas internacionales

**Aplicación:**
- Solo se anonimizan y conservan datos estrictamente necesarios para el propósito científico declarado
- Opción de "anonimización selectiva" por campos
- Documentación del propósito justifica campos incluidos

---

## Transferencia Internacional de Datos

**Restricción Ley 25.326 (Art. 12):**  
Prohibida transferencia a países sin nivel adecuado de protección, salvo tratados específicos.

**Implicancia:**
- Datos deben ser anonimizados **ANTES** de cualquier transferencia internacional
- Modo "local" por defecto, sin salida automática
- Modo "híbrido" requiere documentación y aprobación explícita

---

## Responsabilidades y Roles

### Investigador Responsable
- Define propósito y campos necesarios
- Solicita aprobación al CEI
- Configura reglas de anonimización
- Valida riqueza diagnóstica post-proceso

### Comité de Ética en Investigación (CEI)
- Revisa PRD y configuración de reglas
- Evalúa proporcionalidad riesgo/beneficio
- Aprueba o rechaza uso de datos anonimizados
- Solicita auditorías periódicas

### Responsable de Protección de Datos (DPO)
- Supervisa cumplimiento Ley 25.326
- Audita logs y trazabilidad
- Verifica irreversibilidad de disociación
- Coordina con Dirección Nacional de Protección de Datos Personales

### Auditor Técnico
- Valida implementación de algoritmos
- Simula ataques de reidentificación
- Certifica nivel de riesgo residual
- Recomienda mejoras técnicas

---

## Ciclo de Auditoría Recomendado

1. **Auditoría inicial** (antes del primer uso)
   - Revisión de código y configuración
   - Pruebas con datos sintéticos
   - Simulación de ataques (Sondeck-Laurent)

2. **Auditoría por lote** (cada N casos procesados)
   - Revisión de logs
   - Verificación de consistencia de reglas
   - Análisis estadístico de distribuciones

3. **Auditoría periódica** (anual o bianual)
   - Reevaluación de riesgo residual
   - Actualización de reglas según nuevas amenazas
   - Certificación de cumplimiento normativo

---

## Referencias Normativas Completas

1. **Ley 25.326** - Protección de los Datos Personales (Argentina, 2000)
2. **Disposición 11/2006** - Criterios técnicos de disociación (DNPDP, 2006)
3. **Resolución 1480/2011** - Guía para investigaciones con seres humanos (Ministerio de Salud, 2011)
4. **Sondeck & Laurent (2025)** - *Scientific Reports* 15(1):23223
5. **GDPR** - Reglamento General de Protección de Datos (UE 2016/679)
6. **HIPAA Privacy Rule** - 45 CFR Part 160 and Subparts A and E of Part 164 (EE.UU.)
7. **Guías CONICET** - Buenas Prácticas de Gobernanza de Datos en Investigación Biomédica (2023)

---

## Glosario

- **Disociación:** Proceso irreversible que impide vincular datos a una persona determinada o determinable
- **Identificador Directo (IDD):** Dato que por sí solo identifica a una persona (nombre, DNI)
- **Cuasi-Identificador (QI):** Dato que combinado con otros puede permitir identificación (edad + código postal)
- **Atributo Sensible (AS):** Información especialmente protegida (salud, genética, origen étnico)
- **k-anonimato:** Propiedad por la cual cada combinación de QIs aparece al menos k veces
- **Riesgo Residual:** Probabilidad de reidentificación después de aplicar anonimización
- **Clase de Equivalencia:** Conjunto de registros que comparten la misma combinación de QIs

---

**Última actualización:** 2025-11-14  
**Versión:** 1.0.0
