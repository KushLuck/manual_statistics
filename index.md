# **Manual de manejo de bases de datos – Python & R**

*Guía rápida para lectura, exploración, limpieza y manipulación de datos.*

---

## **1. Lectura de archivos**

### **CSV**

```python
# Python (ruta directa más común)
import pandas as pd
df = pd.read_csv("archivo.csv")  # Ruta relativa o absoluta
df.head()
````

```python
# Python (seleccionando archivo con ventana emergente)
import pandas as pd
from tkinter import Tk
from tkinter.filedialog import askopenfilename

Tk().withdraw()
archivo = askopenfilename(title="Seleccionar archivo CSV", filetypes=[("CSV files", "*.csv")])
df = pd.read_csv(archivo)
df.head()
```

```r
# R (ruta directa)
library(readr)
df <- read_csv("archivo.csv")
head(df)
```

```r
# R (seleccionando archivo manualmente)
library(readr)
ruta <- file.choose()
df <- read_csv(ruta)
head(df)
```

---

### **Excel**

```python
# Python
df = pd.read_excel("archivo.xlsx")
```

```python
# Python (con selección manual)
from tkinter import Tk
from tkinter.filedialog import askopenfilename
import pandas as pd

Tk().withdraw()
archivo = askopenfilename(title="Seleccionar archivo Excel", filetypes=[("Excel files", "*.xlsx *.xls")])
df = pd.read_excel(archivo)
```

```r
# R
library(readxl)
df <- read_excel("archivo.xlsx")
```

```r
# R (seleccionando archivo)
library(readxl)
ruta <- file.choose()
df <- read_excel(ruta)
```

---

### **SQL**

```python
# Python
import sqlite3
import pandas as pd

conn = sqlite3.connect("basedatos.db")
df = pd.read_sql("SELECT * FROM tabla", conn)
```

```python
# Python (con selección manual)
from tkinter import Tk
from tkinter.filedialog import askopenfilename
import sqlite3
import pandas as pd

Tk().withdraw()
archivo = askopenfilename(title="Seleccionar base de datos", filetypes=[("SQLite DB", "*.db *.sqlite")])
conn = sqlite3.connect(archivo)
df = pd.read_sql("SELECT * FROM tabla", conn)
```

```r
# R
library(DBI)
con <- dbConnect(RSQLite::SQLite(), "basedatos.db")
df <- dbGetQuery(con, "SELECT * FROM tabla")
```

```r
# R (seleccionando archivo)
library(DBI)
ruta <- file.choose()
con <- dbConnect(RSQLite::SQLite(), ruta)
df <- dbGetQuery(con, "SELECT * FROM tabla")
```

---

## **2. Exploración de datos**

```python
df.shape           # Dimensiones
df.columns         # Nombres de columnas
df.info()          # Tipos de datos
df.describe()      # Resumen estadístico
```

```r
dim(df)            # Dimensiones
colnames(df)       # Nombres de columnas
str(df)            # Estructura y tipos
summary(df)        # Resumen estadístico
```

---

## **3. Selección y filtrado**

```python
df["columna"]
df[["col1", "col2"]]
df[df["columna"] > 100]
df[(df["edad"] > 30) & (df["sexo"] == "M")]
```

```r
df$columna
df[, c("col1", "col2")]
df[df$columna > 100, ]
subset(df, edad > 30 & sexo == "M")
```

---

## **4. Ordenamiento**

```python
df.sort_values("columna")
df.sort_values("columna", ascending=False)
```

```r
df[order(df$columna), ]
df[order(-df$columna), ]
```

---

## **5. Agrupación y resumen**

```python
df.groupby("columna")["valor"].mean()
df.groupby("columna").agg({"valor":"sum"})
```

```r
library(dplyr)
df %>% group_by(columna) %>% summarise(prom = mean(valor))
df %>% group_by(columna) %>% summarise(suma = sum(valor))
```

---

## **6. Limpieza y transformación**

```python
df.rename(columns={"old":"new"}, inplace=True)
df["nueva"] = df["a"] + df["b"]
df.dropna()
df.fillna(0)
```

```r
library(dplyr)
df <- rename(df, new = old)
df <- mutate(df, nueva = a + b)
df <- na.omit(df)
df[is.na(df)] <- 0
```

---

## **7. Joins**

```python
pd.merge(df1, df2, on="id", how="inner")  # inner, left, right, outer
```

```r
left_join(df1, df2, by="id")              # left, right, inner, full
```

---

## **8. Exportar datos**

```python
df.to_csv("salida.csv", index=False)
df.to_excel("salida.xlsx", index=False)
```

```r
write_csv(df, "salida.csv")
library(writexl)
write_xlsx(df, "salida.xlsx")
```

---

