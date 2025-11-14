# Arquitectura Técnica - Anonimizador Clínico Local

## Diagrama de Flujo Principal

```
┌────────────────────────────────────────────────────────────────┐
│                    INGESTA DE DATOS                            │
│  • TXT, CSV, JSON (datos estructurados)                        │
│  • DOCX, PDF (documentos)                                      │
│  • DICOM, JPG, PNG (imágenes médicas)                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│                 MÓDULO 1: PARSING                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │
│  │ Parser TXT   │ │ Parser CSV   │ │ Parser JSON  │           │
│  └──────────────┘ └──────────────┘ └──────────────┘           │
│                                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │
│  │ Parser DOCX  │ │ Parser PDF   │ │ Parser DICOM │           │
│  └──────────────┘ └──────────────┘ └──────────────┘           │
│                                                                 │
│  ↓ Normalización a estructura común interna                   │
│  {campo: valor, tipo: IDD/QI/AS, metadata: {...}}             │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│            MÓDULO 2: CARGA DE CONFIGURACIÓN                    │
│  • Archivo: rules_config.yaml                                  │
│  • Perfil institucional (ej: UBA-HCJSM-Enfermería)            │
│  • Versión de reglas y métodos de anonimización               │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│         MÓDULO 3: ANONIMIZACIÓN TEXTUAL (Bifurcación)          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 3A: SUPRESIÓN DE IDD                               │       │
│  │ • Nombre, DNI, Teléfono, Email, Dirección          │       │
│  │ • Reemplazar por [SUPRIMIDO]                       │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 3B: GENERALIZACIÓN DE CUASI-IDENTIFICADORES        │       │
│  │ • Edad → Rangos (35-39 años)                       │       │
│  │ • Fechas → Año (15/03/2021 → 2021)                │       │
│  │ • Localidad → Provincia (La Plata → Buenos Aires)  │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 3C: PROTECCIÓN DE ATRIBUTOS SENSIBLES              │       │
│  │ • Diagnósticos raros: Generalizar a categoría sup. │       │
│  │ • Datos genéticos: Suprimir si no esencial        │       │
│  │ • Origen étnico: Suprimir o generalizar            │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│ MÓDULO 4A:               │    │ MÓDULO 4B:                   │
│ HASH IRREVERSIBLE        │    │ ANONIMIZACIÓN MULTIMEDIA     │
│                          │    │                              │
│ • ID Clínico Input       │    │ ┌────────────────────────┐  │
│ • SHA-256 + Salt         │    │ │ 4B1: DICOM             │  │
│ • Truncar a 8 chars      │    │ │ • pydicom.dcmread()    │  │
│ • Output: HC_{HASH}      │    │ │ • Eliminar tags PII    │  │
│ • Irreversible ✓         │    │ │ • Convertir fechas→año │  │
│ • Sin colisiones ✓       │    │ │ • pydicom.dcmwrite()   │  │
│                          │    │ └────────────────────────┘  │
│                          │    │                              │
│                          │    │ ┌────────────────────────┐  │
│                          │    │ │ 4B2: JPG/PNG/EXIF      │  │
│                          │    │ │ • PIL.Image.open()     │  │
│                          │    │ │ • Eliminar metadata    │  │
│                          │    │ │ • OCR si hay texto     │  │
│                          │    │ │ • PIL.Image.save()     │  │
│                          │    │ └────────────────────────┘  │
│                          │    │                              │
│                          │    │ ┌────────────────────────┐  │
│                          │    │ │ 4B3: PDF/DOCX          │  │
│                          │    │ │ • Eliminar metadatos   │  │
│                          │    │ │ • Buscar PII en texto  │  │
│                          │    │ │ • Resguardar contenido │  │
│                          │    │ └────────────────────────┘  │
└──────────────────────────┘    └──────────────────────────────┘
         │                               │
         └───────────────┬───────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│      MÓDULO 5: EVALUACIÓN DE RIESGO (Sondeck-Laurent)          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 5A: Clasificación de Atributos                      │       │
│  │ • IDD (Identificadores Directos) → Supresión       │       │
│  │ • QI (Cuasi-Identificadores) → Generalización      │       │
│  │ • AS (Atributos Sensibles) → Protección            │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 5B: Matriz de Riesgo (Severidad × Exposición)      │       │
│  │                                                      │       │
│  │     Exposición                                       │       │
│  │     B    M    A                                      │       │
│  │  A [4]  [8]  [12]      Severidad                    │       │
│  │  M [2]  [6]  [10]                                   │       │
│  │  B [1]  [3]  [5]                                    │       │
│  │                                                      │       │
│  │ Cada QI obtiene puntuación; suma = riesgo global   │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 5C: Validación k-Anonimato                          │       │
│  │ • Combinación crítica: Edad, Provincia, Diagnóstico│       │
│  │ • Verificar k ≥ 5 en base institucional             │       │
│  │ • Si k < 5 → ALERTA y marcar para revisión         │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 5D: Estimación de Riesgo Residual                   │       │
│  │ • Riesgo Global = f(Severidad, Exposición, k)      │       │
│  │ • Output: BAJO / MEDIO / ALTO                      │       │
│  │ • Generar justificación documentada                │       │
│  └─────────────────────────────────────────────────────┘       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│      MÓDULO 6: AUDITORIA Y REGISTRO (Trazabilidad)            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 6A: Base de Datos SQLite (logs/auditoria.db)        │       │
│  │ • Tabla: operaciones_anonimizacion                  │       │
│  │ • Campos: timestamp, usuario, archivo, reglas, ...  │       │
│  │ • Cifrado AES-256                                   │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 6B: Generación de Tabla de Intervenciones          │       │
│  │ • Listar cada regla aplicada + justificación        │       │
│  │ • Grado de intervención (BAJO/MODERADO/ALTO)       │       │
│  │ • Confirmar riqueza diagnóstica preservada          │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 6C: Generación de JSON Log                          │       │
│  │ {usuario, timestamp, versión_reglas, reglas_usadas} │       │
│  └─────────────────────────────────────────────────────┘       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│      MÓDULO 7: EXPORTACIÓN (Markdown/PDF)                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 7A: Generación de Markdown (.md)                    │       │
│  │ • Historia clínica anonimizada fiel                 │       │
│  │ • Tabla de anonimización exhaustiva                 │       │
│  │ • Sección de auditoría (usuario, fecha, versión)    │       │
│  │ • Referencias a imágenes/informes procesados         │       │
│  │ • Log de operaciones                                │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 7B: Generación de PDF (.pdf)                        │       │
│  │ • Mismo contenido que MD con formato profesional    │       │
│  │ • Generador: ReportLab o WeasyPrint                 │       │
│  │ • Imágenes embebidas o referencias según config     │       │
│  │ • Estilos y headers/footers                         │       │
│  └─────────────────────────────────────────────────────┘       │
│                         │                                       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ 7C: Estructura de Carpetas                          │       │
│  │ HC_0001/                                            │       │
│  │ ├── HC_anonimizada.md                              │       │
│  │ ├── HC_anonimizada.pdf                             │       │
│  │ ├── imagenes/                                      │       │
│  │ │   ├── eco_abd_anon.png                          │       │
│  │ │   ├── tomo_torax_anon.dcm                       │       │
│  │ ├── informes/                                      │       │
│  │ │   ├── laboratorio_anon.pdf                      │       │
│  │ └── auditoria/                                     │       │
│  │     ├── detalle_anonimizacion.md                  │       │
│  │     └── log_operaciones.json                      │       │
│  └─────────────────────────────────────────────────────┘       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│                   SALIDA FINAL                                 │
│  • Historia clínica anonimizada lista para:                    │
│    - Análisis clínico                                          │
│    - Investigación académica                                   │
│    - Entrenamiento de modelos de IA/LLM                        │
│    - Bases de datos externas (MediGemma, OpenClients, etc)    │
│    - Integración con RAG local                                 │
│  • 100% auditable y documentado                                │
│  • Conforme Ley 25.326 + Sondeck-Laurent                      │
└────────────────────────────────────────────────────────────────┘
```

