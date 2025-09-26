# 🌸 SQL x Python — Guía Completa (Partes 0–7) ✦ Compendio

_Shingeki del conocimiento: Emoción → Acción → Cuidado. Kawaii, travieso y profesional._

> Este compendio integra **todas** las partes en un solo documento. Los canvases originales quedan intactos.

---

## Índice vivo

- ✅ **Parte 0 — Setup y primer DB** (SQLite + Python)
- ✅ **Parte 1 — SQL básico** (CREATE, INSERT, SELECT, WHERE, ORDER, LIMIT)
- ✅ **Parte 2 — Pandas + SQL** (read_sql / to_sql / merge / export)
- ✅ **Parte 3 — JOIN, GROUP BY, HAVING** (+ ventanas)
- ✅ **Parte 4 — Índices, EXPLAIN y performance**
- ✅ **Parte 5 — SQLAlchemy Core y ORM**
- ✅ **Parte 6 — Postgres & MySQL (drivers y conexión)**
- ✅ **Parte 7 — Mini‑proyecto final** (CSV → DB → SQLAlchemy → pandas → reporte)

> Requisitos: Python 3.9+, editor (VS Code/Jupyter). `sqlite3` viene con Python. Opcional: `pandas`, `SQLAlchemy`, drivers Postgres/MySQL.

---

# Parte 0 — Setup y primer DB (SQLite + Python)

### 🎯 Objetivo

Crear `data/kawaii.db`, tabla `estudiantes`, insertar datos y consultar desde Python.

### 🧰 Entorno y librerías (opcional)

```bash
python -m venv .venv
# Activar:  Windows: .\.venv\Scripts\Activate.ps1   |   macOS/Linux: source .venv/bin/activate
pip install --upgrade pip
pip install pandas SQLAlchemy

```

### 📁 Estructura sugerida

```
proyecto-sql-kawaii/
├─ data/
├─ setup_sqlite.py
├─ query_sqlite.py
└─ query_pandas.py (opcional)

```

### 🐣 Crear DB y tabla + inserts (`setup_sqlite.py`)

```python
import sqlite3
from pathlib import Path

DB_PATH = Path("data") / "kawaii.db"
DB_PATH.parent.mkdir(parents=True, exist_ok=True)

conn = sqlite3.connect(DB_PATH)
cur = conn.cursor()

cur.execute("""
CREATE TABLE IF NOT EXISTS estudiantes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    curso TEXT NOT NULL,
    nota REAL CHECK (nota BETWEEN 0 AND 10)
);
""")

cur.executemany(
    "INSERT INTO estudiantes (nombre, curso, nota) VALUES (?, ?, ?);",
    [
        ("Miyu", "SQL 101", 9.5),
        ("Kora", "SQL 101", 8.7),
        ("Lyra", "Python", 9.1),
        ("Nia", "Python", 7.9),
    ]
)

conn.commit()
print("DB lista en:", DB_PATH)
conn.close()

```

### 🔎 Consultas mínimas (`query_sqlite.py`)

```python
import sqlite3
from pathlib import Path

DB_PATH = Path("data") / "kawaii.db"

def fetch_all(query: str, params: tuple = ()):
    with sqlite3.connect(DB_PATH) as conn:
        conn.row_factory = sqlite3.Row
        cur = conn.execute(query, params)
        return [dict(r) for r in cur.fetchall()]

print("Todos:")
for r in fetch_all("SELECT id, nombre, curso, nota FROM estudiantes;"):
    print(r)

print("\nSolo curso=Python ordenado por nota:")
for r in fetch_all(
    "SELECT nombre, nota FROM estudiantes WHERE curso = ? ORDER BY nota DESC;",
    ("Python",),
):
    print(r)

```

### 🧪 Mini‑retos 0

1.  `email TEXT UNIQUE` y actualizar filas. 2) 20 estudiantes aleatorias + filtro `nota >= 9`. 3) `SELECT DISTINCT curso`. 4) Exportar CSV con `pandas`.

---

# Parte 1 — SQL básico ✦

### 🎯 Objetivo

Practicar **CREATE**, **INSERT**, **SELECT**, **WHERE**, **ORDER BY**, **LIMIT** con `cursos`.

### 🗂️ Crear tabla

```sql
CREATE TABLE cursos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  nombre TEXT NOT NULL,
  descripcion TEXT,
  duracion_horas INTEGER CHECK (duracion_horas > 0)
);

```

### 🐣 Insertar

```sql
INSERT INTO cursos (nombre, descripcion, duracion_horas) VALUES
  ("SQL 101", "Intro kawaii a bases de datos", 20),
  ("Python", "Aprender scripting útil", 30),
  ("Data Science", "Magia de pandas y numpy", 40);

```

### 🔎 Consultar / WHERE / ORDER / LIMIT

```sql
SELECT id, nombre, duracion_horas FROM cursos;
SELECT * FROM cursos WHERE duracion_horas >= 30;
SELECT * FROM cursos ORDER BY duracion_horas DESC;
SELECT * FROM cursos LIMIT 2;

```

### 🐍 Python

```python
import sqlite3
from pathlib import Path
DB_PATH = Path("data") / "kawaii.db"

with sqlite3.connect(DB_PATH) as conn:
    cur = conn.cursor()
    cur.execute("""
    CREATE TABLE IF NOT EXISTS cursos (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      nombre TEXT NOT NULL,
      descripcion TEXT,
      duracion_horas INTEGER CHECK (duracion_horas > 0)
    );
    """)

    cur.executemany(
      "INSERT INTO cursos (nombre, descripcion, duracion_horas) VALUES (?, ?, ?);",
      [
        ("SQL 101", "Intro kawaii a bases de datos", 20),
        ("Python", "Aprender scripting útil", 30),
        ("Data Science", "Magia de pandas y numpy", 40),
      ]
    )
    conn.commit()

    rows = cur.execute(
      "SELECT nombre, duracion_horas FROM cursos ORDER BY duracion_horas DESC LIMIT 2;"
    ).fetchall()
    print("Top 2 cursos más largos:")
    for r in rows:
        print(r)

```

### 🧪 Mini‑retos 1

1.  `Machine Learning`, 60 h.
2.  2.  `LIKE '%SQL%'`.
3.  3.  Orden por `nombre`.
4.  4.  Menor duración con `ASC LIMIT 1`.

---

# Parte 2 — Pandas + SQL ✦

### 🎯 Objetivo

Leer SQL a `DataFrame`, escribir `DataFrame` a tablas y unir con SQL/pandas.

### `read_sql_query` y parámetros

```python
import sqlite3, pandas as pd
from pathlib import Path
DB_PATH = Path("data") / "kawaii.db"

with sqlite3.connect(DB_PATH) as conn:
    df = pd.read_sql_query(
        "SELECT nombre, duracion_horas FROM cursos ORDER BY duracion_horas DESC;",
        conn
    )
print(df.head())

query = "SELECT * FROM cursos WHERE duracion_horas >= ? AND nombre LIKE ? ORDER BY duracion_horas DESC;"
params = (30, "%SQL%")
with sqlite3.connect(DB_PATH) as conn:
    df = pd.read_sql_query(query, conn, params=params)
print(df)

```

