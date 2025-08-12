# **Manual de manejo de bases de datos – R (Caso ficticio “AsegurAct”)**
*Guía rápida para lectura, exploración, limpieza y manipulación de datos en R para entrevistas técnicas.*

**Contexto del caso (ficticio, consistente en todo el manual):**
- `df_clientes` (csv):  
  `id_cliente` (int), `nombre` (chr), `email` (chr), `ciudad` (chr), `edad` (int),  
  `fecha_alta` (date, alta de la póliza), `segmento` (chr: "Particular"/"PYME"/"Corporativo"),  
  `categoria_riesgo` (chr: "Bajo"/"Medio"/"Alto"), `prima_mensual` (dbl), `activo` (int: 1/0).
- `df_pagos` (excel, hoja "Pagos"):  
  `id_cliente` (int), `fecha_pago` (date), `valor_pago` (dbl), `medio` (chr: "PSE"/"Tarjeta"/"Efectivo").
- `df_siniestros` (sqlite, tabla "siniestros"):  
  `id_cliente` (int), `fecha_siniestro` (date), `valor_siniestro` (dbl), `tipo` (chr: "Médico"/"Odont"/"Otro").

---

## **1. Lectura de archivos**

### 1.1 CSV — ruta directa (clientes)
```r
library(readr)                              # Lectura rápida de archivos de texto (CSV/TSV)
df_clientes <- read_csv("clientes.csv")     # Lee 'clientes.csv' de la carpeta de trabajo
head(df_clientes)                           # Verifica primeras filas y nombres reales
````

### 1.2 CSV — selección manual (clientes)

```r
library(readr)
ruta_cli <- file.choose()                   # Abre diálogo para elegir 'clientes.csv'
df_clientes <- read_csv(ruta_cli)           # Lee el CSV elegido
head(df_clientes)
```

> Si el CSV usa `;`:

```r
df_clientes <- read_delim("clientes.csv", delim = ";")  # Equivalente cuando separador es punto y coma
```

### 1.3 Excel — ruta directa (pagos, hoja específica)

```r
library(readxl)                              # Lectura de Excel sin Java
df_pagos <- read_excel("pagos.xlsx", sheet = "Pagos")   # Lee SOLO la hoja "Pagos"
head(df_pagos)
```

### 1.4 Excel — selección manual (pagos)

```r
library(readxl)
ruta_xlsx <- file.choose()                               # Seleccionar 'pagos.xlsx'
excel_sheets(ruta_xlsx)                                  # (Opcional) ver nombres de hojas
df_pagos <- read_excel(ruta_xlsx, sheet = "Pagos")       # Carga por nombre de pestaña
# df_pagos <- read_excel(ruta_xlsx, sheet = 1)           # Alternativa: por índice
head(df_pagos)
```

### 1.5 SQL (SQLite) — ruta directa (siniestros)

```r
library(DBI)                                            # Interfaz DB
library(RSQLite)                                        # Driver SQLite

con <- dbConnect(SQLite(), "aseguract.db")             # Conecta a archivo .db
df_siniestros <- dbGetQuery(con, "SELECT * FROM siniestros")  # Trae toda la tabla
# dbDisconnect(con)                                    # Cerrar conexión (buena práctica)
head(df_siniestros)
```

### 1.6 SQL — selección manual (siniestros)

```r
library(DBI); library(RSQLite)
ruta_db <- file.choose()                                 # Seleccionar 'aseguract.db'
con <- dbConnect(SQLite(), ruta_db)
df_siniestros <- dbGetQuery(con, "SELECT * FROM siniestros")
head(df_siniestros)
```

---

## **2. Exploración de datos (clientes como ejemplo)**

```r
dim(df_clientes)                    # Dimensiones (filas, columnas)
colnames(df_clientes)               # Nombres de columnas reales
str(df_clientes)                    # Tipos detectados (edad numérica, fechas, etc.)
summary(df_clientes)                # Resumen estadístico (promedios, NA, etc.)
```

Descripción rápida:

* Esperamos `id_cliente` int, `edad` int, `prima_mensual` dbl, `activo` 0/1, `fecha_alta` en formato fecha.

---

## **3. Selección y filtrado (clientes)**

```r
df_clientes$ciudad                                  # Accede a la columna 'ciudad' como vector
df_clientes[, c("id_cliente", "nombre", "email")]   # Selecciona columnas clave por nombre

