Perfecto ✅
Vamos a rehacer tu **manual** desde cero, pero **solo en R**, con:

* Índice inicial simple y clicable.
* Comentarios claros línea por línea, pensados para entrevista.
* Ejemplos reales y listos para copiar/pegar.
* Sin enlaces de "volver al índice" para que sea limpio y directo.

Aquí tienes el nuevo **`manual_estadistica.md`**:

---

````markdown
# **Manual de manejo de bases de datos – R**
*Guía rápida para lectura, exploración, limpieza y manipulación de datos en R.*

---

## 🧭 Índice

1. [Lectura de archivos](#1-lectura-de-archivos)
2. [Exploración de datos](#2-exploración-de-datos)
3. [Selección y filtrado](#3-selección-y-filtrado)
4. [Ordenamiento](#4-ordenamiento)
5. [Agrupación y resumen](#5-agrupación-y-resumen)
6. [Limpieza y transformación](#6-limpieza-y-transformación)
7. [Joins](#7-joins)
8. [Exportar datos](#8-exportar-datos)

---

## **1. Lectura de archivos**

### CSV — ruta directa
```r
library(readr)                         # Librería para lectura rápida de texto (CSV, TSV)
df <- read_csv("archivo.csv")          # Lee CSV con separador coma
head(df)                               # Muestra primeras 6 filas para verificar
````

### CSV — selección manual

```r
library(readr)
ruta <- file.choose()                  # Abre diálogo para seleccionar archivo
df <- read_csv(ruta)                   # Lee CSV elegido
head(df)
```

> 💡 **Tip separador**: si el CSV usa `;`, cambia a `read_delim` con `delim=";"`

```r
df <- read_delim("archivo.csv", delim = ";")
```

---

### Excel — ruta directa

```r
library(readxl)                        # Librería para leer Excel
df <- read_excel("archivo.xlsx")       # Lee primera hoja por defecto
# df <- read_excel("archivo.xlsx", sheet = "Resumen")  # Hoja específica
head(df)
```

### Excel — selección manual

```r
library(readxl)
ruta <- file.choose()                  # Selección manual
df <- read_excel(ruta)
head(df)
```

---

### SQL (SQLite) — ruta directa

```r
library(DBI)                           # Conexión genérica a bases de datos
library(RSQLite)                       # Driver para SQLite

con <- dbConnect(SQLite(), "basedatos.db")  # Conecta/crea BD SQLite
df <- dbGetQuery(con, "SELECT * FROM tabla") # Ejecuta SQL y devuelve data.frame
# dbDisconnect(con)                    # Cierra conexión (buena práctica)
head(df)
```

### SQL — selección manual

```r
library(DBI)
library(RSQLite)

ruta <- file.choose()                  # Seleccionar archivo .db o .sqlite
con <- dbConnect(SQLite(), ruta)
df <- dbGetQuery(con, "SELECT * FROM tabla")
head(df)
```

---

## **2. Exploración de datos**

```r
dim(df)                                # Número de filas y columnas
colnames(df)                           # Nombres de columnas
str(df)                                # Estructura: tipo de cada columna y muestra de datos
summary(df)                            # Resumen estadístico
```

---

## **3. Selección y filtrado**

```r
df$columna                             # Seleccionar una columna (vector)
df[, c("col1", "col2")]                # Seleccionar varias columnas
df[df$columna > 100, ]                 # Filtrar por condición simple
subset(df, edad > 30 & ciudad == "Bogotá")  # Filtrar con múltiples condiciones
```

---

## **4. Ordenamiento**

```r
df[order(df$columna), ]                # Orden ascendente por una columna
df[order(-df$columna), ]               # Orden descendente
# df[order(df$categoria, -df$valor), ]  # Ordenar por múltiples columnas
```

---

## **5. Agrupación y resumen**

```r
library(dplyr)

df %>%
  group_by(categoria) %>%
  summarise(
    total_venta = sum(venta, na.rm = TRUE),   # Suma con NA ignorados
    prom_venta  = mean(venta, na.rm = TRUE)   # Promedio con NA ignorados
  )
```

---

## **6. Limpieza y transformación**

```r
library(dplyr)

df <- df %>%
  rename(new = old) %>%                # Renombra columna old → new
  mutate(
    nueva = a + b,                     # Crea columna derivada
    # precio = if_else(is.na(precio), 0, precio),  # Relleno selectivo de NA
    # precio_usd = precio / 4000                  # Conversión de moneda
  )

df <- na.omit(df)                      # Elimina filas con NA
df[is.na(df)] <- 0                     # Rellena todos los NA con 0 (cuidado: todo el DF)
```

---

## **7. Joins**

```r
library(dplyr)

# LEFT JOIN: mantiene todas las filas de df1
df_join <- left_join(df1, df2, by = "id")

# Ejemplo típico: mantener todos los clientes aunque no tengan ventas
# df_join <- left_join(df_clientes, df_ventas, by = "id_cliente")
```

---

## **8. Exportar datos**

```r
library(readr)
write_csv(df, "salida.csv")            # Exporta a CSV (UTF-8 por defecto)

library(writexl)
write_xlsx(df, "salida.xlsx")          # Exporta a Excel
```

```

---

Si quieres, el siguiente paso podría ser **añadir al final una sección de funciones estadísticas rápidas** (media, mediana, correlación, máximo/mínimo, conteos) para que en la entrevista tengas también lo más usado en análisis exploratorio.  

¿Quieres que te agregue esa sección de análisis estadístico rápido?
```