### `to_sql` y JOIN vs `merge`

```python
import numpy as np, pandas as pd, sqlite3
from pathlib import Path
DB_PATH = Path("data") / "kawaii.db"

nuevos = pd.DataFrame({
  "nombre": ["Miyu","Kora","Lyra","Nia","Sibel"],
  "curso": ["Data Science","Data Science","SQL 101","Python","SQL 101"],
  "nota": np.round(np.random.uniform(6.0, 10.0, 5), 1)
})

with sqlite3.connect(DB_PATH) as conn:
  nuevos.to_sql("estudiantes", conn, if_exists="append", index=False)

```

```python
sql = """
SELECT e.nombre, e.nota, c.nombre AS curso_nombre, c.duracion_horas
FROM estudiantes e
JOIN cursos c ON e.curso = c.nombre
ORDER BY e.nota DESC;
"""
with sqlite3.connect(DB_PATH) as conn:
    df_join = pd.read_sql_query(sql, conn)

with sqlite3.connect(DB_PATH) as conn:
    df_e = pd.read_sql_query("SELECT nombre, curso, nota FROM estudiantes;", conn)
    df_c = pd.read_sql_query("SELECT nombre AS curso, duracion_horas FROM cursos;", conn)
merged = df_e.merge(df_c, on="curso", how="left")

```

### Exportar

```python
merged.to_csv("data/reporte_estudiantes.csv", index=False)
merged.to_parquet("data/reporte_estudiantes.parquet", index=False)

```

### 🧪 Mini‑retos 2

1.  100 estudiantes y `append`. 2) `groupby` promedio por curso. 3) Parquet vs CSV. 4) `nota >= 9` con parámetros. 5) `LIKE 'Data%'`.

---

# Parte 3 — JOIN, GROUP BY, HAVING ✦ (+ ventanas)

### JOINs y agregaciones

```sql
SELECT e.nombre, e.nota, c.nombre AS curso, c.duracion_horas
FROM estudiantes e
INNER JOIN cursos c ON e.curso = c.nombre
ORDER BY e.nota DESC;

SELECT e.nombre, e.curso, c.duracion_horas
FROM estudiantes e
LEFT JOIN cursos c ON e.curso = c.nombre;

SELECT e.curso, ROUND(AVG(e.nota), 2) AS nota_prom
FROM estudiantes e
GROUP BY e.curso
ORDER BY nota_prom DESC;

SELECT e.curso, AVG(e.nota) AS nota_prom, COUNT(*) AS n
FROM estudiantes e
GROUP BY e.curso
HAVING COUNT(*) >= 3 AND AVG(e.nota) >= 8.5
ORDER BY nota_prom DESC;

```

### Ventanas (SQLite)

```sql
SELECT
  e.nombre,
  e.curso,
  e.nota,
  ROUND(AVG(e.nota) OVER (PARTITION BY e.curso), 2) AS prom_curso,
  ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
FROM estudiantes e
ORDER BY e.curso, rn;

```

### 🧪 Mini‑retos 3

1.  `LEFT JOIN` para `IS NULL`. 2) `HAVING n >= 5`. 3) Top1 por curso con `ROW_NUMBER()`. 4) `DENSE_RANK()` global. 5) Reporte CSV `curso, n, nota_prom, top1_nombre, top1_nota`.

---

# Parte 4 — Índices, EXPLAIN y performance ✦

### Índices

```sql
CREATE INDEX idx_estudiantes_curso ON estudiantes(curso);
CREATE INDEX idx_estudiantes_curso_nota ON estudiantes(curso, nota);
DROP INDEX idx_estudiantes_curso;

```

### EXPLAIN y medición

```sql
EXPLAIN QUERY PLAN
SELECT * FROM estudiantes WHERE curso = 'Python' AND nota >= 9;

```

```python
import sqlite3, time
from pathlib import Path
DB_PATH = Path("data") / "kawaii.db"

query = "SELECT * FROM estudiantes WHERE curso = ? AND nota >= ?;"
params = ("Python", 9)
with sqlite3.connect(DB_PATH) as conn:
    t0 = time.time()
    for _ in range(1000):
        conn.execute(query, params).fetchall()
    t1 = time.time()
print("Tiempo total:", round(t1 - t0, 3), "s")

```

### 🧪 Mini‑retos 4

1.  Índice en `cursos(nombre)` y `LIKE 'SQL%'`. 2) Ver uso con `EXPLAIN`. 3) Medir tiempos con/sin índice. 4) Índice único `estudiantes(email)`. 5) Índice compuesto `curso, nota DESC`.

---

# Parte 5 — SQLAlchemy Core y ORM ✦

### Core: tablas, insert, select

```python
from sqlalchemy import create_engine, Table, Column, Integer, String, Float, MetaData, insert, select
engine = create_engine("sqlite:///data/kawaii.db", echo=True)
metadata = MetaData()

estudiantes = Table(
    "estudiantes", metadata,
    Column("id", Integer, primary_key=True),
    Column("nombre", String, nullable=False),
    Column("curso", String, nullable=False),
    Column("nota", Float),
)
metadata.create_all(engine)

with engine.begin() as conn:
    conn.execute(insert(estudiantes), [
        {"nombre":"Nova","curso":"Python","nota":9.8},
        {"nombre":"Sibel","curso":"SQL 101","nota":8.9},
    ])

with engine.connect() as conn:
    for row in conn.execute(select(estudiantes.c.nombre, estudiantes.c.nota)):
        print(row)

```

### ORM: modelo, CRUD

```python
from sqlalchemy.orm import declarative_base, Session
from sqlalchemy import Column, Integer, String, Float

Base = declarative_base()
class Estudiante(Base):
    __tablename__ = "estudiantes"
    id = Column(Integer, primary_key=True)
    nombre = Column(String, nullable=False)
    curso = Column(String, nullable=False)
    nota = Column(Float)

Base.metadata.create_all(engine)
session = Session(engine)

session.add_all([
    Estudiante(nombre="Miyu", curso="Data Science", nota=9.7),
    Estudiante(nombre="Kora", curso="Python", nota=8.8),
])
session.commit()

for est in session.query(Estudiante).filter(Estudiante.nota >= 9).order_by(Estudiante.nota.desc()):
    print(est)

```

### 🧪 Mini‑retos 5

1.  Clase `Curso` y relación 1‑N. 2) Query estudiantes de `SQL 101`. 3) +0.5 a notas de `Python`. 4) Borrar `< 7`. 5) Exportar con pandas desde `session.query()`.

---

# Parte 6 — Postgres & MySQL ✦

### Drivers e `engine`

```bash
pip install psycopg2-binary PyMySQL

```

```python
from sqlalchemy import create_engine
engine_pg = create_engine("postgresql+psycopg2://usuario:password@localhost:5432/mibase", echo=True)
engine_mysql = create_engine("mysql+pymysql://usuario:password@localhost:3306/mibase", echo=True)

```

