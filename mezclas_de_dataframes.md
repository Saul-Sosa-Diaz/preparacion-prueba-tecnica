# Joins, Merges y Concatenaciones Defensivas en Pandas (Nivel Mid/Senior)

---

## 1. Validación Estricta de Cardinalidad (`validate`)

Uno de los errores más destructivos en pipelines de datos es la **multiplicación silenciosa de filas** causada por duplicados inesperados en las claves de unión.

Pandas permite validar la relación directamente dentro de `pd.merge()` mediante el parámetro `validate`:

| Valor de `validate` | Relación Esperada | Comportamiento si falla |
| :--- | :--- | :--- |
| `'1:1'` / `'one_to_one'` | Clave única en tabla izquierda y derecha | Lanza `MergeError` |
| `'1:m'` / `'one_to_many'` | Clave única en izquierda; múltiple en derecha | Lanza `MergeError` |
| `'m:1'` / `'many_to_one'` | Múltiple en izquierda; única en derecha | Lanza `MergeError` |
| `'m:m'` / `'many_to_many'` | Sin restricciones (comportamiento por defecto) | Genera producto cartesiano |

```python
import pandas as pd

df_pedidos = pd.DataFrame({
    'pedido_id': [1, 2, 3],
    'cliente_id': [101, 102, 103],
    'importe': [50, 80, 120]
})

df_clientes = pd.DataFrame({
    'cliente_id': [101, 102, 102],  # ⚠️ 102 duplicado accidentalmente
    'pais': ['ES', 'FR', 'FR']
})

# ✅ Falla de inmediato en lugar de duplicar la fila silenciosamente en producción
try:
    df_merged = df_pedidos.merge(
        df_clientes, 
        on='cliente_id', 
        how='left', 
        validate='many_to_one'  # m:1
    )
except pd.errors.MergeError as e:
    print(f"Error detectado: {e}")
    # MergeError: Merge keys are not unique in right dataset; not a many-to-one merge
```

---

## 2. Auditoría de Integridad con `indicator=True`

Al hacer un `left`, `right` o `outer` join, necesitas saber qué registros hicieron match y cuáles quedaron huérfanos.

```python
df_pedidos = pd.DataFrame({
    'pedido_id': [1, 2, 3],
    'cliente_id': [101, 102, 999]  # 999 no existe en clientes
})

df_clientes = pd.DataFrame({
    'cliente_id': [101, 102, 104],  # 104 no tiene pedidos
    'nombre': ['Ana', 'Carlos', 'David']
})

df_audit = df_pedidos.merge(
    df_clientes, 
    on='cliente_id', 
    how='outer', 
    indicator=True
)
```

**Resultado (`_merge`):**
```text
   pedido_id  cliente_id  nombre      _merge
0        1.0         101     Ana        both
1        2.0         102  Carlos        both
2        3.0         999     NaN   left_only  <-- Pedido sin cliente registrado
3        NaN         104   David  right_only  <-- Cliente sin pedidos
```

### Comprobación de integridad con aserciones:
```python
# Asegurar que el 100% de los pedidos cruzaron correctamente con un cliente:
assert (df_audit['_merge'] == 'left_only').sum() == 0, "Hay pedidos huérfanos sin cliente"
```

---

## 3. `pd.merge_asof()` — Joins por Proximidad Temporal

Se utiliza cuando necesitas cruzar dos datasets temporales donde **los timestamps no coinciden de forma exacta**, pero necesitas asociar el evento con el estado, cotización o tarifa vigente en ese momento.

### Reglas críticas de `merge_asof`:
1. Ambas tablas **deben estar ordenadas** por la columna temporal.
2. La columna temporal debe ser de tipo numérico o `datetime`.

### Parámetros clave:
* `on`: Columna temporal por la que se aproxima.
* `by`: Columna categórica para emparejar de forma exacta antes de aproximar en tiempo (ej. `ticker` o `user_id`).
* `direction`:
  * `'backward'` (por defecto): Toma el último valor **anterior o igual** al timestamp actual.
  * `'forward'`: Toma el primer valor **posterior o igual**.
  * `'nearest'`: Toma el valor temporal más cercano en distancia absoluta.
* `tolerance`: Límite máximo de diferencia de tiempo permitido (ej. `pd.Timedelta('2 hours')`).

```python
# 1. Cotizaciones de divisas (actualizadas periódicamente)
cotizaciones = pd.DataFrame({
    'fecha': pd.to_datetime(['2024-01-01 10:00', '2024-01-01 10:30', '2024-01-01 11:00']),
    'moneda': ['USD', 'USD', 'USD'],
    'tasa_eur': [1.08, 1.09, 1.07]
}).sort_values('fecha')

# 2. Transacciones de clientes (ocurren en minutos arbitrarios)
transacciones = pd.DataFrame({
    'fecha': pd.to_datetime(['2024-01-01 10:05', '2024-01-01 10:45', '2024-01-01 11:15']),
    'moneda': ['USD', 'USD', 'USD'],
    'importe_usd': [100, 250, 50]
}).sort_values('fecha')

# ✅ Cruzar con la tasa de cambio vigente en el momento exacto de la compra (backward)
df_final = pd.merge_asof(
    transacciones,
    cotizaciones,
    on='fecha',
    by='moneda',
    direction='backward',
    tolerance=pd.Timedelta('1 hour')
)
```