---

## Módulos del Sistema

### **Módulo 1: Parsers (Ingesta Multiformato)**
- **Función:** Leer entrada en múltiples formatos
- **Entrada:** TXT, CSV, JSON, DOCX, PDF, DICOM, JPG, PNG
- **Salida:** Estructura normalizada interna `{campo, valor, tipo, metadata}`
- **Dependencias:** Built-in + docx, PyPDF2, Pillow, pydicom
- **Notas:** Cada formato tiene su propio función parse_*()

### **Módulo 2: Configuración (Cargar Reglas)**
- **Función:** Leer rules_config.yaml y seleccionar perfil institucional
- **Entrada:** YAML, nombre de perfil
- **Salida:** Dict con todas las reglas, métodos, parámetros
- **Dependencias:** PyYAML
- **Notas:** Versión y auditoría incorporadas

### **Módulo 3: Anonimización Textual**
- **Función:** Aplicar reglas de supresión, generalización, agrupamiento
- **Subfunciones:**
  - `suprimir_idd()` → Reemplazar por [SUPRIMIDO]
  - `generalizar_edad()` → Convertir a rango
  - `generalizar_fechas()` → Convertir a año
  - `generalizar_ubicacion()` → Convertir a provincia
  - `agrupar_diagnostico_raro()` → CIE-10 nivel superior
