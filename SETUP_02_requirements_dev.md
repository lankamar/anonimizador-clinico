# requirements-dev.txt - Dependencias de Desarrollo

## Dependencias base (incluye requirements.txt)

```
-r requirements.txt
```

## Herramientas de Testing

```
pytest>=7.0
pytest-cov>=4.0
pytest-xdist>=2.5
```

## Herramientas de Calidad de Código

```
pylint>=2.15
black>=22.0
flake8>=4.0
isort>=5.10
mypy>=0.990
```

## Herramientas de Documentación

```
sphinx>=4.5
sphinx-rtd-theme>=1.0
```

## Instalación

```bash
# Instalar entorno completo de desarrollo
pip install -r requirements-dev.txt

# O crear entorno virtual limpio
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate
pip install -r requirements-dev.txt
```

## Flujo de Desarrollo Recomendado

### 1. Formateo y Linting

```bash
# Formatear código automáticamente
black anonimizador/ tests/

# Verificar estilo
pylint anonimizador/

# Análisis estático
flake8 anonimizador/ tests/

# Type checking
mypy anonimizador/
```

### 2. Testing

```bash
# Ejecutar tests con cobertura
pytest tests/ -v --cov=anonimizador --cov-report=html

# Ejecutar en paralelo (más rápido)
pytest tests/ -n auto
```

### 3. Documentación

```bash
# Generar documentación Sphinx
cd docs/
make html
```

## Pre-commit Hooks (Opcional)

Instalar `pre-commit` para ejecutar verificaciones antes de cada commit:

```bash
pip install pre-commit
pre-commit install
```

Crear archivo `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 22.0.0
    hooks:
      - id: black
  - repo: https://github.com/PyCQA/pylint
    rev: pylint-2.15.0
    hooks:
      - id: pylint
  - repo: https://github.com/PyCQA/flake8
    rev: 4.0.1
    hooks:
      - id: flake8
```

## Notas

- Todas las herramientas son opcionales pero **muy recomendadas**
- Los tests son **obligatorios** para PRs
- La cobertura de tests debe ser ≥80%
