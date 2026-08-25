# Categoricals & Optimización de Memoria en Pandas (Nivel Mid/Senior)

---

## 1. El Problema de la Memoria en Pandas (`object` vs `category`)

Por defecto, Pandas almacena las cadenas de texto como tipo `object` (punteros de Python a objetos en memoria). Si una columna tiene 1.000.000 de filas con el valor `'España'`, Pandas guarda 1.000.000 de punteros independientes.

El tipo **`category`** implementa *Dictionary Encoding*:
* Guarda los valores únicos (*categories*) en una lista interna indexada: `['España', 'Francia', 'Alemania']`.
* En la columna solo almacena enteros pequeños (`int8` o `int16`) que apuntan a dicho índice.

### Cuándo usar `category`:
* **Regla de oro:** Cuando el número de valores únicos es menor al **50%** del total de filas (baja y media cardinalidad: países, estados, categorías, género, tallas).
* **Beneficio:** Reducción de uso de RAM entre un **60% y 90%** y aceleración drástica en operaciones de `groupby`, `sort_values` y filtros.

```python
import pandas as pd
import numpy as np

# Simulación de dataset (1 millón de transacciones)
n = 1_000_000
df = pd.DataFrame({
    'pais': np.random.choice(['ES', 'FR', 'DE', 'IT', 'PT'], size=n),
    'tipo_cliente': np.random.choice(['VIP', 'REGULAR', 'NUEVO'], size=n),
    'importe': np.random.uniform(10.0, 500.0, size=n)
})

# Inspección de memoria real (deep=True obligatorio para inspeccionar strings)
print("Memoria antes:")
print(df.memory_usage(deep=True) / 1024**2)  # En MB (~120 MB)

# Conversión a category
df['pais'] = df['pais'].astype('category')
df['tipo_cliente'] = df['tipo_cliente'].astype('category')

print("Memoria después:")
print(df.memory_usage(deep=True) / 1024**2)  # En MB (~10 MB -> Reducción de >90%)
```

---

## 2. Categorías Ordenadas (*Ordered Categoricals*)

Las categorías pueden tener un orden jerárquico inherente (ej. `Bajo < Medio < Alto`, encuestas de satisfacción, tallas `S < M < L < XL`).

### Ventajas:
* Permite comparaciones matemáticas directas (`>`, `<`, `<=`) sin necesidad de mapear a números.
* Funciones como `.min()`, `.max()`, o `.sort_values()` respetan el orden de negocio en lugar del orden alfabético.

```python
from pandas.api.types import CategoricalDtype

# Definir el tipo categórico ordenado
nivel_satisfaccion = CategoricalDtype(
    categories=['Muy Malo', 'Malo', 'Neutro', 'Bueno', 'Excelente'],
    ordered=True
)

df = pd.DataFrame({
    'encuesta': ['Excelente', 'Malo', 'Muy Malo', 'Bueno', 'Neutro', 'Excelente']
})

df['encuesta'] = df['encuesta'].astype(nivel_satisfaccion)

# ✅ Ordenación natural según la lógica de negocio
df_ordenado = df.sort_values('encuesta')

# ✅ Filtrado directo con comparadores matemáticos
df_satisfechos = df[df['encuesta'] >= 'Bueno']
```

---

## 3. Downcasting Numérico (`int` y `float`)

Por defecto, Pandas asigna `int64` (8 bytes por valor) y `float64` (8 bytes por valor), incluso si los datos son números pequeños o porcentajes.

### Tipos enteros y sus límites:
* `int8`: -128 a 127 (1 byte)
* `int16`: -32.768 a 32.767 (2 bytes)
* `int32`: -2.147.483.648 a 2.147.483.647 (4 bytes)
* `int64`: Hasta 9 x 10^18 (8 bytes)

### Reducción automática con `pd.to_numeric(..., downcast=...)`:
```python
# Downcasting de enteros y flotantes
df['edad'] = pd.to_numeric(df['edad'], downcast='integer')       # Pasa de int64 a int8
df['precio'] = pd.to_numeric(df['precio'], downcast='float')     # Pasa de float64 a float32
```

---

## 4. Función de Optimización Automática de DataFrames

Función estándar en producción para reducir el tamaño de cualquier DataFrame antes de procesarlo:

```python
def optimize_dataframe_memory(df: pd.DataFrame) -> pd.DataFrame:
    df_opt = df.copy()
    for col in df_opt.columns:
        col_type = df_opt[col].dtype
        
        # 1. Optimizar objetos / strings a category si cardinalidad < 50%
        if col_type == 'object' or col_type.name == 'string':
            num_unique = df_opt[col].nunique()
            num_total = len(df_opt[col])
            if num_total > 0 and (num_unique / num_total) < 0.5:
                df_opt[col] = df_opt[col].astype('category')
                
        # 2. Optimizar enteros
        elif np.issubdtype(col_type, np.integer):
            df_opt[col] = pd.to_numeric(df_opt[col], downcast='integer')
            
        # 3. Optimizar flotantes
        elif np.issubdtype(col_type, np.floating):
            df_opt[col] = pd.to_numeric(df_opt[col], downcast='float')
            
    return df_opt
```

---

## 5. Lectura Eficiente por Trozos (*Chunking*) y Column Pruning

Cuando el archivo es demasiado grande para caber en memoria RAM:

### A. Cargar solo las columnas necesarias (`usecols`)
```python
# ❌ Carga todas las columnas a memoria
df = pd.read_csv('dataset_gigante.csv')

# ✅ Carga únicamente las indispensables
columnas_clave = ['fecha', 'cliente_id', 'importe']
df = pd.read_csv('dataset_gigante.csv', usecols=columnas_clave)
```

### B. Especificar `dtype` en la lectura inicial
Evita que Pandas gaste memoria infiriendo tipos en `int64` u `object`:
```python
dtypes_dict = {
    'pais': 'category',
    'estado': 'category',
    'edad': 'int8',
    'importe': 'float32'
}

df = pd.read_csv('dataset_gigante.csv', dtype=dtypes_dict)
```

### C. Procesamiento por lotes (`chunksize`)
```python
total_ventas = 0.0

# Procesa el archivo en bloques de 100.000 filas sin desbordar la memoria
for chunk in pd.read_csv('ventas_masivas.csv', chunksize=100_000, usecols=['importe']):
    total_ventas += chunk['importe'].sum()
```

---

## 6. Trampas Comunes con `category` en Entrevistas Técnicas

* **Añadir un valor no existente en las categorías:**
  ```python
  df['pais'] = df['pais'].astype('category')
  # ❌ Si asignas un valor que no estaba en las categorías originales:
  # Lanza TypeError o asigna NaN silenciosamente
  df.loc[0, 'pais'] = 'MX'
  
  # ✅ Solución: Añadir la categoría primero
  df['pais'] = df['pais'].cat.add_categories(['MX'])
  df.loc[0, 'pais'] = 'MX'
  ```
* **Medir memoria con `df.info()` simple:**
  Por defecto, `df.info()` muestra una estimación superficial. En entrevistas debes usar `df.info(memory_usage='deep')` o `df.memory_usage(deep=True)` para que contabilice los punteros de strings en memoria real.
