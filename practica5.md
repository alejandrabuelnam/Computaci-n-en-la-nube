# Práctica 5 — Respaldos automáticos de MySQL con cron y Docker

**Materia:** Computación en la Nube (361)  
**Docente:** GUILLERMO ALEJANDRO CHAVEZ SANCHEZ  
**Integrantes:** Keyna Vianney · Alejandra Buelna Méndez  
**Institución:** Universidad Autónoma de Baja California  
**Fecha:** Mayo 2026

---

## ¿Qué se automatizó?

Se configuró un proceso de respaldo periódico de una base de datos MySQL que corre dentro de un contenedor Docker, usando `mysqldump` y el programador de tareas `cron` desde Ubuntu (WSL). La práctica cubre tres escenarios: respaldo continuo, respaldo en fechas específicas y conservación de un número limitado de copias.

---

## Requisitos del entorno

- Windows con **WSL (Ubuntu)** activo
- **Docker Desktop** abierto y en ejecución
- **DBeaver** instalado para verificar la base de datos
- Contenedor Docker llamado `sql` con MySQL corriendo en el puerto `3306`

---

## Configuración inicial

### Arrancar el contenedor

```bash
docker start sql
docker ps
```

El contenedor debe aparecer con `STATUS: Up` y el puerto `3306:3306` visible. Docker Desktop debe estar abierto en Windows para que los comandos funcionen desde WSL.

### Verificar el servicio cron

```bash
systemctl status cron
```

Si el estado es `inactive (dead)`:

```bash
sudo systemctl start cron
```

### Confirmar la ruta de mysqldump

```bash
which mysqldump
# /usr/bin/mysqldump
```

---

## Escenario 1 — Respaldo cada minuto (prueba continua)

Útil para verificar que el proceso funciona correctamente antes de programarlo en producción.

```bash
crontab -e
```

Se agrega la siguiente línea (sin el `#`):

```bash
* * * * * /usr/bin/mysqldump -h 127.0.0.1 -u TU_USUARIO -pTU_CONTRASEÑA servidor > /mnt/c/Users/Edgar/Documents/servidor/respaldo_$(date +\%F_\%H-\%M).sql && ls /mnt/c/Users/Edgar/Documents/servidor/
```

El `&& ls` al final muestra en terminal los archivos disponibles tras cada respaldo.

Para confirmar que se guardó la tarea:

```bash
crontab -l
```

**Resultado esperado:** cada minuto aparece un archivo nuevo en la carpeta de destino:

```
respaldo_2026-03-25_19-39.sql
respaldo_2026-03-25_19-40.sql
respaldo_2026-03-25_19-41.sql
```

---

## Escenario 2 — Respaldo en fechas y horarios exactos

La sintaxis de crontab define cuándo se ejecuta la tarea con cinco campos:

```
MIN  HORA  DÍA-MES  MES  DÍA-SEMANA
 *    *       *       *        *
```

| Campo | Rango | Descripción |
|-------|-------|-------------|
| Minuto | 0–59 | Minuto de ejecución |
| Hora | 0–23 | Hora del día |
| Día del mes | 1–31 | Día del mes |
| Mes | 1–12 | Mes del año |
| Día de semana | 0–6 | 0 = domingo, 6 = sábado |

### Ejemplos prácticos

**Todos los días a las 11 PM:**
```bash
0 23 * * * /usr/bin/mysqldump -h 127.0.0.1 -u TU_USUARIO -pTU_CONTRASEÑA servidor > /mnt/c/Users/Edgar/Documents/servidor/respaldo_$(date +\%F_\%H-\%M).sql
```

**Cada lunes a las 8 AM:**
```bash
0 8 * * 1 /usr/bin/mysqldump -h 127.0.0.1 -u TU_USUARIO -pTU_CONTRASEÑA servidor > /mnt/c/Users/Edgar/Documents/servidor/respaldo_$(date +\%F_\%H-\%M).sql
```

**El día 1 de cada mes a medianoche:**
```bash
0 0 1 * * /usr/bin/mysqldump -h 127.0.0.1 -u TU_USUARIO -pTU_CONTRASEÑA servidor > /mnt/c/Users/Edgar/Documents/servidor/respaldo_$(date +\%F_\%H-\%M).sql
```

---

## Escenario 3 — Conservar solo los últimos N respaldos

Para evitar que los archivos acumulados llenen el disco, se puede configurar la tarea para eliminar automáticamente los respaldos más antiguos:

```bash
* * * * * /usr/bin/mysqldump -h 127.0.0.1 -u TU_USUARIO -pTU_CONTRASEÑA servidor > /mnt/c/Users/Edgar/Documents/servidor/respaldo_$(date +\%F_\%H-\%M).sql && ls -t /mnt/c/Users/Edgar/Documents/servidor/respaldo_*.sql | tail -n +6 | xargs rm -f
```

### ¿Cómo funciona esta cadena de comandos?

| Segmento | Qué hace |
|---------|----------|
| `mysqldump ... > respaldo_fecha.sql` | Genera el nuevo archivo de respaldo |
| `&&` | Continúa solo si el respaldo fue exitoso |
| `ls -t ... respaldo_*.sql` | Lista los archivos del más nuevo al más antiguo |
| `tail -n +6` | Toma todos a partir del sexto (descarta los 5 más recientes) |
| `xargs rm -f` | Elimina los seleccionados sin pedir confirmación |

> Para conservar los últimos **N** respaldos, usar `tail -n +(N+1)`.  
> Ejemplo: conservar 10 → `tail -n +11`

---

## Resumen de comandos clave

```bash
# Iniciar el contenedor MySQL
docker start sql

# Verificar que está corriendo
docker ps

# Estado del servicio cron
systemctl status cron

# Iniciar cron si está inactivo
sudo systemctl start cron

# Ruta del binario mysqldump
which mysqldump

# Editar tareas programadas
crontab -e

# Ver tareas activas
crontab -l

# Ver archivos en la carpeta de respaldos
ls /mnt/c/Users/Edgar/Documents/servidor/
```

---

## Reflexión

Automatizar los respaldos con cron elimina la dependencia del factor humano: el sistema hace las copias aunque nadie esté presente. El tercer escenario (límite de copias) es especialmente relevante en entornos reales donde el almacenamiento es costoso o limitado. Combinar Docker, MySQL y WSL en un solo flujo de trabajo también refuerza la comprensión de cómo interactúan estas capas de infraestructura.
