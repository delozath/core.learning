# Syllabus

Aplicación CLI para generar syllabus de cursos a partir de configuración YAML administrada con Hydra. El flujo actual está orientado a la UAM-I y soporta dos fuentes de datos para el temario: SQLite y CSV. La salida principal es un archivo `.tex` listo para compilar.

El proyecto fue migrado a una arquitectura más modular con puertos y adaptadores para separar:

- configuración del curso e institución,
- acceso a datos,
- orquestación de tareas,
- publicación del documento final.

## Estado actual

Implementado:

- Generación de syllabus en LaTeX desde CLI.
- Resolución de configuración por institución y curso con Hydra.
- Lectura de temario desde `sqlite` o `csv`.
- Render de secciones complejas con Jinja2.
- Personalización del encabezado del documento mediante configuración YAML.

En migración:

- Cálculo y exportación de calificaciones.

## Requisitos

- Python `>=3.10`
- `pip`

Dependencias actualmente usadas por la app:

- `hydra-core`
- `omegaconf`
- `pandas`
- `pylatex`
- `jinja2`
- `python-dotenv`

## Instalación

Clona el repositorio e instala la app en modo editable desde `apps/syllabus`:

```bash
git clone https://github.com/delozath/core.learning.git
cd core.learning.public/apps/syllabus
pip install -e .
pip install pandas pylatex jinja2 python-dotenv
```

## Variables de entorno

Crea un archivo `.env` en `apps/syllabus/` con las rutas de trabajo:

```env
DB_PATH=/ruta/a/tu/carpeta/de/datos
TEX_FILES_PATH=/ruta/a/tu/carpeta/de/salida_tex
ROOT=/ruta/a/core.learning
```

Notas:

- `DB_PATH` debe apuntar al directorio que contiene `cursos.db`, `temario.csv` y `notas.csv`.
- `TEX_FILES_PATH` es el directorio donde se escribe el archivo `.tex` generado.
- `ROOT` se mantiene como raíz del proyecto para configuraciones auxiliares.

## Estructura relevante

```text
apps/syllabus/
├── files/
├── pyproject.toml
├── README.md
└── src/syllabus/
    ├── conf/
    │   ├── config.yaml
    │   ├── curso/
    │   │   └── symp.yaml
    │   └── institution/
    │       └── uam.yaml
    ├── core.py
    ├── data/
    ├── ports/
    ├── tasks/uam/
    │   ├── _syllabus.py
    │   ├── jinja_driver.py
    │   ├── templates/
    │   └── trimestre.py
    └── main.py
```

## Uso

Ejemplo con fuente CSV:

```bash
python -m syllabus.main institution=uam curso=symp task=syllabus +db_type=csv
```

Ejemplo con fuente SQLite:

```bash
python -m syllabus.main institution=uam curso=symp task=syllabus +db_type=sqlite
```

La resolución de Hydra usa:

- `institution=uam` para cargar `src/syllabus/conf/institution/uam.yaml`
- `curso=symp` para cargar `src/syllabus/conf/curso/symp.yaml`
- `+db_type={csv|sqlite}` para seleccionar el adaptador de datos

La salida se publica como:

```text
${TEX_FILES_PATH}/Syllabus-symp-26-P.tex
```

## Configuración actual del curso `symp`

La versión actual del curso está parametrizada para el trimestre `26-P`:

- UEA: `Secuenciadores y Microprocesadores`
- Clave: `2151024`
- Grupo: `CH51`
- Inicio: `2026-05-06`
- Fin: `2026-07-29`

Además, el bloque `calendario` define:

- sesiones de teoría y laboratorio por día,
- feriados,
- fechas de examen.

## Personalización

La configuración global en `src/syllabus/conf/config.yaml` ahora permite ajustar sin tocar código:

- `docente`
- `email`
- `cubiculo`
- `latex.extra_packages`

Esto alimenta directamente la sección de datos del profesor y la carga de paquetes LaTeX del documento generado.

## Control de cambios

Actualización trimestre `26-P`

- Se automatizó la parametrización de `docente`, `email` y `cubiculo` desde `config.yaml`.
- Se agregó una lista en `config.yaml` para personalizar la carga de paquetes LaTeX mediante `latex.extra_packages`.
- Actualizaron las fechas, sesiones y eventos del calendario para ajustarse al trimestre `26-P`.
- Se migró a un nuevo repositorio para mejorar la organización del proyecto.
