# Configuración de Reglas de Anonimización - rules_config.yaml

**Versión:** 1.0.0  
**Última actualización:** 2025-11-14  
**Descripción:** Archivo parametrizable para reglas de anonimización según Ley 25.326 y Sondeck-Laurent

---

## 1. CONFIGURACIÓN GENERAL

```yaml
general:
  version: "1.0.0"
  nombre_proyecto: "Anonimizador Local de Historias Clínicas"
  institucion: "UBA - CITEP / Hospital de Clínicas José de San Martín"
  modo_operacion: "local"  # Opciones: "local", "hibrido"
  timestamp_formato: "%Y-%m-%d %H:%M:%S"
  encoding: "utf-8"
```

---

## 2. IDENTIFICADORES DIRECTOS (IDD) - SUPRESIÓN OBLIGATORIA

### Nombre Completo
```yaml
nombre_completo:
  campo: "nombre"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Alta"
  metodo: "supresion"
  descripcion: "Nombre y apellido del paciente"
  base_normativa: "Ley 25.326, Art. 2"
  valor_salida: "[SUPRIMIDO]"
```

### DNI
```yaml
dni:
  campo: "dni"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Alta"
  metodo: "supresion"
  descripcion: "Documento Nacional de Identidad"
  base_normativa: "Disposición 11/2006"
  valor_salida: "[SUPRIMIDO]"
```

### CUIL/CUIT
```yaml
cuil:
  campo: "cuil"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Alta"
  metodo: "supresion"
  descripcion: "CUIL/CUIT del paciente"
  base_normativa: "Disposición 11/2006"
  valor_salida: "[SUPRIMIDO]"
```

### Dirección Exacta
```yaml
direccion_exacta:
  campo: "direccion"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Media"
  metodo: "supresion"
  descripcion: "Dirección completa (calle, número, piso, departamento)"
  base_normativa: "Ley 25.326, Art. 7"
  valor_salida: "[SUPRIMIDO]"
```

### Teléfono
```yaml
telefono:
  campo: "telefono"
  tipo: "IDD"
  severidad: "Media"
  exposicion: "Media"
  metodo: "supresion"
  descripcion: "Teléfono de contacto"
  base_normativa: "Disposición 11/2006"
  valor_salida: "[SUPRIMIDO]"
```

### Email
```yaml
email:
  campo: "email"
  tipo: "IDD"
  severidad: "Media"
  exposicion: "Alta"
  metodo: "supresion"
  descripcion: "Dirección de correo electrónico"
  base_normativa: "Disposición 11/2006"
  valor_salida: "[SUPRIMIDO]"
```

### Número de Afiliación
```yaml
numero_afiliacion:
  campo: "numero_afiliacion"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Alta"
  metodo: "supresion"
  descripcion: "Número de afiliación a obra social/prepaga"
  base_normativa: "Ley 25.326, Art. 2"
  valor_salida: "[SUPRIMIDO]"
```

### Número de Historia Clínica
```yaml
numero_hc_interno:
  campo: "numero_hc"
  tipo: "IDD"
  severidad: "Alta"
  exposicion: "Alta"
  metodo: "hash_irreversible"
  descripcion: "Número de historia clínica interno"
  base_normativa: "Disposición 11/2006"
  algoritmo: "SHA-256"
  salt: "institucional_secret_2025"
  truncar_a: 8
  prefijo: "HC_"
```

---

## 3. CUASI-IDENTIFICADORES (QI) - GENERALIZACIÓN Y AGRUPAMIENTO

