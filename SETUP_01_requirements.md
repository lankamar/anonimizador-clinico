# requirements.txt - Dependencias del Proyecto

## Dependencias Core (Producción)

```
PyYAML>=6.0
Pillow>=9.0
pydicom>=2.4
PyPDF2>=3.0
python-docx>=0.8.11
cryptography>=41.0
reportlab>=4.0
```

## Descripción de cada dependencia

| Paquete | Versión | Función | Uso |
|---------|---------|---------|-----|
| PyYAML | ≥6.0 | Lectura de rules_config.yaml | Configuración de reglas |
| Pillow | ≥9.0 | Procesamiento de imágenes JPG/PNG | Anonimización de EXIF (FASE 4) |
| pydicom | ≥2.4 | Procesamiento DICOM | Anonimización de imágenes médicas (FASE 4) |
| PyPDF2 | ≥3.0 | Manipulación de PDFs | Limpieza de metadata, extracción de texto |
| python-docx | ≥0.8.11 | Procesamiento de DOCX | Eliminación de propiedades, limpieza |
| cryptography | ≥41.0 | Cifrado Fernet AES-256 | Cifrado de logs de auditoría |
| reportlab | ≥4.0 | Generación de PDFs | Exportación a PDF con estilos profesionales |

## Instalación

```bash
# Instalar todas las dependencias
pip install -r requirements.txt

# O para desarrollo (incluyendo herramientas)
pip install -r requirements-dev.txt
```

## Notas de Compatibilidad

- Python: 3.8+
- SO: Windows, Linux, macOS
- Sin GPU requerida
- Sin dependencias cloud (100% local)

## Dependencias Opcionales (Futuro)

```
# Para OCR en imágenes (FASE 4)
pytesseract>=0.3.10

# Para interfaz GUI (FASE 5)
PyQt5>=5.15

# Para integración con RAG local (FASE 6)
ollama>=0.1.0
chromadb>=0.3.0
```