### Core / ORM (ejemplo Postgres)

```python
from sqlalchemy import Table, Column, Integer, String, MetaData, insert, select
metadata = MetaData()

cursos = Table(
    "cursos", metadata,
    Column("id", Integer, primary_key=True),
    Column("nombre", String(100), nullable=False),
    Column("descripcion", String(255))
)
metadata.create_all(engine_pg)
with engine_pg.begin() as conn:
    conn.execute(insert(cursos), [
        {"nombre": "SQL avanzado", "descripcion": "joins, índices, tuning"},
        {"nombre": "Python avanzado", "descripcion": "async, APIs, testing"},
    ])
with engine_pg.connect() as conn:
    for row in conn.execute(select(cursos)):
        print(row)

```

### Tips productivos

- Variables de entorno / `.env` para credenciales.
- Migraciones: **Alembic**.
- Pool de conexiones gestionado por SQLAlchemy.
- Paginación y `LIMIT` en consultas grandes.

### 🧪 Mini‑retos 6

1.  Tabla `estudiantes` en Postgres con ORM. 2) `to_sql(engine_pg)` desde CSV. 3) Top 5 cursos en MySQL por nombre. 4) `Session` update + `commit()`. 5) Credenciales con `os.getenv`.

---

# Parte 7 — Mini‑proyecto final ✦

### Dataset

`estudiantes.csv`

```csv
nombre,curso,nota
Miyu,SQL 101,9.5
Kora,Python,8.8
Lyra,Data Science,9.1
Nia,SQL 101,7.9
Nova,Python,9.8

```

### Ingesta CSV → DB

```python
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine("sqlite:///data/kawaii.db")
df = pd.read_csv("estudiantes.csv")
df.to_sql("estudiantes", engine, if_exists="replace", index=False)

```

### Core + pandas

```python
from sqlalchemy import Table, MetaData, select
metadata = MetaData(); metadata.reflect(bind=engine)
estudiantes = metadata.tables["estudiantes"]
with engine.connect() as conn:
    result = conn.execute(
        select(estudiantes.c.curso, estudiantes.c.nota)
        .where(estudiantes.c.nota >= 9)
        .order_by(estudiantes.c.nota.desc())
    )
    for row in result: print(row)

import pandas as pd
with engine.connect() as conn:
    df = pd.read_sql_query(
        "SELECT curso, AVG(nota) AS prom, COUNT(*) AS n FROM estudiantes GROUP BY curso;",
        conn
    )
print(df)
df.to_csv("reporte_final.csv", index=False)

```

### Bonus FastAPI (opcional)

```bash
pip install fastapi uvicorn

```

```python
from fastapi import FastAPI
import pandas as pd
app = FastAPI()
@app.get("/reporte")
def get_reporte():
    df = pd.read_sql_query(
        "SELECT curso, AVG(nota) AS prom, COUNT(*) AS n FROM estudiantes GROUP BY curso;",
        engine
    )
    return df.to_dict(orient="records")

```

# 🌸 Parte 7 — Ampliación Extendida ✦ (Proyecto Final)

Más poder para tu demo: columnas nuevas con **unicidad**, tabla `cursos` relacionada, **append** desde otro CSV, y **rankings** con funciones ventana. Todo listo para pegar a tu proyecto.

---

## A) Agregar columna `email` con **unicidad** y validar datos

```python
import sqlite3
from pathlib import Path
DB = Path("data")/"kawaii.db"

with sqlite3.connect(DB) as conn:
    cur = conn.cursor()
    # 1) Agregar columna si no existiera (SQLite no tiene IF NOT EXISTS por columna; probamos y si falla, ignoramos)
    try:
        cur.execute("ALTER TABLE estudiantes ADD COLUMN email TEXT;")
    except sqlite3.OperationalError:
        pass  # ya existe

    # 2) Crear índice único para asegurar que no se repitan emails
    cur.execute("CREATE UNIQUE INDEX IF NOT EXISTS idx_estudiantes_email ON estudiantes(email);")
    conn.commit()

    # 3) Cargar emails de ejemplo de forma segura
    seeds = [
        ("Miyu","miyu@example.com"),
        ("Kora","kora@example.com"),
        ("Lyra","lyra@example.com"),
        ("Nia","nia@example.com"),
        ("Nova","nova@example.com"),
    ]
    for nombre, email in seeds:
        cur.execute(
            "UPDATE estudiantes SET email = ? WHERE nombre = ? AND email IS NULL;",
            (email, nombre)
        )
    conn.commit()

    # 4) Probar unicidad (este debe fallar si ya existe ese email)
    try:
        cur.execute("INSERT INTO estudiantes(nombre, curso, nota, email) VALUES(?, ?, ?, ?);",
                    ("Zoe","Python",9.0,"miyu@example.com"))
        conn.commit()
    except sqlite3.IntegrityError as e:
        print("Unicidad OK:", e)

```

---

## B) Crear tabla `cursos` y **relacionar** con `estudiantes`

### 1) Esquema y carga de `cursos`

```python
import sqlite3
from pathlib import Path
DB = Path("data")/"kawaii.db"

with sqlite3.connect(DB) as conn:
    cur = conn.cursor()
    cur.execute(
        """
        CREATE TABLE IF NOT EXISTS cursos (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nombre TEXT UNIQUE NOT NULL,
            duracion_horas INTEGER CHECK (duracion_horas > 0)
        );
        """
    )
    cur.executemany(
        "INSERT OR IGNORE INTO cursos(nombre, duracion_horas) VALUES(?, ?);",
        [
            ("SQL 101", 20),
            ("Python", 30),
            ("Data Science", 40),
            ("Machine Learning", 60),
        ]
    )
    conn.commit()

```

### 2) **JOIN** para enriquecer reportes

```python
import pandas as pd
with sqlite3.connect(DB) as conn:
    df = pd.read_sql_query(
        """
        SELECT e.nombre, e.curso, e.nota, c.duracion_horas
        FROM estudiantes e
        LEFT JOIN cursos c ON c.nombre = e.curso
        ORDER BY e.curso, e.nota DESC;
        """,
        conn
    )
print(df.head())

```

---

## C) **Append** desde otro CSV (ingesta incremental)

Supongamos `estudiantes_extra.csv` con mismas columnas (`nombre,curso,nota[,email]`).

```python
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine("sqlite:///data/kawaii.db")

extra = pd.read_csv("estudiantes_extra.csv")
# Validación mínima: evita duplicar emails ya existentes (si tienes email)
if "email" in extra.columns:
    extra = extra.drop_duplicates(subset=["email"])  # en el batch

# Cargar en modo append
extra.to_sql("estudiantes", engine, if_exists="append", index=False)
print("Append listo: +", len(extra), "filas")

```

> Tip: si tenés índice único en `email`, los duplicados exactos fallarán al insertar. Podés atraparlos con `try/except` y loguearlos.

---

