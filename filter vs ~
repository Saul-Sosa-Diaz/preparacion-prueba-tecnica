# Pandas Filter & Negation (`~`) CheatSheet (Nivel Mid/Senior)

---

### 1. Resumen Conceptual Rápido

| Herramienta | Ámbito de Operación | Equivalente SQL | ¿Qué devuelve / modifica? |
| :--- | :--- | :--- | :--- |
| **`df[~mask]`** | Fila a fila (*Row-level*) | `WHERE NOT (...)` | Filtra filas individuales invirtiendo booleanos |
| **`df.filter()`** | Esquema (*Schema / Columns*) | Cláusula `SELECT` | Subconjunto de columnas por nombre/regex |
| **`df.groupby().filter()`** | Entidad / Grupo (*Group-level*) | Cláusula `HAVING` | Conserva o descarta grupos completos |

---

### 2. El Operador `~` (Negación Booleana Fila a Fila)

Invierte cada valor booleano (`True` $
ightarrow$ `False`, `False` $
ightarrow$ `True`).

```python
# 1. Excluir valores específicos con isin:
df_validos = df[~df['estado'].isin(['CANCELADO', 'RECHAZADO'])]

# 2. Excluir cadenas de texto (manejo seguro de nulos con na=False):
df_sin_aa = df[~df['organica'].str.contains('AA', na=False)]

# 3. Excluir nulos explícitamente:
df_con_importe = df[~df['importe'].isna()]

# 4. Excluir rango numérico:
df_fuera_rango = df[~df['edad'].between(18, 65)]
```

> **Regla de Paréntesis con `~`:** En Python, el operador `~` tiene mayor precedencia que las comparaciones (`>`, `<`, `==`). Encierra **siempre** la condición entre paréntesis:
> ```python
> # ❌ df[~df['importe'] > 100]    -> TypeError / Invalido
> # ✅ df[~(df['importe'] > 100)]  -> Correcto
> ```

---

### 3. `DataFrame.filter()` (Selección Dinámica de Columnas)

No evalúa el contenido de las filas, sino los nombres de las columnas o índices.

```python
# A. Por subcadena de texto ('like'):
df_fechas = df.filter(like='fecha')

# B. Por expresión regular ('regex'):
df_codigos_o_ids = df.filter(regex=r'(_id|_cod)$')

# C. Por lista exacta de columnas ('items'):
df_subset = df.filter(items=['cliente_id', 'pais', 'importe'])

# D. Filtrar sobre el índice de filas (axis=0):
df_index_sample = df.filter(like='2024', axis=0)
```

---

### 4. `GroupBy.filter()` (El `HAVING` de SQL)

Evalúa una condición lógica agregada sobre el **grupo completo**. Si devuelve `True`, se devuelven todas las filas originales de ese grupo; si devuelve `False`, se eliminan todas.

```python
# A. Conservar grupos según su tamaño / conteo:
df_recurrentes = df.groupby('cliente_id').filter(lambda g: len(g) >= 3)

# B. Conservar grupos según un total acumulado:
df_top_clientes = df.groupby('cliente_id').filter(lambda g: g['importe'].sum() > 1000)

# C. Conservar grupos donde NINGUNA fila cumpla un criterio (combinando con ~):
df_sin_rechazos = df.groupby('cliente_id').filter(lambda g: ~(g['estado'] == 'RECHAZADO').any())

# D. Conservar grupos con una antigüedad o condición temporal:
df_activos = df.groupby('cliente_id').filter(lambda g: (g['dias_inactivo'] < 30).all())
```

---

### 5. Casos de Estudio en Entrevistas Técnicas

#### Caso A: "Elimina transacciones canceladas" vs "Elimina usuarios con cancelaciones"

```python
# 1. Nivel fila (~): Elimina solo las filas canceladas (el usuario se queda con las demás)
df_limpio = df[~(df['estado'] == 'CANCELADO')]

# 2. Nivel grupo (groupby.filter): Elimina al usuario completo si canceló alguna vez
df_usuarios_limpios = df.groupby('user_id').filter(lambda g: ~(g['estado'] == 'CANCELADO').any())
```

#### Caso B: Limpieza de columnas residuales

```python
# Eliminar columnas temporales de cálculo (ej. las que empiezan por 'tmp_')
columnas_validas = [col for col in df.columns if not col.startswith('tmp_')]
df = df[columnas_validas]

# Equivalente directo con filter regex:
df = df.filter(regex=r'^(?!tmp_)')
```

---

### 6. Buenas vs. Malas Prácticas

* **✓ BIEN:** Usar `na=False` en `.str.contains()` al negar con `~` para no perder o arrastrar `NaN`.
* **✓ BIEN:** Usar `.all()` o `.any()` dentro de `GroupBy.filter()` para devolver un único booleano escalar por grupo.
* **✗ MAL:** Usar `df.filter()` intentando filtrar por valores de filas (ej. `df.filter(df['importe'] > 10)` no existe y lanza error).
* **✗ MAL:** Intentar hacer un `groupby().filter()` que devuelva una Serie en vez de un booleano `True`/`False`.