# Filtrar clientes mayores a 30 años y residentes en Bogotá
df_bogota_30 <- subset(df_clientes, edad > 30 & ciudad == "Bogotá")
head(df_bogota_30)

# Seleccionar solo columnas de contacto de ese filtro
df_contacto_bogota <- df_bogota_30[, c("id_cliente", "nombre", "email")]
head(df_contacto_bogota)
```

---

## **4. Ordenamiento (clientes)**

```r
# Top 10 clientes por prima mensual (descendente)
top10_prima <- df_clientes[order(-df_clientes$prima_mensual), ][1:10, ]
head(top10_prima)

# Orden por múltiples columnas: primero por 'segmento' asc, luego 'prima_mensual' desc
orden_multi <- df_clientes[order(df_clientes$segmento, -df_clientes$prima_mensual), ]
head(orden_multi)
```

---

## **5. Agrupación y resumen (clientes)**

```r
library(dplyr)

# Prima promedio por ciudad
prima_ciudad <- df_clientes %>%
  group_by(ciudad) %>%
  summarise(
    n_clientes = n(),                          # Conteo por grupo
    prima_prom = mean(prima_mensual, na.rm=TRUE)
  )

# Prima y edad promedio por categoría de riesgo
res_riesgo <- df_clientes %>%
  group_by(categoria_riesgo) %>%
  summarise(
    n = n(),
    prima_prom = mean(prima_mensual, na.rm=TRUE),
    edad_prom  = mean(edad, na.rm=TRUE)
  )
```

---

## **6. Limpieza y transformación (clientes)**

```r
library(dplyr)

df_clientes <- df_clientes %>%
  rename(prima = prima_mensual) %>%            # Renombrar por brevedad
  mutate(
    activo = as.logical(activo),               # 1/0 → TRUE/FALSE
    prima_anual = prima * 12,                  # Nueva columna derivada
    segmento = factor(segmento, 
                      levels = c("Particular","PYME","Corporativo")), # Orden explícito
    categoria_riesgo = factor(categoria_riesgo,
                              levels = c("Bajo","Medio","Alto"))
  )

# Rellenos y NA
df_clientes$email[is.na(df_clientes$email)] <- "sin_email@dominio.com"  # Relleno selectivo
df_clientes <- na.omit(df_clientes)          # Elimina filas con NA en cualquier columna (úsalo con criterio)
```

---

## **7. Joins (clientes + pagos agregados)**

```r
library(dplyr)

# Agregamos pagos por cliente (total y último pago)
pagos_resumen <- df_pagos %>%
  group_by(id_cliente) %>%
  summarise(
    total_pagado = sum(valor_pago, na.rm=TRUE),
    ultimo_pago  = max(fecha_pago, na.rm=TRUE)
  )

# LEFT JOIN: conservo TODOS los clientes aunque no tengan pagos
clientes_con_pagos <- df_clientes %>%
  left_join(pagos_resumen, by = "id_cliente")

head(clientes_con_pagos)
```

---

## **8. Exportar datos (resultado final)**

```r
library(readr)
write_csv(clientes_con_pagos, "clientes_con_pagos.csv")   # Exporta CSV limpio

library(writexl)
write_xlsx(clientes_con_pagos, "clientes_con_pagos.xlsx") # Exporta Excel
```

---

## **9. Fechas y tiempos (altas y pagos)**

```r
library(lubridate); library(dplyr)

# Asegurar tipos fecha
df_clientes$fecha_alta <- as.Date(df_clientes$fecha_alta)    # o ymd(...) si viene texto
df_pagos$fecha_pago     <- as.Date(df_pagos$fecha_pago)

# Alta por año/mes
altas_mensual <- df_clientes %>%
  mutate(anio = year(fecha_alta), mes = month(fecha_alta, label=TRUE)) %>%
  count(anio, mes, name = "n_altas")