### Edad
```yaml
edad:
  campo: "edad"
  tipo: "QI"
  severidad: "Media"
  exposicion: "Alta"
  metodo: "agrupamiento"
  descripcion: "Edad exacta del paciente"
  base_normativa: "Sondeck-Laurent, metodología k-anonimato"
  parametros:
    rango_anos: 5
    formato_salida: "{edad_min}-{edad_max}"
    ejemplo: "37 años → 35-39 años"
  grupos:
    - min: 0
      max: 4
      etiqueta: "0-4"
    - min: 5
      max: 9
      etiqueta: "5-9"
    - min: 10
      max: 14
      etiqueta: "10-14"
    - min: 15
      max: 19
      etiqueta: "15-19"
    - min: 20
      max: 24
      etiqueta: "20-24"
    - min: 25
      max: 29
      etiqueta: "25-29"
    - min: 30
      max: 34
      etiqueta: "30-34"
    - min: 35
      max: 39
      etiqueta: "35-39"
    - min: 40
      max: 44
      etiqueta: "40-44"
    - min: 45
      max: 49
      etiqueta: "45-49"
    - min: 50
      max: 54
      etiqueta: "50-54"
    - min: 55
      max: 59
      etiqueta: "55-59"
    - min: 60
      max: 64
      etiqueta: "60-64"
    - min: 65
      max: 69
      etiqueta: "65-69"
    - min: 70
      max: 74
      etiqueta: "70-74"
    - min: 75
      max: 79
      etiqueta: "75-79"
    - min: 80
      max: 120
      etiqueta: "80+"
```

### Fecha de Nacimiento
```yaml
fecha_nacimiento:
  campo: "fecha_nacimiento"
  tipo: "QI"
  severidad: "Alta"
  exposicion: "Media"
  metodo: "generalizacion_temporal"
  descripcion: "Fecha de nacimiento exacta"
  base_normativa: "Metodología Sondeck-Laurent"
  parametros:
    formato_entrada: "%d/%m/%Y"
    formato_salida: "%Y"
    ejemplo: "15/03/1985 → 1985"
    eliminar_dia_mes: true
```

### Fecha de Atención
```yaml
fecha_atencion:
  campo: "fecha_atencion"
  tipo: "QI"
  severidad: "Media"
  exposicion: "Media"
  metodo: "generalizacion_temporal"
  descripcion: "Fecha exacta de la consulta/internación"
  base_normativa: "Sondeck-Laurent"
  parametros:
    formato_entrada: "%d/%m/%Y"
    formato_salida: "%Y"
    ejemplo: "14/11/2025 → 2025"
    eliminar_dia_mes: true
```

### Código Postal
```yaml
codigo_postal:
  campo: "codigo_postal"
  tipo: "QI"
  severidad: "Media"
  exposicion: "Alta"
  metodo: "generalizacion_geografica"
  descripcion: "Código postal exacto"
  base_normativa: "Metodología Sondeck-Laurent, exposición geográfica"
  parametros:
    suprimir_campo: true
    mapeo_provincia: true
    formato_salida: "provincia"
```

### Localidad
```yaml
localidad:
  campo: "localidad"
  tipo: "QI"
  severidad: "Media"
  exposicion: "Alta"
  metodo: "generalizacion_geografica"
  descripcion: "Localidad/ciudad exacta"
  base_normativa: "Sondeck-Laurent"
  parametros:
    suprimir_campo: true
    mapeo_provincia: true
    formato_salida: "provincia"
```

### Provincia
```yaml
provincia:
  campo: "provincia"
  tipo: "QI"
  severidad: "Media"
  exposicion: "Media"
  metodo: "retener"
  descripcion: "Provincia (nivel de agregación aceptable)"
  base_normativa: "Minimización de datos + utilidad epidemiológica"
  parametros:
    mantener: true
    mapeado_desde: "localidad"
```

### Sexo
```yaml
sexo:
  campo: "sexo"
  tipo: "QI"
  severidad: "Baja"
  exposicion: "Baja"
  metodo: "retener"
  descripcion: "Sexo del paciente (M/F/X)"
  base_normativa: "Utilidad clínica"
  parametros:
    mantener: true
```

### Diagnóstico Raro
```yaml
diagnostico_raro:
  campo: "diagnostico_principal"
  tipo: "QI"
  severidad: "Alta"
  exposicion: "Baja"
  metodo: "agrupamiento_diagnostico"
  descripcion: "Diagnóstico con prevalencia < 1% en base institucional"
  base_normativa: "Sondeck-Laurent, atributo sensible"
  parametros:
    umbral_prevalencia: 0.01  # 1%
    accion: "generalizar_a_categoria_superior"
    clasificacion: "CIE-10"
    ejemplo: "G20.1 (Parkinson primario) → G20 (Enfermedad de Parkinson)"
```

