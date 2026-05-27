# Computación en la Nube (361) — Prácticas

**Estudiante:** Alejandra Buelna Méndez  
**Docente:** GUILLERMO ALEJANDRO CHAVEZ SANCHEZ  
**Institución:** Universidad Autónoma de Baja California  
**Semestre:** 2026

---

## Contenido del repositorio

| Archivo | Tema |
|---------|------|
| `practica3.md` | Carga de datos CSV a MySQL con DBeaver |
| `practica4.md` | Despliegue de Streamlit en Docker con UV |
| `practica5.md` | Respaldos automáticos de MySQL con cron y Docker |
| `practica6.md` | Normalización del DENUE en Excel |

---

## Descripción de las prácticas

### Práctica 3 — Importación CSV con DBeaver
Carga del archivo DENUE (INEGI) a una base de datos MySQL usando el asistente gráfico de DBeaver. Se documentan los errores más frecuentes (longitud de columna, formato de fechas, tablas inexistentes) y sus soluciones.

### Práctica 4 — Streamlit + Docker + UV
Configuración de un entorno Python dentro de un contenedor Ubuntu, instalación del gestor moderno UV y ejecución de una aplicación Streamlit. Incluye la solución al conflicto de mapeo de puertos.

### Práctica 5 — Respaldos automáticos con cron
Automatización de `mysqldump` mediante el programador de tareas cron en WSL. Cubre tres escenarios: respaldo continuo, programación por fecha/hora exacta y conservación de un número fijo de copias.

### Práctica 6 — Normalización DENUE en Excel
Diseño de un modelo relacional en Excel aplicando 2FN y 3FN al dataset del DENUE. Incluye diccionario de datos, catálogos geográficos, tablas de análisis y justificación técnica de cada decisión.

---

## Tecnologías utilizadas

- MySQL · DBeaver
- Docker Desktop · Ubuntu (WSL)
- Python 3 · Streamlit · UV
- Microsoft Excel
- Git / GitHub
