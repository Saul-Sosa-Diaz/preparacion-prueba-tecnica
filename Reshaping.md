# Reshaping en Pandas: `melt`, `pivot_table`, `pivot` y `explode`

---

## 1. El Concepto Fundamental: Ancho (*Wide*) vs Largo (*Long / Tidy*)

En análisis de datos y modelado, el formato **Long (Tidy Data)** es el estándar de oro:
* Cada variable forma una columna.
* Cada observación forma una fila.
* Cada tipo de unidad observacional forma una tabla.

| Formato | Aspecto | Ventaja | Desventaja |
| :--- | :--- | :--- | :--- |
| **Wide (Ancho)** | Múltiples columnas para la misma métrica (ej. `2021`, `2022`, `2023`). | Fácil lectura para humanos en Excel. | Difícil de filtrar, agrupar con `groupby`, o graficar con Seaborn/Plotly. |
| **Long (Largo)** | Columnas de identificación + Columna Variable + Columna Valor. | Ideal para `groupby`, ML, agregaciones y visualizaciones. | Menos legible a simple vista en tablas gigantes. |

---

## 2. `pd.melt()` — De Formato Ancho (*Wide*) a Largo (*Long*)

Desenrolla o "despivota" un DataFrame, transformando nombres de columnas en valores dentro de una nueva columna.

### Parámetros Clave:
* `id_vars`: Columnas que se mantienen fijas (identificadores).
* `value_vars`: Columnas que se quieren "desenrollar" a filas (si no se especifica, usa todas las que no están en `id_vars`).
* `var_name`: Nombre de la nueva columna que contendrá los nombres de las antiguas columnas.
* `value_name`: Nombre de la nueva columna que contendrá los valores numéricos/datos.

```python
import pandas as pd

# Dataset en formato Ancho (Típico reporte de ventas en Excel)
df_wide = pd.DataFrame({
    'empleado': ['Ana', 'Carlos'],
    'departamento': ['Ventas', 'IT'],
    '2022': [15000, 22000],
    '2023': [18000, 24000],
    '2024': [21000, 29000]
})

# ✅ Transformar a formato Largo (Tidy)
df_long = pd.melt(
    df_wide,
    id_vars=['empleado', 'departamento'],
    value_vars=['2022', '2023', '2024'],
    var_name='año',
    value_name='ventas'
)
```

**Resultado (`df_long`):**
```text
  empleado departamento   año  ventas
0      Ana       Ventas  2022   15000
1   Carlos           IT  2022   22000
2      Ana       Ventas  2023   18000
3   Carlos           IT  2023   24000
4      Ana       Ventas  2024   21000
5   Carlos           IT  2024   29000
```

---

## 3. `df.pivot()` vs `df.pivot_table()` — De Largo (*Long*) a Ancho (*Wide*)

Ambos convierten filas en columnas, pero tienen una diferencia crítica evaluada en entrevistas:

* **`df.pivot()`:** Requiere que cada combinación de índice y columna sea **única**. Si hay duplicados, lanza `ValueError: Index contains duplicate entries`.
* **`df.pivot_table()`:** Admite duplicados y aplica una función de **agregación** (`mean`, `sum`, `count`, etc.).

### Sintaxis de `pivot_table`:
* `index`: Columnas que quedan como filas en la nueva tabla.
* `columns`: Columna cuyos valores únicos se convertirán en nuevas cabeceras de columna.
* `values`: Columna de donde se extraen los valores a agregar.
* `aggfunc`: Función de agregación (`'sum'`, `'mean'`, etc., por defecto `'mean'`).
* `fill_value`: Valor para reemplazar nulos generados en combinaciones inexistentes.

```python
df_transacciones = pd.DataFrame({
    'tienda': ['Madrid', 'Madrid', 'Barcelona', 'Madrid', 'Barcelona'],
    'categoria': ['Ropa', 'Ropa', 'Ropa', 'Calzado', 'Calzado'],
    'importe': [100, 50, 80, 120, 200]
})

# ✅ Resumen cruzado agregando con suma
df_pivot = df_transacciones.pivot_table(
    index='tienda',
    columns='categoria',
    values='importe',
    aggfunc='sum',
    fill_value=0
).reset_index()
```

**Resultado (`df_pivot`):**
```text
categoria     tienda  Calzado  Ropa
0          Barcelona      200    80
1             Madrid      120   150
```

---

## 4. `df.explode()` — Desempaquetar Listas o Arrays a Filas

Toma columnas que contienen listas, sets o arrays y crea **una nueva fila por cada elemento de la lista**, duplicando los valores del resto de columnas.

### Caso Típico: Tags, Categorías Múltiples o IDs Anidados
```python
df_articulos = pd.DataFrame({
    'articulo_id': [1, 2],
    'etiquetas': [['python', 'datos', 'ia'], ['sql', 'pandas']]
})

# ✅ Explotar la lista a filas individuales
df_exploded = df_articulos.explode('etiquetas')
```

**Resultado (`df_exploded`):**
```text
   articulo_id etiquetas
0            1    python
0            1     datos
0            1        ia
1            2       sql
1            2    pandas
```

### Explotar Cadenas Separadas por Comas:
```python
df_texto = pd.DataFrame({
    'usuario': ['A', 'B'],
    'hobbies': ['futbol,cine,lectura', 'videojuegos,pesca']
})

# Convertir string a lista con .str.split y luego explode:
df_hobbies = (
    df_texto
    .assign(hobbies=df_texto['hobbies'].str.split(','))
    .explode('hobbies')
)
```

---

## 5. `stack()` y `unstack()` — Pivoteo basado en Índices

Trabajan a nivel de `MultiIndex`:

* **`df.unstack()`:** Pasa un nivel del **índice de filas a columnas** (ensancha la tabla).
* **`df.stack()`:** Pasa un nivel de las **columnas a filas** (alarga la tabla).

```python
# Partiendo de un groupby con índice jerárquico:
g = df_transacciones.groupby(['tienda', 'categoria'])['importe'].sum()

# unstack() convierte el nivel 'categoria' en columnas:
tabla_ancha = g.unstack(fill_value=0)

# stack() vuelve a comprimir las columnas en filas:
tabla_larga = tabla_ancha.stack()
```

---

## 6. Matriz Comparativa de Métodos de Reshaping

| Método | Entrada | Salida | ¿Maneja Duplicados? | Caso Típico |
| :--- | :--- | :--- | :--- | :--- |
| **`pd.melt()`** | Ancho | Largo | Sí | Desenrollar reportes de Excel con años/meses en columnas. |
| **`df.pivot()`** | Largo | Ancho | ❌ Lanza error si hay claves repetidas | Matriz uno a uno donde la combinación es clave primaria. |
| **`df.pivot_table()`** | Largo | Ancho | ✅ Agrega (`sum`, `mean`, etc.) | Matrices de cohortes, resúmenes cruzados bidimensionales. |
| **`df.explode()`** | Listas en celdas | Filas individuales | Sí | Normalizar tags, arrays de JSON o strings delimitados. |
| **`df.unstack()`** | MultiIndex en filas | Columnas anchas | Sí (en el MultiIndex) | Aplanar series agrupadas complejas. |