---

## 4. ATRIBUTOS SENSIBLES (AS) - MÁXIMA PROTECCIÓN

### Datos de Salud (Diagnóstico)
```yaml
datos_salud:
  campo: "diagnostico"
  tipo: "AS"
  severidad: "Alta"
  base_normativa: "Ley 25.326, Art. 9"
  metodo: "proteccion_reforzada"
  descripcion: "Información de diagnóstico y estado de salud"
  medidas:
    - generalizacion_cie10_superior
    - agrupamiento_si_raro
    - revisar_manualmente_antes_exportacion
    - revisar_manualmente_si_prevalencia_baja: true
```

### Datos Genéticos
```yaml
datos_geneticos:
  campo: "genetico"
  tipo: "AS"
  severidad: "Alta"
  base_normativa: "Ley 25.326, Art. 9"
  metodo: "supresion"
  descripcion: "Información genética, antecedentes familiares heredables"
  medidas:
    - supresion_total
    - solo_si_indispensable_para_investigacion
    - requiere_consentimiento_expreso
```

### Origen Étnico
```yaml
origen_etnico:
  campo: "origen_etnico"
  tipo: "AS"
  severidad: "Alta"
  base_normativa: "Ley 25.326, Art. 9"
  metodo: "supresion_o_generalizacion"
  descripcion: "Origen étnico o ancestral"
  medidas:
    - generalizacion_a_region_amplia
    - supresion_si_no_relevante_clinico
```

### Orientación Sexual
```yaml
orientacion_sexual:
  campo: "orientacion_sexual"
  tipo: "AS"
  severidad: "Alta"
  base_normativa: "Ley 25.326, Art. 9"
  metodo: "supresion"
  descripcion: "Orientación sexual del paciente"
  medidas:
    - supresion_total
    - solo_si_clinicamente_relevante
```

---

## 5. PROCESAMIENTO DE MULTIMEDIA

### DICOM
```yaml
DICOM:
  descripcion: "Imágenes médicas en formato DICOM"
  extension: ".dcm"
  tags_a_eliminar:
    - tag: "(0010,0010)"
      nombre: "PatientName"
      descrip: "Nombre del paciente"
    - tag: "(0010,0020)"
      nombre: "PatientID"
      descrip: "ID del paciente"
    - tag: "(0008,0080)"
      nombre: "InstitutionName"
      descrip: "Nombre de institución"
    - tag: "(0008,0090)"
      nombre: "ReferringPhysicianName"
      descrip: "Nombre del médico"
    - tag: "(0008,0020)"
      nombre: "StudyDate"
      descrip: "Fecha del estudio (convertir a año)"
    - tag: "(0008,0030)"
      nombre: "StudyTime"
      descrip: "Hora del estudio"
    - tag: "(0010,0030)"
      nombre: "PatientBirthDate"
      descrip: "Fecha de nacimiento del paciente"
  herramienta: "pydicom"
  encoding: "utf-8"
  preserve_image_data: true
```

### JPG/PNG
```yaml
JPG_PNG:
  descripcion: "Imágenes estándar (fotografías, gráficos)"
  extensiones: [".jpg", ".jpeg", ".png", ".bmp", ".gif"]
  metadata_exif_eliminar:
    - GPS_Info
    - Author
    - Copyright
    - DateTime
    - UserComment
  herramienta: "PIL/Pillow + exiftool"
  detectar_texto_en_imagen: true
  metodo_deteccion_texto: "tesseract_ocr"
  accion_texto_detectado: "marcar_para_revision_manual"
```

### PDF
```yaml
PDF:
  descripcion: "Documentos PDF (informes, consentimientos)"
  extension: ".pdf"
  propiedades_eliminar:
    - Author
    - Title
    - Subject
    - Keywords
    - CreationDate
    - ModDate
  buscar_pii_en_texto: true
  herramienta: "PyPDF2"
  encoding: "utf-8"
```

### DOCX
```yaml
DOCX:
  descripcion: "Documentos Word (.docx)"
  extension: ".docx"
  eliminar_propiedades:
    - author
    - subject
    - comments
    - created
  eliminar_revision_track: true
  herramienta: "python-docx"
```

