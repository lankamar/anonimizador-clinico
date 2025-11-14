# Guía Paso a Paso: Lanzar a GitHub

Esta guía te explica exactamente cómo llevar todo el proyecto a GitHub HOY.

---

## Paso 1: Preparar la Estructura Local

### 1.1 Crear carpeta del proyecto

```bash
# En tu máquina local (Windows, Linux o macOS)
mkdir anonimizador-clinico
cd anonimizador-clinico

# Inicializar Git
git init
```

### 1.2 Crear estructura de carpetas

```bash
mkdir -p anonimizador tests config logs
mkdir -p docs examples

# Crear archivos __init__.py
touch anonimizador/__init__.py
touch tests/__init__.py
```

### 1.3 Copiar archivos base

```bash
# En la raíz del proyecto, crear estos archivos:
# (copiar contenido desde los MDs que te proporcioné)

# Documentación
touch README.md           # (copiar del PRD)
touch REGULACION.md       # (copiar del marco normativo)
touch TEMPLATE.md         # (copiar de la plantilla)
touch ARCHITECTURE.md     # (copiar del diagrama técnico)
touch CONTRIBUTING.md     # (copiar de la guía)
touch CHANGELOG.md        # (copiar del changelog)

# Configuración
touch requirements.txt    # (copiar del SETUP_01)
touch requirements-dev.txt # (copiar del SETUP_02)
touch .gitignore         # (copiar del SETUP_03)

# Config
touch config/rules_config.yaml  # (copiar de rules_config.md)
```

---

## Paso 2: Copiar Código Python

### 2.1 Módulo core_anonymization.py

De **FASE3_01_core_anonymization.md**:
- Copiar el bloque ```python ... ```
- Guardar en: `anonimizador/core_anonymization.py`

### 2.2 Módulo hash_and_audit.py

De **FASE3_02_hash_and_audit.md**:
- Copiar el bloque ```python ... ```
- Guardar en: `anonimizador/hash_and_audit.py`

### 2.3 Módulo export_markdown_pdf.py

De **FASE3_03_export_markdown_pdf.md**:
- Copiar el bloque ```python ... ```
- Guardar en: `anonimizador/export_markdown_pdf.py`

### 2.4 Inicializador de módulos

Crear `anonimizador/__init__.py`:

```python
"""
Anonimizador Local de Historias Clínicas

Aplicación local para anonimización de datos clínicos según
Ley 25.326, Disposición 11/2006 y metodología Sondeck-Laurent.

Version: 0.1-alpha
License: MIT
Author: Marcelo Omar Lancry Kamycki
Institution: UBA - CITEP / Hospital de Clínicas José de San Martín
"""

__version__ = "0.1-alpha"
__author__ = "Marcelo Omar Lancry Kamycki"
__email__ = "lankamar@gmail.com"

from .core_anonymization import CoreAnonymizer, AnonymizationRule, anonymize_batch
from .hash_and_audit import IrreversibleHasher, AuditLogger
from .export_markdown_pdf import (
    MarkdownExporter,
    PDFExporter,
    StructuredExport,
    CasoClinico
)

__all__ = [
    'CoreAnonymizer',
    'AnonymizationRule',
    'anonymize_batch',
    'IrreversibleHasher',
    'AuditLogger',
    'MarkdownExporter',
    'PDFExporter',
    'StructuredExport',
    'CasoClinico'
]
```

---

## Paso 3: Primeros Commits en Git

### 3.1 Commit inicial

```bash
# Agregar todos los archivos
git add .

# Verificar qué va a ser commiteado
git status

# Hacer el primer commit
git commit -m "feat: Initial MVP - Core anonymization, hash, audit, export

- Módulo core_anonymization: Anonimización textual con Sondeck-Laurent
- Módulo hash_and_audit: Hashes SHA-256 e auditoría SQLite cifrada
- Módulo export_markdown_pdf: Exportación a Markdown y PDF
- Configuración YAML parametrizable de reglas
- Documentación exhaustiva: PRD, regulaciones, arquitectura
- Tests y ejemplos básicos

Cumplimiento: Ley 25.326, Disposición 11/2006, Resolución 1480/2011
Metodología: Sondeck-Laurent (2025)
Estado: v0.1-alpha - MVP funcional

Hardware compatible: i5, 8GB RAM, sin GPU"
```

### 3.2 Verificar el commit

```bash
# Ver logs
git log --oneline

# Ver cambios
git show HEAD
```

---

## Paso 4: Crear Repositorio en GitHub

### 4.1 Ir a GitHub

- Abrir: https://github.com/new
- O si ya estás autenticado, click en "+" arriba a la derecha → "New repository"

### 4.2 Configurar repositorio

**Nombre:** `anonimizador-clinico`

**Descripción:** 
```
Local clinical data anonymizer compliant with Ley 25.326 (Argentina). 
Implements Sondeck-Laurent methodology for irreversible de-identification 
and exhaustive audit trails. Zero external connectivity.
```

**Visibilidad:** Public (para código abierto)

**Inicializar con:** 
- ❌ NO marcar "Add a README file"
- ❌ NO marcar "Add .gitignore"
- ❌ NO marcar "Add a license"
- ✅ (ya tienes estos archivos localmente)

**Botón:** "Create repository"

---

## Paso 5: Conectar Local con GitHub

### 5.1 Copiar comandos de GitHub

GitHub te mostrará algo así:

