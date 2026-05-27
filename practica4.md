# Práctica 4 — Despliegue de Streamlit en Docker con el gestor UV

**Materia:** Computación en la Nube (361)  
**Docente:** GUILLERMO ALEJANDRO CHAVEZ SANCHEZ  
**Integrantes:** Keyna Vianney · Alejandra Buelna Méndez  
**Institución:** Universidad Autónoma de Baja California  
**Fecha:** Mayo 2026

---

## Objetivo

Configurar un entorno de desarrollo dentro de un contenedor Docker basado en Ubuntu, instalar el gestor moderno de paquetes **UV** y ejecutar una aplicación web interactiva con **Streamlit**, resolviendo los problemas de mapeo de puertos que pueden surgir en el proceso.

---

## Entorno de trabajo

| Componente | Detalle |
|-----------|---------|
| Contenedor | Ubuntu sobre Docker Desktop |
| Lenguaje | Python 3.13.3 |
| Framework web | Streamlit |
| Gestor de paquetes | UV 0.10.9 |
| Control de versiones | Git |
| Puerto de la app | 8501 |

---

## Procedimiento

### 1. Preparación del sistema base

Lo primero fue actualizar el índice de paquetes del sistema y aplicar las actualizaciones disponibles:

```bash
apt update
apt upgrade -y
```

`apt update` sincroniza la lista de paquetes con los repositorios configurados. `apt upgrade -y` actualiza los paquetes instalados sin pedir confirmación manual en cada paso.

---

### 2. Instalación de dependencias esenciales

Con el sistema actualizado, se instalaron las herramientas necesarias para trabajar con Python y repositorios de código:

```bash
apt install -y git curl python3 python3-venv python3-pip
```

Cada herramienta cumple un rol específico:

- **git** — permite clonar proyectos desde GitHub
- **curl** — descarga recursos desde internet mediante línea de comandos
- **python3** — intérprete para ejecutar código Python
- **python3-venv** — creación de entornos virtuales aislados
- **python3-pip** — instalación de paquetes de Python

La versión instalada se verificó con:

```bash
python3 --version
# Python 3.13.3
```

---

### 3. Instalación de UV

UV es un gestor de dependencias y entornos virtuales escrito en Rust, mucho más rápido que pip + venv. Se instaló con:

```bash
curl -Ls https://astral.sh/uv/install.sh | sh
```

Después se actualizó la configuración de la terminal para que el sistema reconozca el comando `uv`:

```bash
source $HOME/.bashrc
uv --version
# uv 0.10.9
```

UV se instala en `/root/.local/bin`.

---

### 4. Clonación del proyecto de demostración

Se descargó un proyecto de ejemplo oficial de Streamlit desde GitHub:

```bash
git clone https://github.com/streamlit/demo-seattle-weather.git
cd demo-seattle-weather
```

Esto crea la carpeta `demo-seattle-weather` con todos los archivos del proyecto listos para usar.

---

### 5. Entorno virtual con UV

Se creó un entorno virtual para aislar las dependencias del proyecto:

```bash
uv venv
source .venv/bin/activate
```

Al activar el entorno, el nombre del proyecto aparece al inicio del prompt de la terminal, indicando que el entorno está activo. Luego se instalaron todas las dependencias declaradas en el proyecto:

```bash
uv sync
```

`uv sync` lee los archivos de configuración del proyecto e instala automáticamente Streamlit, pandas, numpy y demás librerías requeridas.

---

### 6. Ejecución de la aplicación

```bash
streamlit run streamlit_app.py --server.address 0.0.0.0 --server.port 8501
```

| Parámetro | Función |
|----------|---------|
| `streamlit run streamlit_app.py` | Inicia el servidor con el archivo principal |
| `--server.address 0.0.0.0` | Escucha en todas las interfaces de red del contenedor |
| `--server.port 8501` | Define el puerto de acceso |

La aplicación quedó disponible en: **http://localhost:8501**

---

## Problema detectado y solución

### Conflicto de puertos en Docker

El contenedor fue creado a partir de la imagen `ubuntu/nginx`, orientada principalmente al servidor web Nginx. Al configurar el mapeo de puertos como `8501:80`, se generó un conflicto porque Streamlit opera en el puerto 8501 del contenedor, no en el 80.

**Corrección:** Se eliminó el contenedor y se creó uno nuevo con el mapeo correcto:

```bash
-p 8501:8501
```

El formato correcto en Docker es siempre `puerto_host:puerto_contenedor`. Al alinear ambos puertos, la aplicación respondió correctamente desde el navegador.

---

## Aprendizajes

Esta práctica abarcó un flujo completo de trabajo: preparación del sistema operativo, gestión de dependencias con una herramienta moderna, trabajo con repositorios remotos, aislamiento de entornos y despliegue de una aplicación web. El error de puertos, aunque simple, ilustra bien por qué es importante entender el funcionamiento de Docker antes de elegir imágenes base: no toda imagen es adecuada para cualquier tipo de aplicación.

UV demostró ser significativamente más rápido que pip tradicional, lo que se nota especialmente al instalar proyectos con muchas dependencias.
