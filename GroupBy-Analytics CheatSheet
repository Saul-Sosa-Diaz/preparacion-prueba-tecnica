# Pandas GroupBy & Analytics CheatSheet

---

### 1. Anatomía Básica y `as_index`

* **Por defecto (`as_index=True`):** Las columnas agrupadas pasan a formar el índice (Series o `MultiIndex`).
* **Recomendado (`as_index=False`):** Mantiene la estructura tabular plana (*tidy data*) sin crear índices jerárquicos.

```python
# Forma directa y recomendada:
df_res = df.groupby('pais', as_index=False)['importe'].sum()

# Equivalente tradicional más verbose:
df_res = df.groupby('pais')['importe'].sum().reset_index()
```

---

### 2. Named Aggregations (Múltiples Métricas)

Permite calcular distintas métricas sobre columnas variadas asignando alias descriptivos en un único paso, evitando `MultiIndex` en las columnas:

```python
resumen = df.groupby(['pais', 'categoria'], as_index=False).agg(
    total_ventas=('importe', 'sum'),
    ticket_medio=('importe', 'mean'),
    mediana_importe=('importe', 'median'),
    desv_estandar=('importe', 'std'),
    num_pedidos=('pedido_id', 'count'),
    clientes_unicos=('cliente_id', 'nunique'),
    ultima_compra=('fecha', 'max')
)
```

---

### 3. `.transform()` vs `.agg()` (Pregunta Clave)

| Método | Salida | Caso de Uso Principal |
| :--- | :--- | :--- |
| **`.agg()`** | 1 fila por grupo (reduce dimensionalidad) | Tablas de resumen, reportes y KPIs |
| **`.transform()`** | Mismo número de filas que el DataFrame original | Broadcasting, cálculo de ratios, normalización |

```python
# Ratio sobre el total de su categoría (sin hacer merge):
df['total_cat'] = df.groupby('categoria')['importe'].transform('sum')
df['pct_sobre_cat'] = df['importe'] / df['total_cat']

# Diferencia frente a la media del país:
df['media_pais'] = df.groupby('pais')['importe'].transform('mean')
df['diff_media'] = df['importe'] - df['media_pais']
```

---

### 4. `.filter()` a Nivel de Grupo (`HAVING` de SQL)

Filtra y devuelve registros completos del DataFrame original evaluando una condición agregada sobre el grupo:

```python
# Clientes con al menos 3 pedidos:
df_recurrentes = df.groupby('cliente_id').filter(lambda g: len(g) >= 3)

# Categorías con facturación total superior a 10.000€:
df_top_cat = df.groupby('categoria').filter(lambda g: g['importe'].sum() > 10000)
```

---

### 5. Ventanas Móviles, Lags y Acumulados por Grupo

```python
# 1. Ordenar siempre antes por entidad y tiempo:
df = df.sort_values(['cliente_id', 'fecha'])

# 2. Media móvil de las últimas 3 compras por cliente:
df['roll_media_3'] = (
    df.groupby('cliente_id')['importe']
    .transform(lambda x: x.rolling(3, min_periods=1).mean())
)

# 3. Importe de la transacción anterior (Lag):
df['prev_importe'] = df.groupby('cliente_id')['importe'].shift(1)

# 4. Gasto acumulado histórico (Running Total):
df['gasto_acumulado'] = df.groupby('cliente_id')['importe'].cumsum()
```

---

### 6. Buenas vs. Malas Prácticas en Entrevista Técnica

* **✓ BIEN:** Usar `as_index=False` para evitar reseteos de índice manuales.
* **✓ BIEN:** Usar `.transform()` para broadcasting en lugar de generar tablas agregadas intermedias y hacer `df.merge()`.
* **✓ BIEN:** Usar `'nunique'` o `.nunique()` nativo en lugar de `apply(lambda x: len(x.unique()))`.
* **✗ MAL:** Iterar sobre grupos usando `for name, group in df.groupby()`.
* **✗ MAL:** Renombrar columnas a ciegas con `df.columns = [...]` tras un aggregate con listas.

---

### 7. Funciones Nativas de Agregación

| Función | Descripción | Función | Descripción |
| :--- | :--- | :--- | :--- |
| `'sum'` / `'mean'` / `'median'` | Operaciones numéricas ignorando `NaN` | `'count'` vs `'size'` | `count` omite nulos; `size` cuenta todas las filas |
| `'min'` / `'max'` | Valores extremos del grupo | `'nunique'` | Conteo de valores únicos sin `NaN` |
| `'std'` / `'var'` | Desviación estándar y varianza | `'first'` / `'last'` | Primer o último registro según orden actual |
| `'quantile'` | Percentiles (ej. `lambda x: x.quantile(0.75)`) | `'cumsum'` / `'cummax'` | Acumulados secuenciales dentro del grupo |