- **Entrada:** Datos textuales + config
- **Salida:** Datos anonimizados + log de transformaciones

### **Módulo 4A: Hash Irreversible**
- **Función:** Generar ID anonimizado único
- **Algoritmo:** SHA-256 + salt
- **Entrada:** ID clínico original
- **Salida:** HC_{8_CHARS_HASH}
- **Garantía:** Irreversible, sin colisiones locales
- **Dependencias:** hashlib

### **Módulo 4B: Anonimización Multimedia**
- **Subfunciones:**
  - `anonimizar_dicom()` → pydicom: eliminar tags sensibles
  - `anonimizar_imagen()` → PIL + Pillow: limpiar EXIF, detectar OCR
  - `anonimizar_pdf()` → PyPDF2: limpiar metadata, buscar PII
  - `anonimizar_docx()` → python-docx: eliminar propiedades
- **Entrada:** Archivo multimedia
- **Salida:** Archivo con metadata limpia + log de cambios

### **Módulo 5: Evaluación de Riesgo (Sondeck-Laurent)**
- **Subfunciones:**
  - `clasificar_atributos()` → IDD/QI/AS
  - `calcular_matriz_riesgo()` → Severidad × Exposición
  - `validar_k_anonimato()` → Verificar k ≥ 5
  - `estimar_riesgo_residual()` → BAJO/MEDIO/ALTO
- **Entrada:** Datos anonimizados + config
- **Salida:** Matriz de riesgo + estimación cualitativa + justificación

### **Módulo 6: Auditoría y Logs**
- **Subfunciones:**
  - `registrar_operacion()` → SQLite
  - `generar_tabla_intervenciones()` → Lista completa de reglas/métodos
  - `generar_json_log()` → Exportar trazabilidad
  - `cifrar_logs()` → AES-256
- **Entrada:** Datos del proceso
- **Salida:** Base de datos + JSON + confirmación de trazabilidad

### **Módulo 7: Exportación**
- **Subfunciones:**
  - `generar_markdown()` → Crear .md con tabla y auditoría
  - `generar_pdf()` → Crear .pdf profesional
  - `crear_estructura_carpetas()` → Organizar archivos
  - `guardar_imagenes_procesadas()` → Copiar a carpeta destino
- **Entrada:** Datos anonimizados + logs + config de exportación
- **Salida:** Carpeta estructurada con MD/PDF + imágenes + auditoría

---

## Flujos Secundarios

