# Práctica 3 — Carga de Datos CSV en MySQL con DBeaver

**Materia:** Computación en la Nube (361)  
**Docente:** GUILLERMO ALEJANDRO CHAVEZ SANCHEZ  
**Alumna:** Alejandra Buelna Méndez  
**Institución:** Universidad Autónoma de Baja California  
**Fecha:** Mayo 2026

---

## ¿Qué se hizo en esta práctica?

Se realizó la carga de un archivo CSV con datos del **DENUE** (Directorio Estadístico Nacional de Unidades Económicas, INEGI) hacia una base de datos MySQL, usando DBeaver como cliente gráfico. El propósito fue aprender a estructurar tablas a partir de datos reales y manejar los errores más frecuentes en este tipo de proceso.

---

## Software utilizado

| Herramienta | Función en la práctica |
|-------------|------------------------|
| DBeaver | Gestión visual de la base de datos |
| MySQL | Motor de base de datos relacional |
| Archivo CSV (`denue.csv`) | Fuente de datos del INEGI |
| GitHub | Repositorio donde se aloja el CSV |

---

## Desarrollo paso a paso

### Paso 1 — Conexión a la base de datos

Antes de cualquier operación, se abre DBeaver y se establece la conexión al servidor MySQL. Se selecciona la base de datos de destino desde el panel lateral.

---

### Paso 2 — Asistente de importación

Con la base de datos seleccionada, se accede al asistente de importación:

1. Clic derecho sobre la base de datos
2. Opción **Import Data**
3. Formato: **CSV → Import from CSV file(s)**
4. Clic en **Next**

---

### Paso 3 — Selección del archivo

Se localiza el archivo `denue.csv` desde el explorador del sistema y se añade al asistente con el botón **Add File**, seguido de **Next**.

---

### Paso 4 — Definición de la tabla

Se eligió la opción **Create new table** con el nombre `denue`. DBeaver detecta automáticamente los nombres de columna desde el encabezado del CSV. Se revisa que la detección sea correcta antes de continuar.

---

### Paso 5 — Finalización e importación

En la sección **Data load settings** se deja la configuración predeterminada y se presiona **Finish** para iniciar la carga de datos.

---

### Paso 6 — Verificación

Una vez terminada la importación, se ejecuta la siguiente consulta para confirmar que los registros se cargaron correctamente:

```sql
SELECT * FROM denue;
```

---

## Errores encontrados y cómo se resolvieron

### Error 1 — Datos demasiado largos para la columna

```
SQL Error [1406]: Data too long for column 'nom_estab'
```

**Causa:** El campo `nom_estab` fue creado con `VARCHAR(50)` pero algunos registros del CSV superan ese límite.

**Solución:** Ampliar el tamaño del campo con:

```sql
ALTER TABLE denue
MODIFY nom_estab VARCHAR(255);
```

---

### Error 2 — Fallo en inserción por lote

```
Error occurred during batch insert
```

**Causa:** Alguna fila del CSV contiene un valor incompatible con el tipo de dato definido en la columna correspondiente.

**Solución:** Presionar **Skip** en la fila con error o corregir el valor directamente en el CSV antes de reimportar.

---

### Error 3 — Tabla inexistente

```
Table 'database.tabla' doesn't exist
```

**Solución:** Crear la tabla manualmente antes de importar:

```sql
CREATE TABLE denue (
    id            INT,
    nom_estab     VARCHAR(255),
    nombre_act    VARCHAR(255)
);
```

---

### Error 4 — Formato de fecha incorrecto

El CSV puede traer fechas en formato `DD/MM/YYYY` mientras que MySQL requiere `YYYY-MM-DD`. Es necesario ajustar el formato en el archivo fuente o usar una columna de tipo `VARCHAR` para esas fechas si no se necesita operar sobre ellas.

---

## Reflexión personal

Esta práctica evidenció que importar datos no es solo cuestión de cargar un archivo: la calidad y consistencia de los datos originales determina si la importación será exitosa o no. Cada error resuelto durante el proceso refuerza la comprensión de cómo trabaja internamente MySQL con los tipos de dato y los tamaños de columna.

DBeaver resultó ser una herramienta muy útil al ofrecer un asistente visual que simplifica los pasos técnicos, aunque siguen siendo necesarios conocimientos de SQL para corregir los problemas que se presentan.

---

## Repositorio

El archivo CSV utilizado está disponible en GitHub:  
[denue.csv — archivo fuente](https://github.com/user-attachments/files/25883397/denue.csv)
