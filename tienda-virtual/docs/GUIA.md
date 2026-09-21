# Taller 1 — Tienda Virtual con Flask

## Objetivo

Construir una tienda virtual sencilla usando **Flask**, donde el catálogo de productos se carga desde un archivo **JSON** (sin base de datos). Al terminar el taller sabrás:

- Preparar un entorno de desarrollo Python aislado con `venv`.
- Organizar un proyecto Flask siguiendo una estructura de carpetas estándar.
- Servir datos desde un archivo JSON.
- Crear rutas dinámicas y plantillas con Jinja2.

## Requisitos previos

- Ubuntu (o WSL2 sobre Windows) con acceso a una terminal.
- Python 3.10 o superior instalado. Verifícalo con:

  ```bash
  python3 --version
  ```

- Conocimientos básicos de Python (funciones, listas, diccionarios) y HTML.

> todos los comandos de esta guía se ejecutan dentro de tu distribución de Linux Ubuntu.
> Windows. Abre tu terminal de Ubuntu/WSL2 antes de continuar.

## Lista de tareas del taller

Esta es la hoja de ruta que seguiremos (ligeramente ampliada respecto al
plan original, agregando pasos que son buena práctica y que necesitarás
para poder ejecutar y probar tu proyecto):

---

## Paso 1 — Crear el directorio del proyecto

```bash
mkdir tienda-virtual
cd tienda-virtual
```

A partir de aquí, **todos los comandos se ejecutan dentro de `tienda-virtual/`**.

---

## Paso 2 — Entorno virtual e instalación de Flask

Un entorno virtual evita que las librerías de este proyecto se mezclen con
las de otros proyectos o con las del sistema.

```bash
# Crear el entorno virtual (se creará una carpeta venv/)
python3 -m venv venv

# Activarlo (Linux/WSL2)
source venv/bin/activate
```

Cuando el entorno está activo, verás el prefijo `(venv)` al inicio de tu
línea de comandos. A partir de ahí, instala Flask:

```bash
pip install flask
```

Verifica la instalación:

```bash
python3 -c "import flask; print(flask.__version__)"
```

> Recuerda: cada vez que abras una nueva terminal para trabajar en este proyecto, debes volver a activar el entorno con `source venv/bin/activate`. Para salir del entorno usa `deactivate`.

---

## Paso 3 — Preparar el proyecto para Git

Crea un repositorio local con `git init` o clona un repositorio remoto con `git clone ...`, agrega el archivo `.gitignore` si es necesario.

---

## Paso 4 — Estructura de directorios

Vamos a usar una estructura típica de un proyecto Flask pequeño, basada en el patrón *application factory* (una función que construye la app). Esto facilita crecer el proyecto más adelante (agregar pruebas, blueprints, configuración por ambiente, etc.).

Crea la siguiente estructura (puedes usar `mkdir -p` o crear las carpetas desde tu editor):

```
tienda-virtual/
├── app/
│   ├── __init__.py          # Application factory (create_app)
│   ├── routes.py            # Rutas / vistas de la tienda
│   ├── data/
│   │   └── productos.json   # Catálogo de productos
│   ├── static/
│   │   ├── css/
│   │   │   └── style.css
│   │   └── images/          # Fotos de los productos
│   └── templates/
│       ├── base.html
│       ├── index.html
│       └── detalle.html
├── venv/                     # Entorno virtual (no se versiona)
├── requirements.txt
├── run.py                    # Punto de entrada de la aplicación
├── .gitignore
└── docs/
    └── GUIA.md               # Este documento
```

Comando rápido para crear las carpetas necesarias:

```bash
mkdir -p app/data app/static/css app/static/images app/templates docs
```

---

## Paso 5 — Catálogo de productos

Crea el archivo `app/data/productos.json`. Cada producto debe tener al menos estos campos:

| Campo    | Tipo   | Descripción                                              |
|----------|--------|-----------------------------------------------------------|
| `sku`    | string | Identificador único del producto (lo usaremos en la URL) |
| `marca`  | string | Marca del producto                                       |
| `nombre` | string | Nombre del producto                                      |
| `precio` | number | Precio en pesos (sin puntos ni comas)                    |
| `foto`   | string | Ruta relativa dentro de `static/`, ej: `images/mouse.jpg` |

Ejemplo (ya incluido en el proyecto de partida, agrega tus propios productos siguiendo el mismo formato):

```json
[
  {
    "sku": "TEC-001",
    "marca": "Logitech",
    "nombre": "Mouse inalámbrico MX Master",
    "precio": 349900,
    "foto": "images/mouse-mx-master.jpg"
  }
]
```

Agrega al menos 10 productos distintos y coloca las imágenes correspondientes en `app/static/images/`. Si no tienes fotos reales, usa imágenes de prueba (puedes buscar "placeholder image generator").

---

## Paso 6 — Programa principal de la tienda