## D) **Rankings** por curso con funciones ventana (top N)

Top 3 estudiantes por curso, usando ventanas de SQLite y export a CSV.

```python
import pandas as pd, sqlite3
from pathlib import Path
DB = Path("data")/"kawaii.db"

sql_top = """
SELECT nombre, curso, nota
FROM (
  SELECT e.*, ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
  FROM estudiantes e
) t
WHERE rn <= 3
ORDER BY curso, rn;
"""

with sqlite3.connect(DB) as conn:
    top3 = pd.read_sql_query(sql_top, conn)

print(top3)
top3.to_csv("data/top3_por_curso.csv", index=False)

```

---

## E) **Endpoints** extra en FastAPI (ranking + detalles)

Agregá al archivo `api.py`:

```python
from fastapi import FastAPI
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("sqlite:///data/kawaii.db")
app = FastAPI()

@app.get("/reporte")
def get_reporte():
    df = pd.read_sql_query(
        "SELECT curso, AVG(nota) AS prom, COUNT(*) AS n FROM estudiantes GROUP BY curso;",
        engine
    )
    return df.to_dict(orient="records")

@app.get("/top/{n}")
def get_top(n: int = 3):
    sql = f"""
    SELECT nombre, curso, nota FROM (
      SELECT e.*, ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
      FROM estudiantes e
    ) t WHERE rn <= {n}
    ORDER BY curso, rn;
    """
    df = pd.read_sql_query(sql, engine)
    return df.to_dict(orient="records")

@app.get("/curso/{nombre}")
def detalle_curso(nombre: str):
    q = """
    SELECT e.nombre, e.nota, c.duracion_horas
    FROM estudiantes e
    LEFT JOIN cursos c ON c.nombre = e.curso
    WHERE e.curso = ?
    ORDER BY e.nota DESC;
    """
    df = pd.read_sql_query(q, engine, params=(nombre,))
    return df.to_dict(orient="records")

```

Correr:

```bash
uvicorn api:app --reload --port 8000

```

- `GET /top/3` → top 3 por curso.
- `GET /curso/Python` → detalle con `duracion_horas` desde `cursos`.

---

## F) **Checklist** de ampliación ✦

- `email` agregado y protegido con **índice único**.
- Tabla `cursos` creada y **JOIN** funcionando.
- Ingesta incremental por **append** desde CSV extra.
- **Top N** por curso con funciones ventana y export a CSV.
- **API** extendida: `/top/{n}` y `/curso/{nombre}`.

✨ Con esta ampliación, tu mini‑proyecto se convierte en un **micro‑stack de analítica** listo para demo: data in → SQL/ventanas → pandas → export → API ✦

---

# 🌸 Parte 7 — Anexo CLI ✦ Consumir la API desde la línea de comandos

Guía rápida para probar los endpoints del mini‑proyecto con **curl**, **HTTPie** y herramientas útiles como **jq**. Incluye ejemplos para Linux/macOS y Windows PowerShell.

> Asegurate de tener corriendo la API: `uvicorn api:app --reload --port 8000` → `http://127.0.0.1:8000`.

---

## 0) Herramientas opcionales

- **curl**: suele venir por defecto (Linux/macOS). En Windows 10+ también está.
- **HTTPie**: `pip install httpie` (comando `http`).
- **jq** (formatear JSON): macOS `brew install jq` / Ubuntu `sudo apt-get install jq` / Windows: `choco install jq`.

---

## 1) Endpoint `/reporte` — Promedio por curso

### curl

```bash
curl -s http://127.0.0.1:8000/reporte | jq

```

Guardar a archivo:

```bash
curl -s http://127.0.0.1:8000/reporte -o data/reporte_api.json

```

### HTTPie

```bash
http GET :8000/reporte | jq

```

### PowerShell (Windows)

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:8000/reporte" | ConvertTo-Json -Depth 5
# Guardar:
Invoke-RestMethod -Uri "http://127.0.0.1:8000/reporte" | ConvertTo-Json -Depth 5 | Out-File data\reporte_api.json

```

---

## 2) Endpoint `/top/{n}` — Top N por curso

### curl

```bash
N=3
curl -s http://127.0.0.1:8000/top/$N | jq

```

### HTTPie

```bash
http GET :8000/top/5 | jq

```

### PowerShell

```powershell
$N = 3
Invoke-RestMethod -Uri "http://127.0.0.1:8000/top/$N" | ConvertTo-Json -Depth 5

```

---

## 3) Endpoint `/curso/{nombre}` — Detalle por curso

### curl

```bash
curl -s "http://127.0.0.1:8000/curso/Python" | jq

```

> Si el nombre tiene espacios: `SQL%20101` (URL encoded) o comillas: `"SQL 101"`.

```bash
curl -s "http://127.0.0.1:8000/curso/SQL%20101" | jq

```

### HTTPie

```bash
http GET :8000/curso/Python | jq
http GET :8000/curso/"SQL 101" | jq

```

### PowerShell

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:8000/curso/Python" | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri "http://127.0.0.1:8000/curso/SQL%20101" | ConvertTo-Json -Depth 5

```

---

## 4) Pipes útiles (exportar a CSV/NDJSON desde CLI)

### A CSV simple (nombre;nota) desde `/curso/Python`

```bash
curl -s :8000/curso/Python | jq -r '.[] | "\(.nombre),\(.nota)"' > data/curso_python.csv

```

### A NDJSON (una línea por registro)

```bash
curl -s :8000/top/3 | jq -c '.[]' > data/top3.ndjson

```

---

## 5) Test rápidos (asserts en shell)

### Verificar que `/reporte` devuelve al menos 1 curso

```bash
[ $(curl -s :8000/reporte | jq 'length') -ge 1 ] && echo OK || echo FAIL

```

### Asegurar que `/top/3` trae como máximo 3 por curso

_(simple check de estructura: cuenta total y confía en la query; para validación estricta habría que agrupar por curso con jq)_

```bash
TOTAL=$(curl -s :8000/top/3 | jq 'length')
[ $TOTAL -ge 1 ] && echo OK || echo FAIL

```

---

## 6) Headers, compresión y verbose

### Ver cabeceras

```bash
curl -I :8000/reporte

```

### Modo verbose

```bash
curl -v :8000/reporte

```

### Comprimir respuesta (si se habilita gzip)

```bash
curl -H 'Accept-Encoding: gzip' --compressed :8000/reporte | jq

```

---

## 7) Tips de producción

- Usá **variables de entorno** para la URL base (`API_URL=http://127.0.0.1:8000`).
- Probá **time** para medir: `time curl -s $API_URL/reporte > /dev/null`.
- Documentá ejemplos reales en un `README.md` con comandos listos para copiar/pegar.

---

✨ Con este anexo ya podés **testear y scrapear** tus endpoints desde Terminal como una gata pro: rápido, reproducible y sin abrir el navegador ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 1 (SQL básico)

Este anexo contiene las soluciones sugeridas a los mini‑retos de la **Parte 1 — SQL básico** del compendio.

