# Anonimizador Clinico Local - Multiformato

**Cumplimiento: Ley 25.326 (Argentina) | Disposicion 11/2006 | Resolucion 1480/2011 | Metodologia Sondeck-Laurent (2025)**

## Objetivo General

Desarrollar una aplicacion **local, auditable y modular en Python** que procese historias clinicas (texto estructurado/libre) e imagenes/estudios complementarios (DICOM, JPG/PNG, PDF, DOCX), generando copias fieles y completamente anonimizada conforme a la regulacion argentina y metodologia Sondeck-Laurent.

## Alcance

**Entra:**
- Historias clinicas en TXT, CSV, JSON, DOCX, PDF
- Imagenes medicas (JPG/PNG/DICOM)
- Informes y estudios complementarios

**Sale:**
- HC anonimizada fiel (sin resumen, sin interpretacion)
- Adjuntos anonimizados (metadatos limpios)
- Reporte exhaustivo de auditoria
- Exportacion en Markdown y/o PDF

## Requisitos Funcionales

### F1. Ingesta y Parsing
- Parser modular para cada formato (TXT, CSV, JSON, PDF, DOCX, DICOM, JPG/PNG)
- Extrae campos estructurados o reconoce contexto en texto libre

### F2. Anonimizacion Textual
- **Supresion:** Nombre, DNI, direccion, telefono, email, datos de contacto
- **Generalizacion:** Fechas (a ano), geolocalizacion (solo provincia)
- **Agrupamiento:** Edad (rangos 5 anos), diagnosticos raros, variables sensibles
- **Hash irreversible:** ID clinico reemplazado por codigo unico

### F3. Anonimizacion Multimedia
- Limpieza DICOM tags (pydicom): PatientName, InstitutionName, StudyDate, etc.
- Remocion EXIF en JPG/PNG (PIL, exiftool)
- Limpieza de propiedades PDF/DOCX
- Edicion manual de leyendas/texto sobre imagen (si hay PII)

### F4. Auditoria y Documentacion
- Registro automatico de **cada regla aplicada**
- Seccion "Detalle de Anonimizacion" generada dinamicamente
- Grado de intervencion (Minima/Media/Alta)
- Justificacion en base a Sondeck-Laurent
- Log cifrado SQLite/JSON con trazabilidad completa

### F5. Exportacion Compatible
- Markdown con tabla de transformaciones e imagenes embebidas
- PDF profesional con auditoria integrada
- Compatibles con LLM, RAG, MediGemma, openClients
- Sin exposicion de identificadores en ningun flujo

## Regulacion y Cumplimiento

| Normativa | Requisito | Implementacion |
|-----------|-----------|-------------------|
| **Ley 25.326** | Disociacion irreversible de datos | Hash SHA-256 + salt, nunca reversible |
| **Disposicion 11/2006** | Criterios tecnicos de anonimizacion | Sondeck-Laurent: evaluacion granular de riesgo |
| **Resolucion 1480/2011** | Revision etica institucional | Documentacion exhaustiva para Comites |
| **Sondeck-Laurent 2025** | Clasificacion Severidad/Exposicion | Motor de reglas parametrizable + reporte |

## Estado del Proyecto

**v0.1-alpha (MVP Core)**
- Anonimizacion textual: Funcional
- Hash irreversible: Funcional
- Auditoria SQLite: Funcional
- Exportacion MD/PDF: Funcional
- Anonimizacion multimedia (FASE 4): En desarrollo
- CLI interactiva (FASE 4): En desarrollo
- Tests unitarios (FASE 4): En desarrollo

## Autores y Contacto

- **Marcelo Omar Lancry k.** | [@lankamar](https://github.com/lankamar)
- **UBA - CITEP** | Hospital de Clinicas Jose de San Martin

## Licencia

MIT License - Ver [LICENSE](./LICENSE)

---

**NOTA IMPORTANTE:** Este proyecto es para uso institucional/academico. Esta disenado para cumplir estrictamente con regulaciones de proteccion de datos clinicos en Argentina. Cualquier uso debe estar supervisado por Comites de Etica Institucionales.
