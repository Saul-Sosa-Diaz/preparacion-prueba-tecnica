# 📊 Cheatsheet: `.agg()` vs `.transform()` en Pandas

Guía de referencia rápida y visual para dominar las operaciones de agregación y broadcasting en `groupby`.

---

## 🧠 1. Reglas Nemotécnicas Rápidas

| Método | Analogía Visual | Acción sobre la Tabla | Resultado |
| :--- | :--- | :--- | :--- |
| **`.agg()`** | **A**cordeón / **A**plastador | **Encoge la tabla.** Colapsa múltiples filas en una sola fila resumen por grupo. | Devuelve $N$ filas ($N$ = número de grupos únicos). |
| **`.transform()`** | **T**ransparente / E**t**iqueta | **Mantiene el tamaño.** Calcula la métrica del grupo y se la asigna (*broadcast*) a cada fila individual. | Devuelve la **misma longitud** ($M$ filas) que el DataFrame original. |

---

## 🔍 2. Comparativa Visual con Datos

Supongamos el siguiente DataFrame `df` (4 filas):

```python
import pandas as pd

data = {
    'cliente': ['Ana', 'Ana', 'Bob', 'Bob'],
    'importe': [100.0, 300.0, 50.0, 50.0]
}
df = pd.DataFrame(data)
```

```
   cliente  importe
0      Ana    100.0
1      Ana    300.0
2      Bob     50.0
3      Bob     50.0
```

### Opción A: `.agg('sum')`
> *"Dime la foto global por cliente"*

```python
resumen = df.groupby('cliente', as_index=False)['importe'].agg('sum')
```
**Salida (2 filas):**
```
  cliente  importe
0     Ana    400.0
1     Bob    100.0
```

---

### Opción B: `.transform('sum')`
> *"Añade una columna con el total del cliente sin alterar mis filas originales"*

```python
df['total_cliente'] = df.groupby('cliente')['importe'].transform('sum')
```
**Salida (4 filas intactas):**
```
  cliente  importe  total_cliente
0     Ana    100.0          400.0
1     Ana    300.0          400.0
2     Bob     50.0          100.0
3     Bob     50.0          100.0
```

---

## ⚡ 3. Casos de Uso Típicos

### ¿Cuándo usar `.agg()`?
1. **Reportes y Tablas Resumen:** Para exportar KPIs a Excel, BI o dashboards.
2. **Múltiples métricas simultáneas (Named Aggregations):**
   ```python
   kpis = df.groupby('pais', as_index=False).agg(
       total_ventas=('importe', 'sum'),
       ticket_medio=('importe', 'mean'),
       usuarios_unicos=('cliente_id', 'nunique')
   )
   ```

### ¿Cuándo usar `.transform()`?
1. **Cálculo de Ratios / Porcentajes relativos:**
   ```python
   df['total_grupo'] = df.groupby('categoria')['importe'].transform('sum')
   df['peso_en_categoria'] = df['importe'] / df['total_grupo']
   ```
2. **Diferencias / Desviaciones respecto al grupo:**
   ```python
   df['media_pais'] = df.groupby('pais')['importe'].transform('mean')
   df['diff_vs_media'] = df['importe'] - df['media_pais']
   ```
3. **Estandarización / Normalización por grupo (Z-score por grupo):**
   ```python
   mean = df.groupby('categoria')['importe'].transform('mean')
   std = df.groupby('categoria')['importe'].transform('std')
   df['zscore_categoria'] = (df['importe'] - mean) / std
   ```
4. **Imputación contextual de nulos (ej. rellenar con la media de su grupo):**
   ```python
   df['importe_imputado'] = df['importe'].fillna(
       df.groupby('categoria')['importe'].transform('median')
   )
   ```

---

## 🥊 4. Tabla Comparativa Resumen

| Característica | `.agg()` | `.transform()` |
| :--- | :--- | :--- |
| **Dimensión de salida** | 1 fila por grupo | Misma longitud que el DataFrame original |
| **Uso principal** | Crear tablas de resumen / reportes | Feature engineering / comparaciones relativas |
| **Equivalente SQL** | `GROUP BY` estándar | `WINDOW FUNCTION` (ej. `SUM(x) OVER(PARTITION BY grupo)`) |
| **Asignable a columna de `df`** | No directamente (requiere `merge`) | Sí, directamente: `df['nueva'] = ...` |