---

## Reto 1 — Insertar curso `Machine Learning`, 60 h

```sql
INSERT INTO cursos (nombre, descripcion, duracion_horas)
VALUES ("Machine Learning", "Introducción a modelos supervisados", 60);

```

---

## Reto 2 — Buscar cursos con `LIKE '%SQL%'`

```sql
SELECT * FROM cursos
WHERE nombre LIKE '%SQL%';

```

---

## Reto 3 — Ordenar por `nombre`

```sql
SELECT id, nombre, duracion_horas
FROM cursos
ORDER BY nombre ASC;

```

---

## Reto 4 — Seleccionar el curso con menor duración

```sql
SELECT id, nombre, duracion_horas
FROM cursos
ORDER BY duracion_horas ASC
LIMIT 1;

```

---

## Bonus en Python

Ejecutar los retos con `sqlite3` desde Python:

```python
import sqlite3
from pathlib import Path
DB_PATH = Path("data") / "kawaii.db"

with sqlite3.connect(DB_PATH) as conn:
    cur = conn.cursor()

    # Reto 1
    cur.execute("INSERT INTO cursos (nombre, descripcion, duracion_horas) VALUES (?, ?, ?);",
                ("Machine Learning", "Introducción a modelos supervisados", 60))
    conn.commit()

    # Reto 2
    print("Cursos que contienen 'SQL':")
    for row in cur.execute("SELECT * FROM cursos WHERE nombre LIKE '%SQL%';"):
        print(row)

    # Reto 3
    print("\nCursos ordenados por nombre:")
    for row in cur.execute("SELECT id, nombre, duracion_horas FROM cursos ORDER BY nombre ASC;"):
        print(row)

    # Reto 4
    print("\nCurso con menor duración:")
    print(cur.execute("SELECT id, nombre, duracion_horas FROM cursos ORDER BY duracion_horas ASC LIMIT 1;").fetchone())

```

---

✨ Con estos retos resueltos afianzás las bases de `INSERT`, `SELECT`, `WHERE`, `ORDER BY`, `LIMIT` ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 2 (Pandas + SQL)

Soluciones a los mini‑retos de **Parte 2 — Pandas + SQL**. Incluye código en pandas y consultas SQL parametrizadas.

---

## Reto 1 — Generar 100 estudiantes y `append`

```python
import numpy as np
import pandas as pd
import sqlite3
from pathlib import Path

DB_PATH = Path("data")/"kawaii.db"

np.random.seed(42)

nombres = [f"Alum_{i:03d}" for i in range(100)]
cursos = np.random.choice(["Python","SQL 101","Data Science"], size=100)
notas = np.round(np.random.uniform(6.0, 10.0, 100), 1)

batch = pd.DataFrame({"nombre":nombres, "curso":cursos, "nota":notas})

with sqlite3.connect(DB_PATH) as conn:
    batch.to_sql("estudiantes", conn, if_exists="append", index=False)

```

---

## Reto 2 — `groupby` promedio de `nota` por `curso` (orden DESC)

```python
import pandas as pd, sqlite3
from pathlib import Path
DB_PATH = Path("data")/"kawaii.db"

with sqlite3.connect(DB_PATH) as conn:
    df = pd.read_sql_query("SELECT curso, nota FROM estudiantes;", conn)

reporte = (df.groupby("curso")["nota"].mean()
             .round(2)
             .sort_values(ascending=False)
             .rename("promedio"))
print(reporte)

```

---

## Reto 3 — Exportar a Parquet y comparar tamaño vs CSV

```python
from pathlib import Path

out_csv = Path("data/reporte_promedios.csv")
out_parquet = Path("data/reporte_promedios.parquet")

reporte.to_frame().to_csv(out_csv, index=True)
reporte.to_frame().to_parquet(out_parquet, index=True)

print("Tamaño CSV (bytes):", out_csv.stat().st_size)
print("Tamaño Parquet (bytes):", out_parquet.stat().st_size)

```

> Nota: Parquet suele ser más compacto y columnar; en datasets muy chicos, la diferencia puede ser mínima.

---

## Reto 4 — Traer solo estudiantes con `nota >= 9` usando **parámetros**

```python
import sqlite3
from pathlib import Path
DB_PATH = Path("data")/"kawaii.db"

query = "SELECT nombre, curso, nota FROM estudiantes WHERE nota >= ? ORDER BY nota DESC;"
params = (9.0,)
with sqlite3.connect(DB_PATH) as conn:
    conn.row_factory = sqlite3.Row
    rows = conn.execute(query, params).fetchall()
    top = [dict(r) for r in rows]
print(top[:5])

```

---

## Reto 5 — Buscar cursos que empiecen con `Data` (`LIKE 'Data%'`)

**Opción A — SQL directa**

```python
sql = "SELECT * FROM cursos WHERE nombre LIKE 'Data%';"
with sqlite3.connect(DB_PATH) as conn:
    df_like = pd.read_sql_query(sql, conn)
print(df_like)

```

**Opción B — Parámetro con comodines**

```python
sql = "SELECT * FROM cursos WHERE nombre LIKE ?;"
with sqlite3.connect(DB_PATH) as conn:
    df_like = pd.read_sql_query(sql, conn, params=("Data%",))
print(df_like)

```

---

## Bonus — Pipeline compacto (todo junto)

```python
import numpy as np, pandas as pd, sqlite3
from pathlib import Path

DB_PATH = Path("data")/"kawaii.db"
np.random.seed(7)

# 1) Genero y appendeo 100 filas
n=100
df_new = pd.DataFrame({
    "nombre":[f"Seed_{i:03d}" for i in range(n)],
    "curso":np.random.choice(["Python","SQL 101","Data Science"], n),
    "nota":np.round(np.random.uniform(6,10,n),1)
})
with sqlite3.connect(DB_PATH) as conn:
    df_new.to_sql("estudiantes", conn, if_exists="append", index=False)

# 2) Promedio por curso (DESC)
with sqlite3.connect(DB_PATH) as conn:
    df_all = pd.read_sql_query("SELECT curso, nota FROM estudiantes;", conn)
rep = (df_all.groupby("curso")["nota"].mean().round(2).sort_values(ascending=False))

# 3) Export CSV/Parquet + tamaños
out_csv = Path("data/rep2.csv"); out_parq = Path("data/rep2.parquet")
rep.to_frame("prom").to_csv(out_csv)
rep.to_frame("prom").to_parquet(out_parq)
print("CSV bytes:", out_csv.stat().st_size, "| Parquet bytes:", out_parq.stat().st_size)

# 4) Solo >=9 con parámetros
with sqlite3.connect(DB_PATH) as conn:
    top9 = pd.read_sql_query(
        "SELECT nombre, curso, nota FROM estudiantes WHERE nota >= ? ORDER BY nota DESC;",
        conn, params=(9.0,)
    )
print(top9.head())

# 5) Cursos que empiezan con 'Data'
with sqlite3.connect(DB_PATH) as conn:
    data_courses = pd.read_sql_query("SELECT * FROM cursos WHERE nombre LIKE ?;", conn, params=("Data%",))
print(data_courses)

```