### **Flujo A: Validaciones**
- Pre-anonimización: Verificar formato, campos requeridos, encoding
- Post-anonimización: Verificar no quedan IDD, validar k, confirmar richness
- Al exportar: Validar estructura, integridad, metadata limpia

### **Flujo B: Manejo de Errores**
- Si ocurre error en parsing → Marcar archivo, loguear, continuar
- Si k-anonimato falla → ALERTA, marcar para revisión manual
- Si detección de PII fallida → Marcar para revisión, no exportar sin aprobación

### **Flujo C: Integración Futura (RAG Local)**
- Los datos anonimizados pueden alimentar un sistema RAG (Ollama + ChromaDB)
- El RAG recuperaría normativas/metodología Sondeck-Laurent para guiar decisiones
- La arquitectura actual es compatible pero no la implementa en v1.0

---

## Requisitos de Hardware y Dependencias

### Hardware Mínimo
- CPU: i5 generación 2-3+
- RAM: 8 GB
- Almacenamiento: 500 MB (aplicación + deps)
- Sin GPU requerida

### Dependencias Python
```
Python 3.8+
PyYAML >= 6.0
Pillow >= 9.0
pydicom >= 2.4
PyPDF2 >= 3.0
python-docx >= 0.8.11
cryptography >= 41.0
```

### Dependencias Opcionales
```
# Para OCR en imágenes (detección de PII en texto incrustado)
pytesseract >= 0.3.10

# Para exportación PDF avanzada
WeasyPrint >= 60.0  # Alternativa a ReportLab

# Para interfaz GUI futura
PyQt5 >= 5.15
```

---

## Ciclo de Vida de Procesamiento

```
1. USUARIO CARGA ARCHIVO
   ↓
2. VALIDACIÓN INICIAL
   ├─ ✓ Continuar
   └─ ✗ Error → Loguear + Alertar
   ↓
3. PARSING
   ↓
4. CARGA DE CONFIGURACIÓN (rules_config.yaml)
   ↓
5. ANONIMIZACIÓN TEXTUAL + MULTIMEDIA
   ├─ Aplicar supresión/generalización
   ├─ Hash ID
   ├─ Limpiar metadata
   └─ Registrar cada intervención
   ↓
6. EVALUACIÓN DE RIESGO
   ├─ Clasificar atributos
   ├─ Calcular matriz
   ├─ Validar k-anonimato
   └─ Estimar riesgo residual
   ↓
7. AUDITORÍA
   ├─ Registrar en SQLite
   ├─ Generar tabla de intervenciones
   └─ Cifrar logs
   ↓
8. EXPORTACIÓN
   ├─ Crear estructura de carpetas
   ├─ Generar Markdown + PDF
   ├─ Copiar imágenes procesadas
   └─ Guardar logs
   ↓
9. SALIDA FINAL
   ├─ Carpeta HC_0001/ lista
   ├─ Documentación exhaustiva
   ├─ Auditoría completa
   └─ Listo para uso secundario

```

---

## Seguridad y Privacidad por Diseño

1. **Datos nunca salen del servidor (modo local)**
   - No hay conexión a internet por defecto
   - Modo híbrido requiere aprobación explícita

2. **Anonimización irreversible**
   - SHA-256 + salt para IDs
   - Múltiples capas (supresión + generalización + agrupamiento)
   - Validación k-anonimato

3. **Logs cifrados**
   - AES-256 en SQLite
   - Trazabilidad completa sin exposición de PII

4. **Auditoría documentada**
   - Cada intervención justificada
   - Riesgo estimado según metodología formal
   - Confirmación de riqueza clínica preservada

---

## Próximas Fases (v2.0+)

- [ ] Interfaz GUI simple (PyQt5)
- [ ] Sistema multiusuario con permisos
- [ ] Integración RAG local opcional (Ollama + ChromaDB)
- [ ] Dashboard de auditoría
- [ ] API REST para integración con EMR
- [ ] Validación automática de ataques de reidentificación

---

**Documento de Arquitectura v1.0**  
**Última actualización:** 2025-11-14
