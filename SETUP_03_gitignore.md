# .gitignore - Archivos a Ignorar en Git

## Directorios y archivos Python

```
# Caché de Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST
```

## Entornos virtuales

```
# virtualenv
venv/
env/
ENV/
env.bak/
venv.bak/
.venv
.env
```

## IDEs y editores

```
# PyCharm
.idea/
*.iml

# VS Code
.vscode/
*.code-workspace

# Sublime Text
*.sublime-project
*.sublime-workspace

# Vim
*.swp
*.swo
*~
.DS_Store
```

## Archivos de proyecto específicos

```
# Logs
logs/
*.log

# Base de datos de auditoría
*.db
*.sqlite
*.sqlite3
logs/auditoria.db

# Carpetas de casos procesados (locales)
casos_exportados/
casos_clinicos_anonimizados/

# Archivos de configuración local
config/local_rules.yaml
.secrets
*.key

# Datos de prueba locales
test_data/
*.hc
*.txt
*.csv
```

## Sistema operativo

```
# macOS
.DS_Store
.AppleDouble
.LSOverride

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# Linux
.directory
```

## Herramientas de análisis

```
# Coverage
.coverage
.coverage.*
htmlcov/
.pytest_cache/

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pylint
.pylintrc
```

## Reportes y salidas

```
# Reportes generados
reports/
*.html
*.pdf
*.docx

# Auditorías exportadas
auditoria_*.json
```

## Archivos temporales

```
# Archivos temporales
*.tmp
*.bak
*.swp
*.orig
```

## Archivos de desarrollo local (IMPORTANTE: Nunca commitear datos de prueba)

```
# Datos clínicos de prueba (NUNCA commitear datos reales)
test_hc_*.txt
sample_hc_*.csv
ejemplo_historia_*.pdf

# Claves y tokens locales
.env.local
secrets.yaml
```

---

## Cómo usar este archivo

1. Copiar este contenido a archivo `.gitignore` en raíz del proyecto
2. Git automáticamente ignorará estos archivos/carpetas
3. Para verificar qué está siendo ignorado:
   ```bash
   git check-ignore -v *
   ```

4. Si necesitas agregar un archivo ignorado:
   ```bash
   git add -f archivo_especifico
   ```

## Excepciones Importantes

Si alguna carpeta/archivo debe ser commiteado pero está en .gitignore:

```bash
# Por ejemplo, si quieres un logs/ ejemplo:
echo "logs/example.log" >> .gitignore  # Ignorar logs/
git add -f logs/README.md             # Pero agregar README de explicación
```