---

✨ Con estas soluciones, cerrás la Parte 2 dominando `read_sql_query`, `to_sql`, `groupby`, export y filtros parametrizados ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 3 (JOIN, GROUP BY, HAVING + ventanas)

Soluciones a los mini‑retos de **Parte 3 — JOIN, GROUP BY, HAVING + ventanas**.

---

## Reto 1 — `LEFT JOIN` para detectar `IS NULL`

Cursos sin estudiantes (usando `LEFT JOIN`):

```sql
SELECT c.id, c.nombre
FROM cursos c
LEFT JOIN estudiantes e ON e.curso = c.nombre
WHERE e.id IS NULL;

```

---

## Reto 2 — `HAVING COUNT(*) >= 5`

Cursos con al menos 5 estudiantes:

```sql
SELECT e.curso, COUNT(*) AS n
FROM estudiantes e
GROUP BY e.curso
HAVING COUNT(*) >= 5;

```

---

## Reto 3 — Top1 por curso con `ROW_NUMBER()`

```sql
SELECT * FROM (
  SELECT e.nombre, e.curso, e.nota,
         ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
  FROM estudiantes e
) t
WHERE rn = 1;

```

---

## Reto 4 — Ranking global con `DENSE_RANK()`

```sql
SELECT e.nombre, e.curso, e.nota,
       DENSE_RANK() OVER (ORDER BY e.nota DESC) AS rnk
FROM estudiantes e
ORDER BY rnk, nombre;

```

---

## Reto 5 — Reporte CSV consolidado

Armar `curso, n, nota_prom, top1_nombre, top1_nota`:

```sql
WITH agg AS (
  SELECT curso, COUNT(*) AS n, ROUND(AVG(nota),2) AS nota_prom
  FROM estudiantes
  GROUP BY curso
),
top1 AS (
  SELECT curso, nombre AS top1_nombre, nota AS top1_nota
  FROM (
    SELECT e.curso, e.nombre, e.nota,
           ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
    FROM estudiantes e
  )
  WHERE rn = 1
)
SELECT a.curso, a.n, a.nota_prom, t.top1_nombre, t.top1_nota
FROM agg a
JOIN top1 t USING(curso)
ORDER BY a.nota_prom DESC;

```

---

## Bonus en Python (pandas + sqlite3)

```python
import sqlite3, pandas as pd
from pathlib import Path
DB_PATH = Path("data")/"kawaii.db"

with sqlite3.connect(DB_PATH) as conn:
    # Cursos sin estudiantes
    sin_est = pd.read_sql_query("""
    SELECT c.id, c.nombre
    FROM cursos c
    LEFT JOIN estudiantes e ON e.curso = c.nombre
    WHERE e.id IS NULL;
    """, conn)
    print("Cursos sin estudiantes:\n", sin_est)

    # Cursos con >=5 estudiantes
    muchos = pd.read_sql_query("""
    SELECT e.curso, COUNT(*) AS n
    FROM estudiantes e
    GROUP BY e.curso
    HAVING COUNT(*) >= 5;
    """, conn)
    print("\nCursos con >=5 estudiantes:\n", muchos)

    # Top1 por curso
    top1 = pd.read_sql_query("""
    SELECT * FROM (
      SELECT e.nombre, e.curso, e.nota,
             ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
      FROM estudiantes e
    ) t
    WHERE rn = 1;
    """, conn)
    print("\nTop1 por curso:\n", top1)

    # Ranking global
    ranking = pd.read_sql_query("""
    SELECT e.nombre, e.curso, e.nota,
           DENSE_RANK() OVER (ORDER BY e.nota DESC) AS rnk
    FROM estudiantes e
    ORDER BY rnk, nombre;
    """, conn)
    print("\nRanking global:\n", ranking)

    # Reporte consolidado
    reporte = pd.read_sql_query("""
    WITH agg AS (
      SELECT curso, COUNT(*) AS n, ROUND(AVG(nota),2) AS nota_prom
      FROM estudiantes
      GROUP BY curso
    ),
    top1 AS (
      SELECT curso, nombre AS top1_nombre, nota AS top1_nota
      FROM (
        SELECT e.curso, e.nombre, e.nota,
               ROW_NUMBER() OVER (PARTITION BY e.curso ORDER BY e.nota DESC) AS rn
        FROM estudiantes e
      )
      WHERE rn = 1
    )
    SELECT a.curso, a.n, a.nota_prom, t.top1_nombre, t.top1_nota
    FROM agg a
    JOIN top1 t USING(curso)
    ORDER BY a.nota_prom DESC;
    """, conn)
    print("\nReporte consolidado:\n", reporte)

```

---

✨ Con estos retos resueltos, dominás joins, agrupaciones, filtros con HAVING y ventanas (ranking/topN) ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 4 (Índices, EXPLAIN y performance)

Soluciones a los mini‑retos de **Parte 4 — Índices, EXPLAIN y performance**.

---

## Reto 1 — Índice en `cursos(nombre)` y búsqueda `LIKE 'SQL%'`

```sql
CREATE INDEX IF NOT EXISTS idx_cursos_nombre ON cursos(nombre);

-- Consulta acelerada por índice (cuando aplica)
SELECT id, nombre, duracion_horas
FROM cursos
WHERE nombre LIKE 'SQL%'
ORDER BY nombre ASC;

```

> Nota: índices ayudan más con comparaciones y prefijos (`LIKE 'SQL%'`). Un patrón con comodín inicial (`'%SQL%'`) **no** usa el índice estándar.

---

## Reto 2 — Verificar uso con `EXPLAIN QUERY PLAN`

```sql
EXPLAIN QUERY PLAN
SELECT id, nombre FROM cursos WHERE nombre LIKE 'SQL%';

```

Salida esperada (ejemplo): `SEARCH cursos USING INDEX idx_cursos_nombre (nombre>? AND nombre<?)`

---

## Reto 3 — Medir tiempos con y sin índice (Python)

```python
import sqlite3, time
from pathlib import Path
DB = Path('data')/'kawaii.db'

query = "SELECT * FROM cursos WHERE nombre LIKE 'SQL%' ORDER BY nombre;"

with sqlite3.connect(DB) as conn:
    # Borrar índice para medir "sin"
    conn.execute("DROP INDEX IF EXISTS idx_cursos_nombre;")
    t0 = time.time();
    for _ in range(2000):
        conn.execute(query).fetchall()
    t1 = time.time()
    sin_idx = t1 - t0

    # Crear índice y medir "con"
    conn.execute("CREATE INDEX IF NOT EXISTS idx_cursos_nombre ON cursos(nombre);")
    t0 = time.time();
    for _ in range(2000):
        conn.execute(query).fetchall()
    t1 = time.time()
    con_idx = t1 - t0

print(f"Tiempo sin índice: {sin_idx:.3f}s | con índice: {con_idx:.3f}s")

```

