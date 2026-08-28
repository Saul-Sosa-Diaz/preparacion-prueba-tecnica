https://platform.stratascratch.com/coding/10352-users-by-avg-session-time
```python
# Import your libraries
import pandas as pd

# Start writing code
facebook_web_log.head()

facebook_web_log["date"] = facebook_web_log["timestamp"].dt.date

loads = (
    facebook_web_log[facebook_web_log["action"] == "page_load"]
    .groupby(["user_id", "date"])["timestamp"]
    .max()
    .reset_index(name="latest_page_load")
)

exits = (
    facebook_web_log[facebook_web_log["action"] == "page_exit"]
    .groupby(["user_id", "date"])["timestamp"]
    .min()
    .reset_index(name="earliest_page_exit")
)

sessions = pd.merge(loads, exits, on=["user_id", "date"], how="inner")

valid_sessions = sessions[
    sessions["latest_page_load"] < sessions["earliest_page_exit"]
].copy()

valid_sessions["session_duration"] = (
    valid_sessions["earliest_page_exit"] - valid_sessions["latest_page_load"]
).dt.total_seconds()

result = (
    valid_sessions.groupby("user_id")["session_duration"]
    .mean()
    .reset_index(name="session_time")
)
```
---

https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date
```python
# Import your libraries
import pandas as pd
import numpy as np

# Start writing code
fb_friend_requests
accepted = fb_friend_requests[fb_friend_requests["action"]=="accepted"]

sended = fb_friend_requests[fb_friend_requests["action"]=="sent"]

tre = pd.merge(sended, accepted, on=["user_id_sender", "user_id_receiver"], how="left")

tre['bin'] = np.where(tre['action_y'].isna(), 0, 1)
tre = tre[tre["bin"] > 0]

tre.groupby("date_x")["bin"].mean().reset_index()
```
---
https://platform.stratascratch.com/coding/10322-finding-user-purchases
```python
# Import your libraries
import pandas as pd
import numpy as np

# Start writing code
amazon_transactions.head()

amazon_transactions = amazon_transactions.sort_values(['user_id', "created_at"])
amazon_transactions = amazon_transactions.drop_duplicates(["user_id", "created_at"])
amazon_transactions  = amazon_transactions.groupby('user_id').head(2).reset_index()
amazon_transactions['prev_created_at'] = amazon_transactions.groupby('user_id')['created_at'].shift(1) # desplaza los valores 

amazon_transactions = amazon_transactions[((amazon_transactions['created_at'] - amazon_transactions['prev_created_at']).dt.days <= 7 ) & (amazon_transactions['prev_created_at'].notna())]

amazon_transactions['user_id'].to_list()
```
---
https://platform.stratascratch.com/coding/10304-risky-projects
```python
# Import your libraries
import pandas as pd
import math
import numpy as np

# Start writing code
linkedin_projects.head()

# Primero sacar el número de meses de un proyecto
linkedin_projects["Days_of_proyect"] = (linkedin_projects["end_date"] - linkedin_projects["start_date"]).dt.days

linkedin_projects

# Ahora sacar que Gente trabaja en que proyecto 
linkedin_employees.rename(columns={'id': 'emp_id'}, inplace=True)
linkedin_projects.rename(columns={'id': 'project_id'}, inplace=True)
tmp = linkedin_emp_projects.merge(linkedin_employees, on="emp_id")

projects_with_emp = linkedin_projects.merge(tmp, on="project_id")

projects_with_emp

# Calcular el precio de la gente trabajando
# 365 -> salary
# Days_of_project -> X
# X = (Days_of_project * Salary) / 365 
projects_with_emp["prorated_expense"] = (projects_with_emp["Days_of_proyect"] * projects_with_emp["salary"]) / 365

# Ahora sumar por proyecto
final = projects_with_emp.groupby(["project_id","title","budget"])["prorated_expense"].agg("sum").reset_index()
final["prorated_expense"] = np.ceil(final["prorated_expense"])
final = final[final["budget"] < final["prorated_expense"]]

final.drop(["project_id"], axis=1)
```
---
https://platform.stratascratch.com/coding/10159-ranking-most-active-guests
```python
# Import your libraries
import pandas as pd
import numpy as np
#Mi solucion pocha
# Start writing code
airbnb_contacts.head()

messages_by_guests = airbnb_contacts.groupby("id_guest")["n_messages"].sum().reset_index()
messages_by_guests.sort_values(["n_messages"], inplace=True, ascending=False)

messages_by_guests["rank"] = np.arange(1, len(messages_by_guests) + 1)

max_numbermessage = messages_by_guests["n_messages"].max()

rank = 1
def rank_control(prev, curr):
   global rank
   if prev == curr:
     return rank
   rank = rank + 1
   return rank

messages_by_guests["prev_num_message_guest"] = messages_by_guests.shift(1,fill_value=max_numbermessage)["n_messages"]

messages_by_guests["ranking"] = np.vectorize(rank_control)(messages_by_guests["prev_num_message_guest"], messages_by_guests["n_messages"])


messages_by_guests = messages_by_guests[["ranking", "id_guest", "n_messages"]]

messages_by_guests.rename(columns={"n_messages": "sum_n_messages"}, inplace=True)

messages_by_guests

# La solucion chachi
import pandas as pd

# 1. Agrupar y sumar mensajes por huésped
result = (
    airbnb_contacts.groupby("id_guest", as_index=False)["n_messages"]
    .sum()
)

# 2. Calcular el ranking denso (sin saltos de número en empates)
result["ranking"] = result["n_messages"].rank(method="dense", ascending=False).astype(int)

# 3. Ordenar y seleccionar columnas requeridas
result = (
    result.sort_values(by=["ranking", "id_guest"])
    [["ranking", "id_guest", "n_messages"]]
)
```
---
https://platform.stratascratch.com/coding/10156-number-of-units-per-nationality

```python
# Import your libraries
import pandas as pd

# Start writing code
airbnb_hosts.head()

# Me quedo con la peña más pequeña de 30
airbnb_hosts = airbnb_hosts[airbnb_hosts["age"] < 30]

# Filtrar solo los apartamentos
airbnb_units = airbnb_units[airbnb_units["unit_type"]=="Apartment"]

# Ahora hay que hacer un merge
hosts_filtereds_by_ages_with_apartaments_types = airbnb_hosts.merge(airbnb_units, on="host_id", how='inner')


young_hosts_apartment_counts = (
    hosts_filtereds_by_ages_with_apartaments_types.groupby(
        ["host_id", "nationality"]
    )["unit_id"]
    .nunique()
    .reset_index(name="apartment_count")
)

young_hosts_apartment_counts[["nationality", "apartment_count"]]

```
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