**Resultado (`df_final`):**
```text
                fecha moneda  importe_usd  tasa_eur
0 2024-01-01 10:05:00    USD          100      1.08  <-- Usa la tasa de las 10:00
1 2024-01-01 10:45:00    USD          250      1.09  <-- Usa la tasa de las 10:30
2 2024-01-01 11:15:00    USD           50      1.07  <-- Usa la tasa de las 11:00
```

---

## 4. Concatenación Defensiva (`pd.concat`)

Concatenar múltiples DataFrames sin validar índices o columnas genera nulos y desalineaciones silenciosas.

### Buenas Prácticas en `pd.concat`:
* **`ignore_index=True`:** Obligatorio cuando se apilan filas para evitar índices duplicados (`0, 1, 2, 0, 1, 2`).
* **`verify_integrity=True`:** Lanza error si la concatenación resulta en índices repetidos.
* **`join='inner'`:** Conserva únicamente las columnas comunes a todos los DataFrames involucrados.

```python
# Alinear columnas antes de concatenar listas dinámicas de DataFrames:
dfs = [df_2022, df_2023, df_2024]

# Comprobación de que todos comparten el mismo conjunto de columnas:
columnas_base = set(dfs[0].columns)
for i, d in enumerate(dfs[1:], start=1):
    diferencia = set(d.columns) ^ columnas_base
    if diferencia:
        raise ValueError(f"El DataFrame en posición {i} tiene columnas dispares: {diferencia}")

df_completo = pd.concat(dfs, ignore_index=True)
```

---

## 5. Sufijos y Colisiones de Nombres (`suffixes`)

Por defecto, si dos tablas tienen columnas con el mismo nombre que no forman parte de la clave `on`, Pandas añade `_x` e `_y`. Esto ensucia el código y dificulta la mantenibilidad.

```python
# ❌ Genera 'fecha_x' y 'fecha_y' sin contexto claro
df_merged = df_pedidos.merge(df_envios, on='pedido_id')

# ✅ Nombres explícitos y autodescriptivos
df_merged = df_pedidos.merge(
    df_envios, 
    on='pedido_id', 
    how='left', 
    suffixes=('_pedido', '_envio')
)
```

---

## 6. Guía Definitiva de Selección de `how` (`inner`, `left`, `right`, `outer`)

Para no dudar nunca al elegir el tipo de join, hazte **una sola pregunta**:

> **"¿Qué entidad define el universo principal de mi análisis?"**

### Regla Mental Rápida

| Tipo de Join (`how`) | Pregunta Clave | Comportamiento con Filas no Coincidentes | Cuándo Usarlo | Impacto en Agregaciones / Métricas |
| :--- | :--- | :--- | :--- | :--- |
| **`'inner'`** | *¿Ambos lados son estrictamente obligatorios?* | **Elimina** las filas que no coincidan en ambos DataFrames. | Cuando solo interesan entidades con actividad/relación confirmada. | **Métricas de actividad pura** (ej. gasto medio solo de compradores reales). |
| **`'left'`** | *¿La tabla izquierda es mi población maestra?* | Mantiene **todo lo de la izquierda**; rellena la derecha con `NaN`. | Cuando necesitas conservar la base completa (ej. usuarios aunque tengan 0 pedidos). | **Promedios globales/tasas de conversión** (el 0 cuenta para no inflar la media). |
| **`'right'`** | *¿La tabla derecha es mi población maestra?* | Mantiene **todo lo de la derecha**; rellena la izquierda con `NaN`. | Rara vez necesario (conviene reordenar las tablas y usar `left` por legibilidad). | Idem a `left`, pero anclado a la derecha. |
| **`'outer'`** | *¿Quiero ver todo lo que existe en cualquier lado?* | **Conserva todo**; rellena con `NaN` donde no haya cruce. | Auditorías de integridad de datos, reconciliación o reportes consolidados. | Identificación de registros huérfanos en ambos lados (`indicator=True`). |

---

## 7. Matriz de Decisión General de Operaciones de Unión

| Necesidad de Negocio / Técnico | Método Recomendado | Parámetros Clave |
| :--- | :--- | :--- |
| Unir por clave relacional exacta | `pd.merge()` | `on`, `how`, `validate='1:m'`, `suffixes` |
| Auditar registros sin cruce / huérfanos | `pd.merge()` | `how='outer'`, `indicator=True` |
| Unir por proximidad temporal / logs | `pd.merge_asof()` | `on='fecha'`, `by='id'`, `direction='backward'` |
| Apilar filas o columnas alineadas | `pd.concat()` | `ignore_index=True`, `join='inner'` |
| Cruzar directamente sobre índices | `df.join()` | `lsuffix`, `rsuffix`, `how` |
