https://platform.stratascratch.com/coding/10352-users-by-avg-session-time
 
 Guía de Práctica para Entrevistas Técnicas: StrataScratch (Python / Pandas)

Esta recopilación contiene las preguntas clave y más representativas de **StrataScratch** para preparar entrevistas técnicas de Data Analyst en Python/Pandas, organizadas por patrón técnico y empresa.

---

## 📌 Preguntas Imprescindibles

| ID | Nombre de la Pregunta | Empresa | Dificultad | Patrón Técnico Evaluado |
| :--- | :--- | :--- | :--- | :--- |
| **#10308** | *Salaries Differences* | Dropbox | Easy / Medium | Filtrado condicional, manejo de nulos y cálculo escalar. |
| **#10353** | *Workers With The Highest Salaries* | Amazon | Medium | Joins, ordenación y filtrado dinámico por valor máximo (sin hardcodear). |
| **#9913** | *Order Details* | Amazon | Easy / Medium | Merges multi-tabla y agregaciones con condiciones (`groupby` + `isin`). |
| **#10049** | *Reviews of Categories* | Yelp | Medium | Manejo y separación de cadenas/listas (`str.split()`, `explode()`) + conteo. |
| **#9894** | *Employee and Manager Salaries* | Uber | Medium | Self-joins en Pandas (`pd.merge` sobre el mismo DataFrame) y filtros booleanos. |
| **#10319** | *Top Streamers* | Twitch | Medium | Funciones de ranking (`rank(method='dense')`, `head()`) agrupadas. |
| **#9782** | *Customer Revenue In March* | Meta / Facebook | Medium | Parseo de fechas, extracción de componentes temporales y agregación. |
| **#10078** | *Host Popularity Rental Prices* | Airbnb | Medium / Hard | Transformaciones complejas, segmentación numérica (`pd.cut`) y medias agrupadas. |

---

## 🗺️ Hoja de Ruta Sugerida

### 1. Bloque 1: Transformaciones Básicas y Joins
- **#10308** (*Salaries Differences*)
- **#10353** (*Workers With The Highest Salaries*)
- **#9913** (*Order Details*)

*Objetivo:* Calentar con joins simples, agregaciones básicas y sintaxis vectorizada limpia sin recurrir a bucles `for`.

### 2. Bloque 2: Lógica Relacional, Fechas y Strings
- **#9894** (*Employee and Manager Salaries*)
- **#9782** (*Customer Revenue In March*)
- **#10049** (*Reviews of Categories*)

*Objetivo:* Dominar cruces relacionales complejos (self-joins), manejo de accesores vectorizados (`.dt` y `.str`) y desanidado de datos con `.explode()`.

### 3. Bloque 3: Ventanas, Segmentaciones y Rankings
- **#10319** (*Top Streamers*)
- **#10078** (*Host Popularity Rental Prices*)

*Objetivo:* Aplicar funciones de ventana en Pandas (`rank()`, `transform()`, `shift()`) y agrupaciones por intervalos con `pd.cut()` / `pd.qcut()`.