# Pagos promedio por mes
pagos_mensual <- df_pagos %>%
  mutate(anio = year(fecha_pago), mes = month(fecha_pago, label=TRUE)) %>%
  group_by(anio, mes) %>%
  summarise(pago_prom = mean(valor_pago, na.rm=TRUE))
```

---

## **10. Tipos y conversión**

```r
# Conversión explícita por si llegan como texto
df_clientes$edad          <- as.integer(df_clientes$edad)
df_clientes$prima         <- as.numeric(df_clientes$prima)
df_clientes$activo        <- as.logical(df_clientes$activo)
df_clientes$ciudad        <- as.character(df_clientes$ciudad)
df_clientes$categoria_riesgo <- factor(df_clientes$categoria_riesgo,
                                       levels = c("Bajo","Medio","Alto"))
```

---

## **11. Valores perdidos y outliers**

```r
# Conteo de NA por columna (clientes)
colSums(is.na(df_clientes))

# Eliminar registros sin 'prima' o sin 'edad'
df_clientes <- df_clientes %>% filter(!is.na(prima), !is.na(edad))

# Rango percentilar para detectar outliers en prima
quantile(df_clientes$prima, probs = c(.01,.99), na.rm=TRUE)
```

---

## **12. Pivot (ancho ↔ largo)**

```r
library(tidyr); library(dplyr)

# Pagos por medio → ancho (una columna por medio de pago)
pagos_ancho <- df_pagos %>%
  group_by(id_cliente, medio) %>%
  summarise(total = sum(valor_pago, na.rm=TRUE), .groups="drop") %>%
  pivot_wider(names_from = medio, values_from = total, values_fill = 0)

# Métricas de clientes → largo (ejemplo didáctico)
clientes_largo <- df_clientes %>%
  select(id_cliente, prima, edad) %>%
  pivot_longer(cols = c(prima, edad), names_to = "metrica", values_to = "valor")
```

---

## **13. Conteos y proporciones**

```r
library(dplyr)

# Frecuencias por categoría de riesgo
tabla_riesgo <- df_clientes %>% count(categoria_riesgo, name = "n")

# Proporciones por segmento
prop_segmento <- df_clientes %>%
  group_by(segmento) %>%
  summarise(prop = n()/nrow(df_clientes))
```

---

## **14. Duplicados y llaves**

```r
any(duplicated(df_clientes$id_cliente))     # TRUE si hay IDs repetidos
df_clientes <- df_clientes %>% distinct()   # Quita filas duplicadas exactas
```

---

## **15. Orden multi-columna y Top-N**

```r
library(dplyr)

top5_por_segmento <- df_clientes %>%
  arrange(segmento, desc(prima)) %>%
  group_by(segmento) %>%
  slice_head(n = 5)
```

---

## **16. Chequeos rápidos (QA)**

```r
stopifnot(!any(is.na(df_clientes$id_cliente)))  # Falla si hay NA en la llave
stopifnot(is.numeric(df_clientes$prima))        # Asegura tipo de 'prima'
```

---

## **17. Muestras y splits (si te piden separar conjuntos)**

```r
set.seed(123)
idx <- sample.int(nrow(df_clientes), 0.7*nrow(df_clientes))  # 70% aleatorio
train <- df_clientes[idx, ]
test  <- df_clientes[-idx, ]
```

---

## **18. Gráficos rápidos (línea de pagos en el tiempo)**

```r
# Serie de tiempo agregada por mes
library(dplyr); library(lubridate)
pagos_ts <- df_pagos %>%
  mutate(fecha = as.Date(fecha_pago)) %>%
  group_by(anio = year(fecha), mes = month(fecha, label=TRUE)) %>%
  summarise(total_mes = sum(valor_pago, na.rm=TRUE))

# Gráfico base R (rápido en entrevista)
plot(x = seq_len(nrow(pagos_ts)),
     y = pagos_ts$total_mes,
     type = "l",
     xlab = "Mes (ordenado en el dataset)",
     ylab = "Total de pagos",
     main = "Total mensual de pagos (AsegurAct)")
```

``` 
