# 🧹 Employee Dataset Cleaning — MundoData

Limpieza de un dataset sintético de empleados (1,020 filas) usando un flujo de trabajo estructurado y repetible en Pandas. El objetivo no fue arreglar un archivo puntual, sino construir un **proceso reutilizable** para cualquier dataset "messy".

Proyecto: https://roadmap.sh/projects/clean-csv

## 📋 Descripción del proyecto

El dataset incluye problemas típicos de datos del mundo real: tipos de datos mal inferidos, fechas en formato de texto, valores nulos, columnas categóricas por estandarizar y un campo numérico con datos claramente inválidos. Este proyecto documenta cada decisión de limpieza tomada, priorizando la trazabilidad y evitando "inventar" datos que no se pueden recuperar.

**Dataset original:** [Messy Employee Dataset (Kaggle)](https://www.kaggle.com/datasets/desolution01/messy-employee-dataset)

## 🔁 Flujo de trabajo (5 etapas)

| Etapa | Descripción |
|---|---|
| **1. Load** | Inspección del archivo crudo como texto plano antes de cargarlo (delimitador, encoding) |
| **2. Inspect** | `.info()`, `.isnull().sum()`, `.value_counts()` para mapear tipos, nulos y categorías |
| **3. Clean** | Tipos de dato explícitos vía `dtype`, conversión de fechas con `pd.to_datetime(errors='coerce')`, estandarización de categóricas, auditoría de campos inválidos |
| **4. Review** | Comparación tipo por tipo (original vs. limpio) y verificación final de nulos y duplicados |
| **5. Export** | Exportación a CSV + relectura de verificación para confirmar integridad de los datos |

## 🔑 Decisiones clave de limpieza

- **`Age` → `Int64`** (no `int64`): pandas convertía automáticamente la columna a `float64` por la presencia de nulos (*silent type casting*). Se usó el tipo nullable `Int64` de pandas para mantener enteros reales sin forzar la eliminación de nulos.
- **`Join_Date` → `datetime64`**: convertida con `pd.to_datetime(format="%m/%d/%Y", errors='coerce')`. No se encontraron fechas inválidas (0 valores `NaT`).
- **Categóricas estandarizadas con cuidado**: se detectó que aplicar `.str.title()` de forma genérica rompía términos compuestos como `DevOps` (lo convertía en `Devops`). Se corrigió preservando la capitalización original donde correspondía.
- **`Phone` no se "corrigió" artificialmente**: el 100% de los valores son negativos y no representan teléfonos válidos. En vez de fabricar un dato plausible (ej. quitando el signo), se documentó como campo inválido y se agregó una columna auxiliar `Phone_Invalido` (booleana) para su fácil filtrado.
- **Nulos de `Age` (20.7%) y `Salary` (2.4%) no se imputaron**: la distribución de `Age` es discreta y pareja entre 4 valores posibles (25, 30, 35, 40); imputar con la mediana habría duplicado artificialmente una categoría, distorsionando el dataset. Se dejaron como nulos y quedaron documentados.
- **Los `dtype` no persisten en CSV**: al exportar y volver a leer el archivo sin especificar `dtype`, los tipos se pierden (`Age` vuelve a `float64`, `Phone` a `int64`, `Join_Date` a texto). Por eso el `dtype_map` usado en la carga queda documentado como parte del proceso, no solo el archivo final.

## 🛠️ Tecnologías usadas

- Python
- Pandas
- Jupyter Notebook (VS Code)

## 📁 Estructura del repositorio

```
├── Messy_Employee_dataset.csv     # Dataset original (crudo)
├── employee_cleaning.ipynb        # Notebook con el proceso completo
├── employees_clean.csv            # Dataset limpio, exportado
└── README.md
```

## ▶️ Cómo reproducirlo

```bash
pip install pandas notebook
jupyter notebook employee_cleaning.ipynb
```

**Importante:** para recargar `employees_clean.csv` con los tipos correctos, usar el mismo `dtype_map` definido en el notebook y volver a aplicar `pd.to_datetime()` sobre `Join_Date` — el CSV no conserva esos tipos por sí solo.

```python
dtype_map = {
    "Employee_ID": "str",
    "First_Name": "str",
    "Last_Name": "str",
    "Age": "Int64",
    "Department_Region": "str",
    "Status": "str",
    "Join_Date": "str",
    "Salary": "float64",
    "Email": "str",
    "Phone": "str",
    "Performance_Score": "str",
}

df = pd.read_csv("employees_clean.csv", dtype=dtype_map)
df["Join_Date"] = pd.to_datetime(df["Join_Date"], errors="coerce")
```

## 📈 Resultado

- ✅ 1,020 filas procesadas, 0 duplicados
- ✅ Tipos de dato corregidos y explícitos en toda las columnas
- ✅ Fechas convertidas a formato real, sin errores de parseo
- ✅ Categorías estandarizadas sin pérdida de información
- ✅ Datos inválidos documentados en lugar de fabricados
- ✅ Proceso de limpieza 100% reutilizable para otros datasets similares

## 🧠 Lo aprendido

Más allá del código, este proyecto reforzó ideas clave para el trabajo de un analista de datos:

- Los cambios de tipo silenciosos de pandas pueden pasar completamente desapercibidos si no se inspecciona explícitamente cada columna.
- Automatizar limpieza (como `.str.title()`) sin revisar el resultado puede introducir errores nuevos, no solo corregir los existentes.
- No todos los datos "problemáticos" deben corregirse: a veces la decisión correcta es documentar la limpieza, no imputarla.