---

## 6. MODELO DE PRIVACIDAD - K-ANONIMATO

```yaml
privacidad_k_anonimato:
  habilitado: true
  k_minimo: 5
  atributos_combinacion_critica:
    - edad
    - provincia
    - sexo
    - diagnostico_principal_categoria
  metodo_validacion: "verificar_clases_equivalencia"
  accion_si_k_insuficiente: "alertar_y_marcar_para_revision"
```

---

## 7. AUDITORÍA Y LOGS

```yaml
auditoria:
  base_datos: "sqlite"
  archivo: "logs/auditoria.db"
  tabla_operaciones: "operaciones_anonimizacion"
  campos_registro:
    - id (autoincrement)
    - timestamp
    - usuario
    - archivo_entrada
    - archivo_salida
    - versión_reglas
    - reglas_aplicadas
    - grado_intervención
    - riesgo_residual_estimado
    - estado (exitoso/error)
    - observaciones
  cifrado_logs: true
  algoritmo_cifrado: "AES-256"
```

---

## 8. EXPORTACIÓN

```yaml
exportacion:
  formatos_soportados:
    - markdown
    - pdf
    - ambos
  
  markdown:
    extension: ".md"
    encoding: "utf-8"
    incluir_tabla_auditoria: true
    incluir_metadata_proceso: true
    incluir_observaciones: true
  
  pdf:
    generador: "ReportLab"  # Alternativa: "WeasyPrint"
    extension: ".pdf"
    encoding: "utf-8"
    incluir_imagenes_embebidas: false
    estilos: "profesional"
    incluir_tabla_auditoria: true
    incluir_metadata_proceso: true
  
  estructura_carpetas:
    raiz: "HC_{{ id_anonimizado }}"
    subcarpetas:
      - "imagenes_anonimizadas"
      - "informes_anonimizados"
      - "auditoria"
    archivos:
      - "HC_anonimizada.md"
      - "HC_anonimizada.pdf"
      - "auditoria/detalle_anonimizacion.md"
      - "auditoria/log_operaciones.json"
```

---

## 9. MODOS DE OPERACIÓN

```yaml
modos:
  local:
    descripcion: "100% offline, datos nunca salen del servidor"
    permitir_exportacion_internet: false
    logs_cifrados: true
    modo_por_defecto: true
  
  hibrido:
    descripcion: "Local + opcional integración con APIs externas (previa aprobación)"
    permitir_exportacion_internet: true
    requiere_aprobacion_explicita: true
    registro_transferencia_obligatorio: true
    solo_datos_anonimizados: true
```

---

## 10. VALIDACIONES Y CHEQUEOS

```yaml
validaciones:
  pre_anonimizacion:
    - verificar_formato_entrada
    - detectar_campos_requeridos
    - validar_encoding
    - buscar_pii_obvias_no_mapeadas

  post_anonimizacion:
    - verificar_no_quedan_idd
    - validar_k_anonimato
    - confirmar_riqueza_diagnostica
    - verificar_hash_irreversible

  al_exportar:
    - validar_estructura_carpetas
    - verificar_integridad_archivos
    - confirmar_metadata_limpia
    - generar_checksum
```

---

## 11. PERFILES INSTITUCIONALES

```yaml
perfiles:
  default:
    nombre: "Perfil General - Configuración Estándar"
    aplica_todas_reglas: true
  
  uba_hcjsm_enfermeria:
    nombre: "UBA HCJSM - Carrera de Enfermería"
    institucion: "Hospital de Clínicas José de San Martín"
    departamento: "Carrera de Enfermería"
    reglas_adicionales:
      - enfoque_atencion_primaria
      - proteccion_reforzada_diagnosticos_raros
```

---

## 12. NOTAS Y REFERENCIAS

- Este archivo es parametrizable y versionado
- Cualquier cambio debe ser documentado en CHANGELOG.yaml
- La modificación de reglas requiere revisión por comité de ética
- Se recomienda auditoría anual de esta configuración
- Para agregar nuevas reglas, contactar al responsable técnico

---

**Documento v1.0 - Markdown Version**  
**Convertido para NotebookLM: 2025-11-14**
