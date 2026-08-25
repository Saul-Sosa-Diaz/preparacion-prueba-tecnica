# Visualización de Datos para Pruebas Técnicas (Matplotlib & Seaborn)

Guía práctica de visualización aplicada a pruebas técnicas de Data Analyst (Nivel Mid/Senior). En una prueba técnica no se evalúa hacer gráficos decorativos, sino la **claridad del mensaje, legibilidad inmediata, ausencia de saturación (*chart junk*) y rigor estadístico**.

---

## 1. Los 5 Gráficos Fundamentales en Entrevistas de Negocio

| Tipo de Pregunta / Caso de Negocio | Tipo de Gráfico | Método Recomendado |
| :--- | :--- | :--- |
| **Evolución temporal / Métricas YoY** | Gráfico de Líneas | `sns.lineplot()` |
| **Comparativa de categorías / Rankings** | Barras Horizontales | `sns.barplot()` / `plt.barh()` |
| **Distribución / Dispersión de valores** | Boxplot / Histograma + KDE | `sns.boxplot()` / `sns.histplot()` |
| **Correlación entre variables continuas** | Scatter Plot / Regresión | `sns.scatterplot()` / `sns.regplot()` |
| **Matriz de Cohortes / Retención** | Heatmap con anotaciones | `sns.heatmap()` |

---

## 2. Plantilla Base Profesional (Configuración Global)

Configura siempre el lienzo y el estilo al inicio del script o notebook para garantizar proporciones legibles y tipografía clara:

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np

# Configuración de estilo sobrio y profesional
sns.set_theme(style="whitegrid", palette="tab10")
plt.rcParams.update({
    'figure.figsize': (10, 5),
    'axes.titlesize': 14,
    'axes.titleweight': 'bold',
    'axes.labelsize': 11,
    'xtick.labelsize': 10,
    'ytick.labelsize': 10,
    'figure.autolayout': True  # Evita que se corten los textos de los ejes
})
```

---

## 3. Código y Patrones Listos para Entrevistas

### A. Gráfico de Barras Horizontal Ordenado (Comparativa / Top N)
> **Regla de oro:** Las barras horizontales facilitan la lectura de nombres de categorías largos sin necesidad de rotar el texto del eje X a 45° o 90°.

```python
# 1. Preparar datos ordenados
df_top = (
    df.groupby('categoria', as_index=False)['importe']
    .sum()
    .sort_values('importe', ascending=False)
    .head(10)
)

# 2. Renderizar gráfico
fig, ax = plt.subplots(figsize=(10, 5))
sns.barplot(data=df_top, x='importe', y='categoria', color='#2563eb', ax=ax)

# 3. Etiquetas de datos directas en cada barra
for p in ax.patches:
    width = p.get_width()
    ax.annotate(
        f'{width:,.0f}€', 
        (width, p.get_y() + p.get_height() / 2.),
        xytext=(6, 0), 
        textcoords='offset points', 
        va='center', 
        fontsize=9,
        weight='bold'
    )

ax.set_title("Top 10 Categorías por Facturación Total (2024)")
ax.set_xlabel("Facturación (€)")
ax.set_ylabel("")
sns.despine(left=True, bottom=True)
plt.show()
```

---

### B. Evolución Temporal con Línea de Tendencia / Comparativa de Grupos
> **Caso de uso:** Visualizar evolución mensual de ventas separadas por segmento (ej. Nuevos vs Recurrentes).

```python
fig, ax = plt.subplots(figsize=(11, 5))
sns.lineplot(
    data=df_mensual,
    x='fecha',
    y='importe',
    hue='segmento',
    marker='o',
    linewidth=2,
    ax=ax
)

ax.set_title("Evolución de Ingresos Mensuales por Segmento de Cliente")
ax.set_xlabel("Mes de Operación")
ax.set_ylabel("Facturación Neta (€)")
ax.legend(title="Segmento", frameon=True)
plt.xticks(rotation=0)
plt.show()
```

---

### C. Matriz de Retención / Cohortes (Heatmap)
> **Pregunta estrella:** Mostrar cómo cae la retención de usuarios a lo largo de los meses.

```python
# Suponiendo que 'cohort_matrix' es una tabla pivote con porcentajes (0.0 a 1.0)
fig, ax = plt.subplots(figsize=(12, 6))

sns.heatmap(
    cohort_matrix,
    annot=True,
    fmt='.1%',
    cmap='Blues',
    vmin=0.0,
    vmax=0.8,
    cbar_kws={'label': '% Retención'},
    linewidths=0.5,
    ax=ax
)

ax.set_title("Matriz de Retención por Cohortes Mensuales")
ax.set_xlabel("Meses transcurridos desde el registro")
ax.set_ylabel("Cohorte de Adquisición")
plt.show()
```

---

### D. Distribución y Detección de Outliers (Boxplot + Strip/Jitter)
> **Caso de uso:** Comparar la dispersión del ticket medio entre países.

```python
fig, ax = plt.subplots(figsize=(9, 5))

# Boxplot base
sns.boxplot(
    data=df, 
    x='pais', 
    y='importe', 
    showmeans=True, 
    meanprops={"marker":"o", "markerfacecolor":"white", "markeredgecolor":"black"},
    palette='Blues_r', 
    ax=ax
)

ax.set_title("Distribución de Importe de Compra por País (Punto blanco = Media)")
ax.set_xlabel("País")
ax.set_ylabel("Importe (€)")
plt.show()
```

---

## 4. Buenas vs. Malas Prácticas en Visualización

| Aspecto | ✓ Buena Práctica (Senior) | ✗ Mala Práctica (Junior) |
| :--- | :--- | :--- |
| **Gráficos de Tarta (Pie Charts)** | Usar gráficos de barras (horizontal o vertical). | Usar gráficos de tarta con más de 3 categorías. |
| **Ejes numéricos continuos** | Comenzar siempre el eje Y en 0 para evitar distorsiones visuales. | Cortar el eje Y artificialmente para exagerar diferencias mínimas. |
| **Textos en ejes** | Ordenar barras por valor y colocarlas horizontales para leer fácil. | Nombres en diagonal a 90° difíciles de leer. |
| **Uso del Color** | Usar colores intencionales (ej. destacar solo la barra líder en azul y el resto en gris). | Usar paletas arcoíris con 15 colores distintos sin justificación. |
| **Contexto** | Añadir unidades de medida (€, %, días) y títulos explicativos. | Títulos genéricos como *"Gráfico 1"* sin unidades en los ejes. |

---

## 5. El Patrón "Highlight" (Para Destacar en la Presentación Técnica)

En entrevistas técnicas para perfiles analíticos destaca mucho guiar la atención del evaluador hacia el hallazgo clave usando colores neutros y un único color de énfasis:

```python
# Resaltar solo la barra líder o anómala:
colores = ['#2563eb' if cat == 'Tech' else '#cbd5e1' for cat in df_top['categoria']]

fig, ax = plt.subplots(figsize=(10, 4))
sns.barplot(data=df_top, x='importe', y='categoria', palette=colores, ax=ax)
ax.set_title("Tech lidera con más del 40% del volumen total de ventas")
sns.despine()
plt.show()
```
