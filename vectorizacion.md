# Cheatsheet: Optimizando Pandas con NumPy (`np.where` & `np.select`) y Vectorización de Filtros

Guía rápida y mejores prácticas para reemplazar `.apply(lambda ...)` y bucles imperativos por operaciones vectorizadas en Pandas y NumPy, logrando mejoras de rendimiento de hasta **100x**.

---

## 1. Asignación Condicional (Reemplazando `apply(lambda ...)`)

### `np.where` — Condición Binaria (`if-else`)
Ideal para evaluar una sola condición de 2 caminos.

```python
import pandas as pd
import numpy as np

# ❌ LENTO
df['categoria'] = df['precio'].apply(lambda x: 'Caro' if x > 100 else 'Barato')

# ✅ RÁPIDO (Vectorizado)
df['categoria'] = np.where(df['precio'] > 100, 'Caro', 'Barato')
```

### `np.select` — Múltiples Condiciones (`if-elif-else`)
Ideal para cadenas de condiciones. Evita `lambda`s anidados e ininteligibles.

```python
# ❌ LENTO
df['nivel'] = df['precio'].apply(lambda x: 'Alto' if x > 150 else ('Medio' if x >= 70 else 'Bajo'))

# ✅ RÁPIDO Y LIMPIO (Vectorizado)
condiciones = [
    df['precio'] > 150,
    (df['precio'] >= 70) & (df['precio'] <= 150)
]
opciones = ['Alto', 'Medio']

df['nivel'] = np.select(condiciones, opciones, default='Bajo')
```

---

## 2. Transformaciones y Cálculos en los Datos

### Cálculos Matemáticos Condicionales
Puedes pasar columnas completas con operaciones aritméticas como resultado en `np.where` y `np.select`.

```python
# ✅ Descuento condicional vectorizado
df['precio_final'] = np.where(
    df['tipo_cliente'] == 'VIP',
    df['precio'] * 0.80,  # 20% descuento si es VIP
    df['precio'] + 15     # Recargo de 15 si no
)

# ✅ Múltiples fórmulas según categoría
condiciones = [
    df['tipo'] == 'VIP',
    df['tipo'] == 'Regular'
]
opciones = [
    df['precio'] * 0.80,
    df['precio'] * 0.95
]

df['precio_final'] = np.select(condiciones, opciones, default=df['precio'])
```

### Transformaciones Directas (Sin `apply` ni `lambda`)
Para transformaciones globales, utiliza la API nativa vectorizada de Pandas/NumPy:

| Operación | ❌ Forma Lenta (`apply`) | ✅ Forma Rápida (Vectorizada) |
| :--- | :--- | :--- |
| **Suma de columnas** | `df.apply(lambda r: r['a'] + r['b'], axis=1)` | `df['a'] + df['b']` |
| **Funciones matemáticas** | `df['val'].apply(lambda x: np.log(x))` | `np.log(df['val'])` |
| **Texto / Strings** | `df['txt'].apply(lambda x: x.upper().strip())` | `df['txt'].str.strip().str.upper()` |
| **Fechas** | `df['fecha'].apply(lambda x: x.year)` | `df['fecha'].dt.year` |

### Casos Complejos: `np.vectorize`
Si la lógica requiere funciones Python puras difíciles de traducir a expresiones vectoriales simples:

```python
def logica_compleja(precio, tipo):
    if tipo == 'VIP' and precio > 150:
        return precio * 0.7
    return precio

df['precio_final'] = np.vectorize(logica_compleja)(df['precio'], df['tipo_cliente'])
```

---

## 3. Vectorización de Filtros y Reducción de Filas

> **Regla de oro:** NO usar `np.where` ni bucles `apply` para filtrar o eliminar filas. Para reducir la dimensionalidad de un DataFrame se deben usar máscaras booleanas vectorizadas, operadores bit a bit (`&`, `|`, `~`) o métodos vectorizados nativos (`isin`, `between`, `str.contains`).

### A. Filtrado Simple vs Filtrado con Negación (`~`)
El operador `~` invierte la máscara booleana a nivel de bits de forma instantánea en C/NumPy:

