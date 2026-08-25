# Cheatsheet: Optimizando Pandas con NumPy (`np.where` & `np.select`)

Guía rápida y mejores prácticas para reemplazar `.apply(lambda ...)` por operaciones vectorizadas en Pandas y NumPy, logrando mejoras de rendimiento de hasta **100x**.

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

## 3. Filtrado de Filas (Uso Correcto de `~`)

> **Regla de oro:** NO usar `np.where` ni `np.select` para filtrar o eliminar filas (reducir la dimensión del DataFrame). Usa máscaras booleanas directas y el operador de negación bit a bit `~`.

### Filtrado con Negación (`~`)
```python
# Crear máscara booleana
filtro_inactivos = df['estado'] == 'Inactivo'

# ✅ Excluir inactivos (NOT)
df_activos = df[~filtro_inactivos]

# ✅ Combinación con paréntesis obligatorios (...)
df_filtrado = df[~filtro_inactivos & (df['compras'] > 100)]
```

*Nota:* Recuerda envolver siempre cada subcondición entre paréntesis `(...)` al combinar con `&` (AND), `|` (OR) o `~` (NOT).

---

## 4. Matriz de Decisión (Senior Cheat Sheet)

| Tarea | Herramienta Correcta | Ejemplo de Código |
| :--- | :--- | :--- |
| **Filtrar / Reducir filas** | Máscara Booleana + `~` | `df[~condicion]` |
| **Nueva columna (2 opciones)** | `np.where` | `np.where(c, val_true, val_false)` |
| **Nueva columna (3+ opciones)** | `np.select` | `np.select(conds, opts, default)` |
| **Modificar filas específicas *in-place*** | `df.loc[]` | `df.loc[cond, 'col'] = nuevo_valor` |
| **Transformación estándar (str/math/dt)** | Métodos nativos | `df['col'].str.lower()` |
