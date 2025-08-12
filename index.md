````markdown
# **Manual de manejo de bases de datos – R**
*Guía rápida para lectura, exploración, limpieza y manipulación de datos en R para entrevistas técnicas.*

---

## **1. Lectura de archivos**

### CSV — ruta directa
```r
library(readr)
df <- read_csv("archivo.csv")    # Lee CSV con separador coma
head(df)                         # Primeras 6 filas
````

### CSV — selección manual

```r
library(readr)
ruta <- file.choose()            # Abre diálogo para seleccionar archivo
df <- read_csv(ruta)
head(df)
```

> 💡 Si usa `;` como separador:

```r
df <- read_delim("archivo.csv", delim = ";")
```

---

### Excel — ruta directa

```r
library(readxl)
df <- read_excel("archivo.xlsx")                # Lee primera hoja
head(df)
```

#### Leer una hoja específica

```r
library(readxl)

# Por nombre
df <- read_excel("archivo.xlsx", sheet = "Resumen")

# Por índice
df <- read_excel("archivo.xlsx", sheet = 2)

# Ver todas las hojas antes de leer
excel_sheets("archivo.xlsx")

# Varias hojas combinadas
library(purrr)
df <- excel_sheets("archivo.xlsx") %>%
  set_names() %>%
  map_dfr(~ read_excel("archivo.xlsx", sheet = .x), .id = "hoja")

# Solo un rango específico
df <- read_excel("archivo.xlsx", range = "A1:D10")
```

### Excel — selección manual

```r
library(readxl)
ruta <- file.choose()
df <- read_excel(ruta)
head(df)
```

---

### SQL (SQLite) — ruta directa

```r
library(DBI)
library(RSQLite)

con <- dbConnect(SQLite(), "basedatos.db")      # Conecta/crea BD
df <- dbGetQuery(con, "SELECT * FROM tabla")    # Ejecuta consulta SQL
# dbDisconnect(con)                             # Cierra conexión
head(df)
```

### SQL — selección manual

```r
library(DBI)
library(RSQLite)

ruta <- file.choose()
con <- dbConnect(SQLite(), ruta)
df <- dbGetQuery(con, "SELECT * FROM tabla")
head(df)
```

---

## **2. Exploración de datos**

```r
dim(df)                # Filas y columnas
colnames(df)           # Nombres de columnas
str(df)                # Tipos y estructura
summary(df)            # Resumen estadístico
```

---

## **3. Selección y filtrado**

```r
df$columna                             # Una columna
df[, c("col1", "col2")]                # Varias columnas
df[df$columna > 100, ]                 # Filtrar por condición
subset(df, edad > 30 & ciudad == "Bogotá")  # Filtrar por varias condiciones
```

---

## **4. Ordenamiento**

```r
df[order(df$columna), ]                # Ascendente
df[order(-df$columna), ]               # Descendente
# df[order(df$categoria, -df$valor), ]  # Orden múltiple
```

---

## **5. Agrupación y resumen**

```r
library(dplyr)

df %>%
  group_by(categoria) %>%
  summarise(
    total_venta = sum(venta, na.rm = TRUE),
    prom_venta  = mean(venta, na.rm = TRUE)
  )
```

---

## **6. Limpieza y transformación**

```r
library(dplyr)

df <- df %>%
  rename(new = old) %>%                # Renombrar columna
  mutate(
    nueva = a + b,                     # Nueva columna
    # precio = if_else(is.na(precio), 0, precio),
    # precio_usd = precio / 4000
  )

df <- na.omit(df)                      # Eliminar filas con NA
df[is.na(df)] <- 0                     # Rellenar NA con 0
```

---

## **7. Joins**

```r
library(dplyr)
df_join <- left_join(df1, df2, by = "id")  # LEFT JOIN
```

---

## **8. Exportar datos**

```r
library(readr)
write_csv(df, "salida.csv")            # Exporta CSV

library(writexl)
write_xlsx(df, "salida.xlsx")          # Exporta Excel
```

---

## **9. Fechas y tiempos**

```r
library(lubridate)
df$fecha <- as.Date(df$fecha)          # Convierte a fecha
df$anio  <- year(df$fecha)             # Extrae año
df$mes   <- month(df$fecha, label=TRUE)# Extrae mes abreviado

# Promedio mensual
df %>%
  group_by(anio, mes) %>%
  summarise(prom_trm = mean(trm, na.rm = TRUE))
```

---

## **10. Tipos y conversión**

```r
df$valor <- as.numeric(df$valor)       # Texto → numérico
df$categoria <- as.factor(df$categoria)# Texto → factor
```

---

## **11. Valores perdidos y outliers**

```r
colSums(is.na(df))                     # Conteo NA por columna
df <- df %>% filter(!is.na(valor))     # Eliminar NA en columna
quantile(df$valor, probs = c(.01, .99), na.rm = TRUE) # Percentiles
```

---

## **12. Pivot (ancho ↔ largo)**

```r
library(tidyr)

# Largo → Ancho
wide <- df %>% pivot_wider(names_from = categoria, values_from = valor)

# Ancho → Largo
long <- df %>% pivot_longer(cols = -id, names_to = "metrica", values_to = "valor")
```

---

## **13. Conteos y proporciones**

```r
library(dplyr)
df %>% count(categoria, name = "n")              # Conteo
df %>% group_by(categoria) %>% summarise(prop = n()/nrow(df))  # Proporción
```

---

## **14. Duplicados y llaves**

```r
any(duplicated(df$id))                # TRUE si hay duplicados
df_sin_dups <- df %>% distinct()      # Quitar duplicados
```

---

## **15. Ordenamiento multi-columna y top-N**

```r
df %>% arrange(desc(valor), categoria) %>% head(10)
```

---

## **16. Chequeos rápidos (QA)**

```r
stopifnot(!any(is.na(df$id)))          # Falla si hay NA en id
```

---

## **17. Muestras y splits**

```r
set.seed(123)
idx <- sample.int(nrow(df), 0.7*nrow(df))
train <- df[idx, ]
test  <- df[-idx, ]
```

---

## **18. Gráficos rápidos**

```r
plot(df$fecha, df$trm, type="l")       # Serie de tiempo simple
```

```
