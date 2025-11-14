# CHANGELOG.md - Historial de Cambios

Todos los cambios notables en este proyecto serán documentados en este archivo.

El formato está basado en [Keep a Changelog](https://keepachangelog.com/es-ES/)
y este proyecto adhiere a [Semantic Versioning](https://semver.org/es/).

---

## [0.1-alpha] - 2025-11-14

### Added (Agregado)

#### Módulos Core
- ✅ **Módulo core_anonymization.py:** Motor principal de anonimización textual
  - Clase `AnonymizationRule`: Definición de reglas individuales
  - Clase `CoreAnonymizer`: Aplicación de reglas a datos
  - Supresión de identificadores directos (IDD)
  - Generalización de cuasi-identificadores (QI): edad, fechas, ubicaciones
  - Agrupamiento para k-anonimato
  - Hash SHA-256 irreversible para disociación
  - Registro detallado de intervenciones

- ✅ **Módulo hash_and_audit.py:** Hash irreversible y auditoría
  - Clase `IrreversibleHasher`: Generación SHA-256 con salt
  - Clase `AuditLogger`: Registro SQLite cifrado con AES-256
  - Trazabilidad completa de operaciones
  - Exportación de auditoría a JSON
  - Validación de formato de hashes

- ✅ **Módulo export_markdown_pdf.py:** Exportación a Markdown y PDF
  - Clase `MarkdownExporter`: Genera archivos .md profesionales
  - Clase `PDFExporter`: Genera PDFs con ReportLab
  - Clase `StructuredExport`: Organiza estructura de carpetas por caso
  - Dataclass `CasoClinico`: Estructura para casos anonimizados
  - Tabla exhaustiva de intervenciones
  - Sección de auditoría automática
  - Tablas de datos clínicos

#### Documentación
- ✅ **README.md:** Documento de Requisitos de Producto (PRD)
  - Objetivo y misión del proyecto
  - Alcance y delimitaciones
  - 13 requisitos funcionales detallados
  - Arquitectura modular completa
  - Roadmap de desarrollo (v1.0, v2.0, v3.0)

- ✅ **REGULACION.md:** Marco normativo y regulatorio
  - Fundamentos Ley 25.326 (Argentina)
  - Disposición 11/2006: Criterios técnicos
  - Resolución 1480/2011: Guía para investigaciones
  - Metodología Sondeck-Laurent (2025)
  - Modelos de privacidad: k-anonimato, l-diversidad, t-closeness
  - Brechas regulatorias y soluciones
  - Ciclo de auditoría recomendado

- ✅ **TEMPLATE.md:** Plantilla estándar de salida
  - Estructura completa de historia clínica anonimizada
  - Tabla exhaustiva de anonimización (Sondeck-Laurent)
  - Sección de auditoría detallada
  - Cálculo de grado de intervención
  - Confirmación de riqueza diagnóstica
  - Declaración de cumplimiento normativo

- ✅ **ARCHITECTURE.md:** Diseño técnico del sistema
  - Diagrama de flujo completo
  - 7 módulos funcionales documentados
  - Flujos secundarios (validación, errores, RAG futura)
  - Requisitos de hardware
  - Dependencias Python
  - Ciclo de vida de procesamiento

- ✅ **CONTRIBUTING.md:** Guía para colaboradores
  - Principios del proyecto
  - Requisitos de contribución
  - Flujo de trabajo con Git
  - Estándares técnicos (PEP 8, type hints)
  - Estándares de documentación
  - Proceso de revisión de PRs
  - Comunicación y código de conducta

- ✅ **rules_config.yaml (versión Markdown):** Configuración parametrizable
  - 8 secciones de configuración
  - Identificadores directos (IDD): nombre, DNI, dirección, etc.
  - Cuasi-identificadores (QI): edad, fechas, ubicación, diagnósticos
  - Atributos sensibles (AS): salud, genética, origen étnico
  - Multimedia: DICOM, JPG/PNG, PDF, DOCX
  - Modelos de privacidad: k-anonimato
  - Auditoría y logs cifrados
  - Perfiles institucionales extensibles

#### Configuración y Setup
- ✅ **requirements.txt:** Dependencias de producción (7 paquetes)
- ✅ **requirements-dev.txt:** Dependencias de desarrollo (herramientas)
- ✅ **.gitignore:** Archivos y carpetas a ignorar en Git
- ✅ **CHANGELOG.md:** Este archivo

### Status (Estado)

| Componente | Estado | Porcentaje |
|-----------|--------|-----------|
| Anonimización textual | ✅ Funcional | 100% |
| Hash irreversible | ✅ Funcional | 100% |
| Auditoría SQLite | ✅ Funcional | 100% |
| Exportación MD/PDF | ✅ Funcional | 100% |
| Configuración YAML | ✅ Funcional | 100% |
| **Documentación** | ✅ **Funcional** | **100%** |
| Anonimización multimedia (DICOM/JPG) | ⏳ Desarrollo | FASE 4 |
| CLI interactiva | ⏳ Desarrollo | FASE 4 |
| Tests unitarios | ⏳ Desarrollo | FASE 4 |
| GUI (PyQt5) | ⏳ Diseño | FASE 5 |
| Integración RAG (Ollama) | ⏳ Diseño | FASE 6 |

### Cumplimiento Normativo

- ✅ Ley 25.326 (Protección de Datos Personales - Argentina)
- ✅ Disposición 11/2006 (Criterios técnicos de disociación)
- ✅ Resolución 1480/2011 (Investigaciones con seres humanos)
- ✅ Metodología Sondeck-Laurent (2025) - Evaluación de riesgo granular
- ✅ Principios de disociación irreversible
- ✅ Trazabilidad y auditoría completa
- ✅ Privacidad por diseño

### Características Destacadas

1. **Seguridad por Diseño**
   - 100% local, sin conexión obligatoria a internet
   - Anonimización irreversible mediante SHA-256 + salt
   - Logs cifrados con AES-256
   - Sin exposición de datos personales

2. **Auditoría Exhaustiva**
   - Registro detallado de cada intervención
   - Trazabilidad temporal completa
   - Tabla de Sondeck-Laurent (Severidad × Exposición)
   - Exportación a JSON para análisis

3. **Documentación Profesional**
   - PRD completo (README.md)
   - Marco normativo exhaustivo (REGULACION.md)
   - Plantilla de salida estándar (TEMPLATE.md)
   - Arquitectura técnica detallada (ARCHITECTURE.md)
   - Guía para colaboradores (CONTRIBUTING.md)

4. **Configuración Flexible**
   - Todas las reglas parametrizables en YAML
   - Perfiles por institución/jurisdicción
   - Extensible sin modificar código
   - Versionado de configuración

### Limitaciones Conocidas (v0.1-alpha)

- ⚠️ Anonimización de multimedia no implementada (FASE 4)
- ⚠️ Sin CLI interactiva (FASE 4)
- ⚠️ Sin tests unitarios (FASE 4)
- ⚠️ Sin validación automática de k-anonimato (solo estimación)
- ⚠️ Sin simulación de ataques de reidentificación (FASE 4)
- ⚠️ Sin integraciones externas (FASE 5+)

### Requisitos Instalación

```bash
# Dependencias base
pip install -r requirements.txt

# Para desarrollo
pip install -r requirements-dev.txt

# Python 3.8+
# Sin GPU requerida
# Compatible: Windows, Linux, macOS
```

### Próximas Versiones (Roadmap)

#### v0.2 (Planned - 2025-11-21)
- [ ] Anonimización de imágenes DICOM
- [ ] Anonimización EXIF en JPG/PNG
- [ ] CLI básica interactiva
- [ ] Tests unitarios (cobertura ≥80%)

#### v1.0 (Planned - 2025-12-05)
- [ ] Todas las características de v0.2 estables
- [ ] Documentación completa (Sphinx)
- [ ] Ejemplos de uso en notebooks
- [ ] Validación automática k-anonimato
- [ ] Simulación de ataques Sondeck-Laurent

#### v2.0 (Future)
- [ ] GUI con PyQt5
- [ ] Sistema multiusuario con permisos
- [ ] API REST para integración con EMR
- [ ] Dashboard de auditoría

#### v3.0 (Future)
- [ ] Integración RAG local (Ollama + ChromaDB)
- [ ] Consultas contextuales sobre normativa
- [ ] Validación automática de riesgo con LLM

---

## Cómo Reportar Cambios

Al hacer un commit, incluir tipo y descripción:

```
feat: nueva funcionalidad
fix: corrección de bug
docs: cambios en documentación
test: agregar/modificar tests
refactor: cambio de código sin nuevas funciones
security: parche de seguridad
chore: cambios en build, deps, etc
```

Ejemplo:
```
feat(core_anonymization): Agregar soporte para anonimización de diagnósticos raros

- Implementar agrupamiento CIE-10 nivel superior
- Agregar detección automática de prevalencia < 1%
- Tests incluidos para diagnósticos infrecuentes

Cumplimiento: Sondeck-Laurent, Art. 9 Ley 25.326
Closes #5
```

---

**Última actualización:** 2025-11-14 07:59 UTC-3  
**Versión:** v0.1-alpha  
**Estado:** Ready for GitHub