```python
# ❌ LENTO (Uso innecesario de apply para filtrar)
df_activos = df[df['estado'].apply(lambda x: x != 'Inactivo')]

# ✅ RÁPIDO (Máscara booleana directa)
df_activos = df[df['estado'] == 'Activo']

# ✅ RÁPIDO (Negación vectorizada con ~)
filtro_inactivos = df['estado'] == 'Inactivo'
df_activos = df[~filtro_inactivos]
```

### B. Condiciones Múltiples Vectorizadas (AND / OR / NOT)
En Pandas/NumPy no se usan las palabras clave `and`, `or`, `not` para arrays, sino los operadores bit a bit `&`, `|`, `~`.

```python
# ❌ INCORRECTO: Lanzará ValueError: The truth value of a Series is ambiguous
# df_filtrado = df[df['precio'] > 50 and df['stock'] > 0]

# ✅ CORRECTO: Paréntesis obligatorios por precedencia de operadores
df_filtrado = df[(df['precio'] > 50) & (df['stock'] > 0)]

# ✅ Combinación compleja con negación:
df_complejo = df[~(df['categoria'] == 'Obsol') & ((df['ventas'] > 100) | (df['rating'] >= 4.5))]
```

### C. Métodos Vectorizados Nativos de Filtrado
Evitan escribir múltiples comparaciones manuales y están altamente optimizados:

| Caso de Uso | ❌ Forma Lenta / Imperativa | ✅ Método Vectorizado Nativo |
| :--- | :--- | :--- |
| **Pertenencia a lista** | `df[df['pais'].apply(lambda x: x in ['ES', 'FR', 'DE'])]` | `df[df['pais'].isin(['ES', 'FR', 'DE'])]` |
| **Exclusión de lista** | `df[df['pais'].apply(lambda x: x not in ['ES', 'FR'])]` | `df[~df['pais'].isin(['ES', 'FR'])]` |
| **Rango numérico** | `df[(df['edad'] >= 18) & (df['edad'] <= 65)]` | `df[df['edad'].between(18, 65)]` |
| **Búsqueda de texto** | `df[df['codigo'].apply(lambda x: 'AA' in str(x))]` | `df[df['codigo'].str.contains('AA', na=False)]` |
| **Exclusión de texto** | `df[df['codigo'].apply(lambda x: 'AA' not in str(x))]` | `df[~df['codigo'].str.contains('AA', na=False)]` |
| **Valores nulos** | `df[df['importe'].apply(lambda x: pd.isna(x))]` | `df[df['importe'].isna()]` / `df[~df['importe'].isna()]` |

### D. Vectorización a Nivel de Grupo (`GroupBy.filter` vs `.transform`)
Para filtrar filas evaluando condiciones sobre el **grupo completo** (el `HAVING` de SQL):

```python
# ❌ LENTO: Calcular agregados, hacer merge y luego filtrar
totales = df.groupby('cliente_id')['importe'].sum().reset_index()
df_merged = df.merge(totales, on='cliente_id', suffixes=('', '_tot'))
df_top = df_merged[df_merged['importe_tot'] > 1000]

# ✅ RÁPIDO OPCIÓN 1: Máscara vectorizada con transform (Permite indexación booleana directa)
mask_vip = df.groupby('cliente_id')['importe'].transform('sum') > 1000
df_top = df[mask_vip]

# ✅ RÁPIDO OPCIÓN 2: GroupBy.filter nativo
df_top = df.groupby('cliente_id').filter(lambda g: g['importe'].sum() > 1000)
```

---

## 4. Matriz de Decisión (Senior Cheat Sheet)

| Tarea | Herramienta Correcta | Ejemplo de Código |
| :--- | :--- | :--- |
| **Filtrar / Reducir filas** | Máscara Booleana + `~` / `.isin()` / `.between()` | `df[~condicion]` o `df[df['c'].between(a, b)]` |
| **Filtrar por métrica de grupo** | Máscara con `.transform()` o `groupby().filter()` | `df[df.groupby('id')['val'].transform('sum') > 100]` |
| **Nueva columna (2 opciones)** | `np.where` | `np.where(c, val_true, val_false)` |
| **Nueva columna (3+ opciones)** | `np.select` | `np.select(conds, opts, default)` |
| **Modificar filas específicas *in-place*** | `df.loc[]` | `df.loc[cond, 'col'] = nuevo_valor` |
| **Transformación estándar (str/math/dt)** | Métodos nativos de Series | `df['col'].str.lower()` / `np.log(df['col'])` |
