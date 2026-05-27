# Práctica 6 — Normalización del DENUE en Excel

**Materia:** Computación en la Nube (361)  
**Docente:** GUILLERMO ALEJANDRO CHAVEZ SANCHEZ  
**Herramienta:** Microsoft Excel  
**Fuente:** INEGI — DENUE (Directorio Estadístico Nacional de Unidades Económicas)  
**Alumna:** Alejandra Buelna Méndez  
**Institución:** Universidad Autónoma de Baja California  
**Fecha:** Mayo 2026

---

## ¿Por qué normalizar?

El archivo original del DENUE tiene 26 columnas con alta redundancia: la misma información geográfica (entidad, municipio, localidad) se repite en cada fila. Los campos de vialidad están fragmentados en múltiples columnas (`tipo_vial`, `nom_vial`, `tipo_v_e_1`, etc.) y los nombres de actividad económica se duplican en cientos de registros.

Aplicar **normalización relacional** resuelve estos problemas: cada dato vive en un único lugar y se referencia desde donde se necesite.

---

## Estructura del archivo normalizado

El archivo `DENUE_Normalizado.xlsx` organiza la información en **8 hojas**:

| Hoja | Tipo | Contenido |
|------|------|-----------|
| `Establecimientos` | Tabla de hechos | 678 registros · 11 columnas (reducidas de 26) |
| `CAT_Actividades` | Catálogo | Actividades SCIAN con sector de 2 dígitos |
| `CAT_Entidades` | Catálogo | Entidades federativas |
| `CAT_Municipios` | Catálogo | Municipios con referencia a entidad |
| `CAT_Localidades` | Catálogo | Localidades con jerarquía completa |
| `CAT_PersonalOcupado` | Catálogo | Rangos de empleados con estimado numérico y clasificación |
| `Tablas_Especificas` | Análisis | KPIs pre-calculados por municipio, actividad y tamaño |
| `Justificacion` | Documentación | Descripción técnica de cada decisión tomada |

---

## Diccionario de datos

### Tabla `Establecimientos`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `ID` | Numérico | Identificador único del registro |
| `Nombre Establecimiento` | Texto | Nombre comercial del negocio |
| `Razón Social` | Texto | Nombre legal (puede estar vacío) |
| `Clave Actividad` | Numérico | FK → `CAT_Actividades` |
| `Rango Personal Ocupado` | Texto | FK → `CAT_PersonalOcupado` |
| `Dirección` | Texto | Dirección consolidada en un solo campo |
| `Código Postal` | Numérico | CP del establecimiento |
| `ID Localidad` | Numérico | FK → `CAT_Localidades` |
| `ID Municipio` | Numérico | FK → `CAT_Municipios` |
| `ID Entidad` | Numérico | FK → `CAT_Entidades` |

### Catálogo `CAT_Actividades`

| Campo | Descripción |
|-------|-------------|
| `Código Actividad` | Código SCIAN de 6 dígitos |
| `Nombre Actividad` | Descripción de la actividad económica |
| `Sector (2 dígitos)` | Agrupación macro del sector productivo |

### Catálogo `CAT_Entidades`

| Campo | Descripción |
|-------|-------------|
| `Clave Entidad` | Clave numérica |
| `Nombre Entidad` | Nombre de la entidad federativa |

### Catálogo `CAT_Municipios`

| Campo | Descripción |
|-------|-------------|
| `Clave Municipio` | Clave numérica del municipio |
| `Nombre Municipio` | Nombre del municipio |
| `Clave Entidad` | FK → `CAT_Entidades` |

### Catálogo `CAT_Localidades`

| Campo | Descripción |
|-------|-------------|
| `Clave Localidad` | Clave numérica |
| `Nombre Localidad` | Nombre de la localidad |
| `Clave Municipio` | FK → `CAT_Municipios` |
| `Clave Entidad` | FK → `CAT_Entidades` |

### Catálogo `CAT_PersonalOcupado`

| Campo | Descripción |
|-------|-------------|
| `Rango` | Categoría textual (ej. "0 a 5 personas") |
| `Estimado Personas` | Valor numérico representativo del rango |
| `Clasificación Tamaño` | Micro / Pequeña / Mediana / Grande |

---

## Tablas de análisis (`Tablas_Especificas`)

### A — Concentración geográfica
Muestra cuántos establecimientos hay por municipio, su participación porcentual y el personal estimado total. Permite identificar qué zonas concentran la mayor actividad económica.

### B — Actividades económicas dominantes
Agrupa los establecimientos por código SCIAN para identificar qué sectores tienen mayor presencia en el universo analizado.

### C — Estructura del tejido empresarial
Clasifica los establecimientos por tamaño (Micro / Pequeña / Mediana / Grande) y muestra el peso de cada categoría dentro del total.

---

## Decisiones de normalización y su justificación

| # | Elemento | Acción | Razón técnica |
|---|----------|--------|---------------|
| 1 | Tabla principal | Reducción de 26 a 11 columnas; dirección en campo único | Eliminación de redundancia en campos de vialidad fragmentados |
| 2 | `CAT_Actividades` | Catálogo separado con sector | **3FN**: el nombre de actividad depende solo del código SCIAN, no del establecimiento |
| 3 | `CAT_Entidades` | Tabla independiente | 678 repeticiones del nombre de entidad se reducen a 2 registros únicos |
| 4 | `CAT_Municipios` | Catálogo con FK a entidad | El municipio depende de (`cve_ent`, `cve_mun`); separarlo cumple la **2FN** |
| 5 | `CAT_Localidades` | Jerarquía completa entidad → municipio → localidad | Elimina dependencias transitivas de la tabla principal (**3FN**) |
| 6 | `CAT_PersonalOcupado` | Catálogo con estimado numérico y clasificación | Agrega valor analítico al campo categórico original; estandariza criterios INEGI/SE |
| 7 | `Tablas_Especificas` | KPIs pre-calculados | El analista obtiene métricas clave sin manipular el dato crudo |
| 8 | Valores nulos | Se conservan como vacío | `raz_social`, `numero_int` y `cod_postal` ausentes son nulos **válidos** en el DENUE |

---

## Principios aplicados en el diseño

La normalización siguió un enfoque orientado al analista con cuatro objetivos:

**Reducción de ruido** — pasar de 26 a 11 campos elimina columnas redundantes y facilita la lectura del modelo.

**Reutilización sin repetición** — actualizar un nombre de actividad o municipio requiere cambiar solo el catálogo correspondiente, no cientos de filas.

**Jerarquía geográfica explícita** — la cadena Entidad → Municipio → Localidad se representa mediante llaves foráneas navegables.

**Integridad de nulos** — los campos vacíos del DENUE no se imputan con valores inventados; se documentan como ausencias legítimas.

---

## Repositorio

```
DENUE-Normalizacion/
├── README.md                   ← Documentación del proyecto
└── DENUE_Normalizado.xlsx      ← Archivo con la normalización completa
```

---

## Fuentes

- INEGI — DENUE: https://www.inegi.org.mx/app/mapa/denue/  
- Sistema SCIAN: https://www.inegi.org.mx/app/scian/