```bash
# Or push an existing repository from the command line
git remote add origin https://github.com/TU_USUARIO/anonimizador-clinico.git
git branch -M main
git push -u origin main
```

### 5.2 Ejecutar en tu terminal local

```bash
# Agregar remote
git remote add origin https://github.com/TU_USUARIO/anonimizador-clinico.git

# Cambiar rama a "main" (si estás en "master")
git branch -M main

# Hacer push (primera vez con -u)
git push -u origin main
```

### 5.3 Verificar en GitHub

- Ir a https://github.com/TU_USUARIO/anonimizador-clinico
- Deberías ver todos tus archivos

---

## Paso 6: Configuración Final de GitHub

### 6.1 Agregar Topics

En la página principal del repo (settings o sobre el código):
- Click en "Add topics"
- Agregar: `healthcare`, `privacy`, `anonymization`, `argentina`, `ley-25326`, `sondeck-laurent`, `python`

### 6.2 Configurar Branch Protection (Opcional pero Recomendado)

En Settings → Branches → Add rule:
- **Branch name pattern:** `main`
- ✅ Require a pull request before merging
- ✅ Require status checks to pass
- ✅ Require branches to be up to date

### 6.3 Habilitar Issues

Settings → General:
- ✅ Issues
- ✅ Discussions
- ✅ Sponsors

### 6.4 Agregar Licencia

En Code, arriba a la derecha: "Add file" → crear `LICENSE`:

Copiar contenido de licencia MIT (para proyectos académicos):

```
MIT License

Copyright (c) 2025 Marcelo Omar Lancry Kamycki

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Paso 7: Verificar que Todo Funciona

### 7.1 Clonar el repo (prueba)

```bash
# En otra carpeta
cd temp
git clone https://github.com/TU_USUARIO/anonimizador-clinico.git
cd anonimizador-clinico

# Instalar dependencias
pip install -r requirements.txt

# Verificar que importa
python -c "from anonimizador import CoreAnonymizer; print('✓ Importación exitosa')"
```

### 7.2 Verificar estructura

```bash
# Debería verse:
tree -L 2
# anonimizador/
# ├── __init__.py
# ├── core_anonymization.py
# ├── hash_and_audit.py
# ├── export_markdown_pdf.py
# config/
# ├── rules_config.yaml
# tests/
# ├── __init__.py
# README.md
# REGULACION.md
# CONTRIBUTING.md
# etc...
```

---

## Paso 8: Anunciar el Proyecto

### Opcionales pero Recomendados:

1. **LinkedIn (profesional):**
   ```
   Acabo de lanzar "Anonimizador Clínico Local" en GitHub:
   Una herramienta Python para anonimización de historias clínicas
   conforme Ley 25.326 (Argentina) y metodología Sondeck-Laurent.
   
   🔒 100% local, sin internet, auditable
   📋 Disociación irreversible SHA-256
   📊 Registros cifrados AES-256
   
   github.com/lankamar/anonimizador-clinico
   ```

2. **UBA/CITEP:**
   - Informar a supervisores/directores del proyecto
   - Agregar link en repositorio institucional si existe

3. **Comunidad:**
   - Dev.to post sobre el proyecto
   - Foros de IA/healthcare en español

---

## Estructura Final en GitHub

Tu repositorio verá así:

```
anonimizador-clinico/
├── README.md ......................... PRD del proyecto
├── REGULACION.md ..................... Marco normativo completo
├── ARCHITECTURE.md ................... Diseño técnico
├── TEMPLATE.md ....................... Plantilla de salida
├── CONTRIBUTING.md ................... Guía para colaboradores
├── CHANGELOG.md ...................... Historial de cambios
├── LICENSE ........................... Licencia MIT
├── requirements.txt .................. Dependencias producción
├── requirements-dev.txt .............. Dependencias desarrollo
├── .gitignore ........................ Archivos a ignorar
├── anonimizador/
│   ├── __init__.py
│   ├── core_anonymization.py
│   ├── hash_and_audit.py
│   └── export_markdown_pdf.py
├── tests/
│   └── __init__.py
├── config/
│   └── rules_config.yaml
├── logs/ ............................ (vacío, para auditoría)
├── docs/ ............................ (para FASE 5+)
└── examples/ ........................ (ejemplos futuros)
```

---

## Checklist Final

Antes de "publicar", verificar:

- ✅ Todos los archivos .md están en la raíz
- ✅ Código Python en `anonimizador/`
- ✅ `__init__.py` en todos los paquetes
- ✅ requirements.txt correcto
- ✅ .gitignore actualizado
- ✅ README.md visible en GitHub
- ✅ CHANGELOG.md actualizado
- ✅ Git history limpio (sin commits accidentales)
- ✅ Licencia MIT agregada
- ✅ Topics agregados
- ✅ Descripción completa en GitHub

---

## Próximas Acciones Post-Lanzamiento

1. **Día 1-2:** Anunciar en redes/comunidades
2. **Día 3-7:** Recopilar feedback
3. **Semana 2:** Agregar FASE 4 (multimedia + CLI + tests)
4. **Semana 3+:** Iterar según feedback

---

## Soporte y Preguntas

Si necesitas ayuda con Git:

```bash
# Ver estado actual
git status

# Ver commits
git log --oneline

# Deshacer cambio no commiteado
git checkout -- archivo.py

# Deshacer último commit (pero guardar cambios)
git reset --soft HEAD~1
```

---

**¡Listo para GitHub! 🚀**

**Última actualización:** 2025-11-14  
**Versión:** v0.1-alpha-ready