> En tablas pequeñas puede no notarse; en tablas grandes la diferencia es clara.

---

## Reto 4 — Índice **único** en `estudiantes(email)` y prueba de duplicados

```sql
-- Agregar columna si no existe
ALTER TABLE estudiantes ADD COLUMN email TEXT;  -- si ya existe, ignorar este paso

-- Índice único
CREATE UNIQUE INDEX IF NOT EXISTS idx_estudiantes_email ON estudiantes(email);

-- Insert válido
INSERT INTO estudiantes(nombre, curso, nota, email)
VALUES('Zoe','Python',9.0,'zoe@example.com');

-- Insert duplicado (debe FALLAR por unicidad)
INSERT INTO estudiantes(nombre, curso, nota, email)
VALUES('Otra Zoe','Python',8.0,'zoe@example.com');

```

> El segundo `INSERT` debería lanzar un error de violación de unicidad.

---

## Reto 5 — Índice compuesto `curso, nota DESC` y consulta que lo aprovecha

```sql
CREATE INDEX IF NOT EXISTS idx_estudiantes_curso_nota
ON estudiantes(curso, nota DESC);

-- Consulta típica que lo usa: filtro por curso + orden por nota
SELECT nombre, curso, nota
FROM estudiantes
WHERE curso = 'Python'
ORDER BY nota DESC
LIMIT 5;

```

---

## Bonus en Python — EXPLAIN + verificación rápida

```python
import sqlite3
from pathlib import Path
DB = Path('data')/'kawaii.db'

with sqlite3.connect(DB) as conn:
    conn.row_factory = sqlite3.Row

    # A) EXPLAIN de prefijo 'SQL%'
    plan = conn.execute(
        "EXPLAIN QUERY PLAN SELECT id, nombre FROM cursos WHERE nombre LIKE 'SQL%';"
    ).fetchall()
    print("PLAN cursos LIKE 'SQL%':", [dict(r) for r in plan])

    # B) EXPLAIN de top por curso (aprovecha índice compuesto)
    plan2 = conn.execute(
        "EXPLAIN QUERY PLAN SELECT nombre, curso, nota FROM estudiantes WHERE curso = 'Python' ORDER BY nota DESC LIMIT 5;"
    ).fetchall()
    print("PLAN top Python:", [dict(r) for r in plan2])

    # C) Probar unicidad de email (capturar error)
    try:
        conn.execute("INSERT INTO estudiantes(nombre, curso, nota, email) VALUES(?, ?, ?, ?);",
                     ("Test Duplicado","Python",7.5,"zoe@example.com"))
        conn.commit()
    except sqlite3.IntegrityError as e:
        print("Unicidad OK:", e)

```

---

✨ Con estos retos resueltos, ya sabés **cuándo** crear índices, **cómo** comprobar que se usen, y **por qué** afectan lectura/escritura ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 5 (SQLAlchemy Core y ORM)

Soluciones a los mini‑retos de **Parte 5 — SQLAlchemy Core y ORM**. Incluye esquema ORM 1‑N, consultas, updates/borrados y export con pandas.

> Nota: para no chocar con las tablas existentes (`estudiantes` con `curso` TEXT), este anexo usa **tablas nuevas** con clave foránea: `curso_orm` y `estudiante_orm`.

---

## Setup común

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey, select, update, delete
from sqlalchemy.orm import declarative_base, relationship, Session
import pandas as pd

engine = create_engine("sqlite:///data/kawaii.db", echo=False)
Base = declarative_base()

```

---

## Reto 1 — Clase `Curso` y relación 1‑N con `Estudiante`

```python
class Curso(Base):
    __tablename__ = "curso_orm"
    id = Column(Integer, primary_key=True)
    nombre = Column(String(100), unique=True, nullable=False)
    descripcion = Column(String(255))

    # 1‑N → un curso tiene muchas estudiantes
    estudiantes = relationship("Estudiante", back_populates="curso", cascade="all, delete-orphan")

class Estudiante(Base):
    __tablename__ = "estudiante_orm"
    id = Column(Integer, primary_key=True)
    nombre = Column(String(100), nullable=False)
    nota = Column(Float)
    curso_id = Column(Integer, ForeignKey("curso_orm.id"), nullable=False)

    curso = relationship("Curso", back_populates="estudiantes")

Base.metadata.create_all(engine)

# Semilla de datos
with Session(engine) as s:
    if not s.query(Curso).first():
        sql101 = Curso(nombre="SQL 101", descripcion="Intro kawaii a BD")
        py = Curso(nombre="Python", descripcion="Scripting útil")
        ds = Curso(nombre="Data Science", descripcion="pandas + numpy")
        s.add_all([
            sql101, py, ds,
            Estudiante(nombre="Miyu", nota=9.5, curso=sql101),
            Estudiante(nombre="Kora", nota=8.8, curso=py),
            Estudiante(nombre="Lyra", nota=9.1, curso=ds),
            Estudiante(nombre="Nia", nota=7.9, curso=py),
        ])
        s.commit()

```

---

## Reto 2 — Consultar todas las estudiantes de `SQL 101` (ORM)

```python
with Session(engine) as s:
    q = (
        s.query(Estudiante)
         .join(Estudiante.curso)
         .filter(Curso.nombre == "SQL 101")
         .order_by(Estudiante.nota.desc())
    )
    for e in q.all():
        print(e.nombre, e.nota)

```

**Opción Core (equivalente):**

```python
from sqlalchemy import select
with engine.connect() as conn:
    stmt = (
        select(Estudiante.nombre, Estudiante.nota)
        .join(Curso, Estudiante.curso_id == Curso.id)
        .where(Curso.nombre == "SQL 101")
        .order_by(Estudiante.nota.desc())
    )
    print(conn.execute(stmt).fetchall())

```

---

## Reto 3 — Subir +0.5 a notas de todas las de `Python` (ORM)

```python
with Session(engine) as s:
    q = (
        s.query(Estudiante)
         .join(Estudiante.curso)
         .filter(Curso.nombre == "Python")
    )
    for e in q.all():
        e.nota = min((e.nota or 0) + 0.5, 10.0)
    s.commit()

```

**Opción Core (UPDATE masivo):**

```python
with engine.begin() as conn:
    stmt = (
        update(Estudiante)
        .where(Estudiante.curso_id == select(Curso.id).where(Curso.nombre=="Python").scalar_subquery())
        .values(nota=((Estudiante.nota + 0.5)))
    )
    conn.execute(stmt)

```

> En SQLite, `UPDATE` con expresión funciona; límite a 10.0 podría hacerse con `MIN()` si querés estrictamente.

---

## Reto 4 — Borrar estudiantes con `nota < 7`

```python
with Session(engine) as s:
    s.execute(
        delete(Estudiante).where(Estudiante.nota < 7)
    )
    s.commit()