Usaremos tres archivos: `run.py` (punto de entrada), `app/__init__.py` (construye la aplicación) y `app/routes.py` (las rutas). Los archivos de partida ya existen en el proyecto con comentarios `# TODO` que debes completar.

### `app/__init__.py`

Debes:
- Crear una función `create_app()` que instancie `Flask(__name__)`.
- Importar el *blueprint* `main` definido en `routes.py`.
- Registrarlo con `app.register_blueprint(main)`.
- Retornar la instancia `app`.

### `app/routes.py`

Debes:
- Definir un `Blueprint` llamado `main`.
- Tener una función `cargar_productos()` que abra `productos.json` y retorne la lista de productos como diccionarios de Python (usa el módulo `json`).
- Tener una función `buscar_producto_por_sku(sku)` que recorra la lista de productos y retorne el que coincida, o `None` si no existe.
- Definir la ruta `"/"` (función `index`) que cargue los productos y renderice `index.html` enviándoselos.
- Definir la ruta `"/producto/<sku>"` (función `detalle`) que busque el producto por su SKU y:
  - Si no existe, responda con `abort(404)`.
  - Si existe, renderice `detalle.html` enviándole ese producto.

### `run.py`

Debes:
- Importar `create_app` desde `app`.
- Crear la instancia `app = create_app()`.
- Ejecutar `app.run(debug=True)` dentro del bloque `if __name__ == "__main__":`.

> El modo `debug=True` es solo para desarrollo: recarga el servidor automáticamente al guardar cambios y muestra errores detallados en el navegador. **Nunca se usa en producción.**

Revisa los archivos `app/__init__.py`, `app/routes.py` y `run.py` del proyecto de partida: cada `# TODO` indica exactamente qué línea o bloque de código debes escribir.

---

## Paso 7 — Plantillas HTML (Jinja2)

Las plantillas usan **Jinja2**, el motor de templates de Flask, que permite insertar valores de Python dentro del HTML con `{{ variable }}` y usar estructuras de control como `{% for %}` o `{% if %}`.

### `base.html`

Ya está completo en el proyecto de partida: define la estructura HTML común (head, header, hoja de estilos) y dos bloques (`titulo` y `contenido`) que las otras plantillas van a rellenar con `{% extends %}`.

### `index.html` — catálogo

Debe:
- Extender `base.html`.
- Recorrer la lista `productos` (enviada desde la ruta `index`) con un ciclo `{% for producto in productos %}`.
- Por cada producto, mostrar: imagen, nombre, marca, precio, y un enlace al detalle usando `url_for('main.detalle', sku=producto.sku)`.
- Cerrar el ciclo con `{% endfor %}`.

### `detalle.html` — ficha de producto

Debe:
- Extender `base.html`.
- Mostrar la imagen, nombre, marca, SKU y precio del producto recibido (variable `producto`).
- Incluir un enlace para volver al catálogo (`url_for('main.index')`).

Los archivos `index.html` y `detalle.html` de partida tienen comentarios `{# TODO ... #}` indicando qué debes escribir en cada bloque.

---

## Paso 8 — Congelar dependencias

Una vez tengas Flask instalado y tu proyecto funcionando, guarda las dependencias exactas del proyecto en `requirements.txt`:

```bash
pip freeze > requirements.txt
```

Esto permite que cualquier otra persona (o tú mismo en otro computador) recree el mismo entorno con:

```bash
pip install -r requirements.txt
```

---

## Paso 9 — Ejecutar y probar la aplicación

Con el entorno virtual activado y parado en la raíz del proyecto:

```bash
python3 run.py
```

Deberías ver algo como:

```
 * Running on http://127.0.0.1:5000
```

Abre esa dirección en tu navegador (si usas WSL2, puedes abrirla
directamente desde el navegador de Windows, WSL2 expone el puerto
automáticamente).

Verifica:
1. Que `/` muestre el catálogo con todos tus productos.
2. Que al hacer clic en "Ver detalle" te lleve a `/producto/<sku>` y
   muestre la información correcta.
3. Que una URL con un SKU inexistente (ej. `/producto/NO-EXISTE`) muestre
   un error 404.

---

## Checklist final

- [ ] El entorno virtual se activa sin errores y Flask está instalado.
- [ ] La estructura de carpetas coincide con la propuesta.
- [ ] `productos.json` tiene al menos 6 productos completos.
- [ ] `cargar_productos()` y `buscar_producto_por_sku()` funcionan
      correctamente.
- [ ] La ruta `/` muestra el catálogo completo.
- [ ] La ruta `/producto/<sku>` muestra el detalle correcto o un 404.
- [ ] `requirements.txt` está generado.
- [ ] La aplicación corre con `python run.py` sin errores.

## Posibles extensiones (para talleres siguientes)

- Agregar un buscador de productos por nombre o marca.
- Agregar categorías de productos.
- Agregar un carrito de compras usando la sesión de Flask (`flask.session`).
- Migrar el catálogo de JSON a una base de datos (SQLite + SQLAlchemy).