```

> Si la relación está con `cascade="all, delete-orphan"`, borrar un curso borra sus estudiantes. Acá sólo eliminamos alumnas por condición.

---

## Reto 5 — Exportar resultados con pandas desde `session.query()`

```python
with Session(engine) as s:
    q = (
        s.query(Curso.nombre.label("curso"), Estudiante.nombre.label("estudiante"), Estudiante.nota)
         .join(Estudiante)
         .order_by(Curso.nombre, Estudiante.nota.desc())
    )
    df = pd.read_sql(q.statement, s.bind)  # usar el statement SQL generado

df.to_csv("data/orm_export.csv", index=False)
print(df.head())

```

---

## (Extra) Crear rápidamente un curso y añadir estudiantes (patrón de trabajo)

```python
with Session(engine) as s:
    ml = s.query(Curso).filter_by(nombre="Machine Learning").one_or_none()
    if ml is None:
        ml = Curso(nombre="Machine Learning", descripcion="modelos supervisados")
        s.add(ml); s.commit()

    s.add_all([
        Estudiante(nombre="Nova", nota=9.8, curso=ml),
        Estudiante(nombre="Sibel", nota=8.9, curso=ml),
    ])
    s.commit()

```

---

✨ Con estas soluciones dominás el ciclo ORM completo: **definir relaciones**, **consultar**, **actualizar**, **borrar** y **exportar** ✦

---

# 🌸 Anexo ✦ Mini‑retos Resueltos — Parte 6 (Postgres & MySQL)

Soluciones a los mini‑retos de **Parte 6 — Conexión a Postgres/MySQL**. Incluye ORM en Postgres, carga desde CSV y consultas en MySQL, con credenciales via variables de entorno.

> Requisitos: `SQLAlchemy`, `psycopg2-binary` (Postgres), `PyMySQL` (MySQL), `pandas`, `python-dotenv` (opcional).

```bash
pip install SQLAlchemy psycopg2-binary PyMySQL pandas python-dotenv

```

---

## Setup de credenciales (seguro y kawaii)

**.env** (no lo subas a Git):

```
PG_URL=postgresql+psycopg2://usuario:password@localhost:5432/mibase
MYSQL_URL=mysql+pymysql://usuario:password@localhost:3306/mibase

```

**cargar_env.py**

```python
import os
from dotenv import load_dotenv
from sqlalchemy import create_engine

load_dotenv()  # carga .env
engine_pg = create_engine(os.getenv("PG_URL"), echo=True)
engine_mysql = create_engine(os.getenv("MYSQL_URL"), echo=True)

```

---

## Reto 1 — Tabla `estudiantes` en Postgres con ORM

```python
from sqlalchemy.orm import declarative_base, Session
from sqlalchemy import Column, Integer, String, Float
from cargar_env import engine_pg

Base = declarative_base()

class EstudiantePG(Base):
    __tablename__ = "estudiantes"
    id = Column(Integer, primary_key=True)
    nombre = Column(String(100), nullable=False)
    curso = Column(String(100), nullable=False)
    nota = Column(Float)

Base.metadata.create_all(engine_pg)

# Insert demo
with Session(engine_pg) as s:
    s.add_all([
        EstudiantePG(nombre="Miyu", curso="SQL 101", nota=9.5),
        EstudiantePG(nombre="Kora", curso="Python", nota=8.8),
    ])
    s.commit()

# Verificar
with Session(engine_pg) as s:
    for e in s.query(EstudiantePG).order_by(EstudiantePG.nota.desc()).all():
        print(e.nombre, e.curso, e.nota)

```

---

## Reto 2 — `to_sql(engine_pg)` desde CSV

```python
import pandas as pd
from cargar_env import engine_pg

# Suponé un estudiantes.csv con columnas: nombre,curso,nota
df = pd.read_csv("estudiantes.csv")
# Carga reemplazando la tabla (o usá append según tu caso)
df.to_sql("estudiantes", engine_pg, if_exists="append", index=False)

# Verificá con una pequeña lectura
print(pd.read_sql_query("SELECT COUNT(*) AS n FROM estudiantes;", engine_pg))

```

> Si necesitás esquema/campos estrictos, preferí ORM/Core para crear la tabla y `to_sql` solo para ingesta.

---

## Reto 3 — Top 5 cursos en MySQL por nombre (alfabético)

```python
from sqlalchemy import text
from cargar_env import engine_mysql

query = text("SELECT id, nombre FROM cursos ORDER BY nombre ASC LIMIT 5;")
with engine_mysql.connect() as conn:
    rows = conn.execute(query).fetchall()
    print(rows)

```

> Si tu tabla `cursos` no existe aún en MySQL, créala antes con Core/ORM.

---

## Reto 4 — `Session` update + `commit()` en Postgres

Subir +0.5 a las notas del curso `Python` (tope 10.0):

```python
from sqlalchemy import update
from sqlalchemy.orm import Session
from cargar_env import engine_pg
from main_models import EstudiantePG  # o definilo en el mismo archivo del Reto 1

with Session(engine_pg) as s:
    # Opción ORM fila a fila
    q = s.query(EstudiantePG).filter(EstudiantePG.curso == "Python").all()
    for e in q:
        e.nota = min((e.nota or 0) + 0.5, 10.0)
    s.commit()

# Opción Core masiva (una sola sentencia)
from sqlalchemy import select
with engine_pg.begin() as conn:
    # Nota: en muchos motores podés usar funciones tipo LEAST/CASE para clamp
    conn.execute(update(EstudiantePG)
                 .where(EstudiantePG.curso == "Python")
                 .values(nota=EstudiantePG.nota + 0.5))

```

---

## Reto 5 — Credenciales con `os.getenv` (ya aplicado)

El archivo `cargar_env.py` muestra el patrón correcto:

- Mantener URLs en `.env`.
- Cargar con `dotenv`.
- Construir `engine` con `create_engine(os.getenv(...))`.

Para comprobar que todo está bien:

```python
import os
assert os.getenv("PG_URL"), "Falta PG_URL en .env"
assert os.getenv("MYSQL_URL"), "Falta MYSQL_URL en .env"

```

---

## Bonus — Carga CSV → Postgres y consulta con pandas

```python
import pandas as pd
from cargar_env import engine_pg

# Cargar CSV (append)
pd.read_csv("estudiantes.csv").to_sql("estudiantes", engine_pg, if_exists="append", index=False)

# Reporte de promedios por curso
df = pd.read_sql_query(
    "SELECT curso, AVG(nota) AS prom, COUNT(*) AS n FROM estudiantes GROUP BY curso ORDER BY prom DESC;",
    engine_pg
)
print(df)

```

---

✨ Con estos retos resueltos, ya sabés conectar con seguridad, cargar datos reales y consultar tanto en Postgres como en MySQL ✦

---

## 🎉 Cierre

Si llegaste hasta acá, ya dominás **SQL + Python** con SQLite, pandas, SQLAlchemy, y conexiones a Postgres/MySQL. Hmph~ no es que me enorgullezca… pero sí, me enorgullezco ✦

**Siguiente paso opcional:** anexar este compendio con tus notas y datasets reales; versionarlo en Git y preparar una demo con FastAPI o un notebook de showcase.
