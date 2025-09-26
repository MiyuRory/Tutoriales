# 🌸 Precuela Fractal ✦ Índice General

## Partes

- **A — Python base kawaii**: variables, listas, diccionarios, funciones, bucles, entornos virtuales.
- **B — Pensar datos en Python**: listas de diccionarios como tablas, claves únicas, relaciones entre colecciones.
- **C — Pandas como juguete previo**: crear DataFrames, filtrar, agrupar, exportar.
- **D — Puente a bases de datos**: tablas, filas, columnas, claves primarias, relaciones, CRUD conceptual.
- **E — Checklist final**: autoevaluación de todo lo aprendido para saltar al tutorial SQL+Python (0–7).

---

✨ Este índice resume la ruta de la precuela: del Python más básico → pensamiento tabular → pandas → conceptos de bases de datos → checklist de preparación ✦

# 🌸 Parte A — Python base kawaii

### 🎯 Objetivo

Conocer las piezas mínimas de Python para manipular datos y preparar la cabeza para pandas y bases de datos.

---

## 1. Variables y tipos primitivos

En Python, podés guardar valores en variables sin declarar el tipo:

```python
nombre = "Miyu"       # string (texto)
nota = 9.5            # float (decimal)
edad = 22             # int (entero)
aprobada = True       # bool (booleano)

```

`print()` te deja inspeccionar:

```python
print(nombre, nota, edad, aprobada)

```

💡 Tip kawaii: usá nombres claros, como `nota_final` en vez de `x`.

---

## 2. Listas (colecciones ordenadas)

```python
cursos = ["Python", "Data Science", "SQL 101"]
print(cursos[0])   # primer elemento
print(cursos[-1])  # último elemento

```

Podés modificar:

```python
cursos.append("Machine Learning")
print(cursos)

```

---

## 3. Diccionarios (colecciones clave → valor)

```python
estudiante = {
  "nombre": "Miyu",
  "curso": "Python",
  "nota": 9.5
}

print(estudiante["curso"])

```

Podés cambiar valores:

```python
estudiante["nota"] = 10

```

---

## 4. Funciones kawaii

Sirven para encapsular lógica:

```python
def promedio(notas):
    return sum(notas) / len(notas)

print(promedio([8, 9, 10]))

```

---

## 5. Bucles

```python
for curso in cursos:
    print("Estoy aprendiendo:", curso)

i = 0
while i < 3:
    print("Iteración", i)
    i += 1

```

---

## 6. Entorno virtual y librerías

Para mantener tu proyecto ordenado:

```bash
python -m venv .venv
# Activar
source .venv/bin/activate   # Linux/Mac
.\.venv\\Scripts\\activate  # Windows
# Instalar pandas
pip install pandas

```

---

### 🧪 Mini-retos A

1.  Lista con 5 números y mostrarlos con un bucle `for`.
2.  Diccionario con `nombre, edad, hobby`.
3.  Función `aprobada(nota)` que devuelva True si ≥ 7.

---

# 🌸 Parte B — Pensar datos en Python (fractal)

### 🎯 Objetivo

Aprender a representar datos como si fueran tablas, usando solo estructuras de Python. Esto te prepara para entender cómo funcionan las bases de datos sin necesidad de SQL todavía.

---

## 1. Tablas como listas de diccionarios

En vez de pensar en filas y columnas, en Python usamos una **lista de diccionarios**:

```python
estudiantes = [
    {"nombre": "Miyu", "curso": "Python", "nota": 9.5},
    {"nombre": "Kora", "curso": "SQL 101", "nota": 8.7},
]

for e in estudiantes:
    print(e["nombre"], e["nota"])

```

Esto es como una **mini-tabla**:

nombre

curso

nota

Miyu

Python

9.5

Kora

SQL 101

8.7

---

## 2. Claves únicas

Para diferenciar registros, usamos un `id`.

```python
estudiantes = [
    {"id": 1, "nombre": "Miyu", "curso": "Python"},
    {"id": 2, "nombre": "Miyu", "curso": "SQL 101"},
]

```

Aunque tengan el mismo nombre, cada estudiante tiene un `id` distinto.

---

## 3. Relaciones entre colecciones

Un curso puede tener muchas alumnas:

```python
curso = {
  "nombre": "Python",
  "alumnas": ["Miyu", "Kora", "Lyra"]
}

```

Y al revés: una alumna puede estar en muchos cursos.

```python
alumna = {
  "nombre": "Miyu",
  "cursos": ["Python", "SQL 101"]
}

```

Esto es la semilla para después entender las **relaciones entre tablas**.

---

### 🧪 Mini-retos B

1.  Crear lista con 3 cursos, cada uno con su lista de alumnas.
2.  Recorrer y mostrar: “Curso X tiene N alumnas”.
3.  Agregar una nueva alumna a un curso existente.
4.  Pensar: ¿cómo representarías las notas si una alumna está en varios cursos?

---

# 🌸 Parte C — Pandas como juguete previo (fractal)

### 🎯 Objetivo

Explorar `pandas` como herramienta para manipular datos tipo tablas, sin entrar todavía en SQL. Es como un campo de juegos para acostumbrarse a pensar en filas y columnas.

---

## 1. Crear un DataFrame

```python
import pandas as pd

df = pd.DataFrame({
  "nombre": ["Miyu", "Kora", "Lyra"],
  "curso": ["Python", "SQL 101", "Python"],
  "nota": [9.5, 8.7, 9.1]
})

print(df)

```

Salida:

```
  nombre    curso  nota
0   Miyu   Python   9.5
1   Kora  SQL 101   8.7
2   Lyra   Python   9.1

```

---

## 2. Filtrar datos

```python
print(df[df["nota"] >= 9])

```

Salida:

```
  nombre   curso  nota
0   Miyu  Python   9.5
2   Lyra  Python   9.1

```

---

## 3. Agrupar y calcular

```python
print(df.groupby("curso")["nota"].mean())

```

Salida:

```
curso
Python     9.3
SQL 101    8.7
Name: nota, dtype: float64

```

---

## 4. Exportar resultados

```python
df.to_csv("reporte.csv", index=False)

```

Esto genera un archivo CSV que podés abrir con Excel o un editor de texto.

---

### 🧪 Mini-retos C

1.  Crear un DataFrame con 5 alumnas.
2.  Filtrar todas con nota < 8.
3.  Calcular promedio de notas por curso.
4.  Exportar a `alumnas.csv` y abrirlo para ver el resultado.
5.  Agregar una columna nueva: `aprobada = nota >= 7`.

---

✨ Con pandas ya te vas acostumbrando a manipular datos como tablas, lo cual hace que después SQL se sienta natural ✦

---

# 🌸 Parte D — Puente a bases de datos (fractal)

### 🎯 Objetivo

Entender los **conceptos esenciales de bases de datos** a partir de lo que ya conocés con listas, diccionarios y pandas. Sin SQL aún: solo la idea.

---

## 1. Tabla ≈ Hoja de Excel

- Una **tabla** es como una hoja con filas y columnas.
- Cada **fila** es un registro (ej: una estudiante).
- Cada **columna** es un atributo (ej: nombre, curso, nota).

Ejemplo:

id

nombre

curso

nota

1

Miyu

Python

9.5

2

Kora

SQL 101

8.7

---

## 2. Clave primaria

Es un identificador único de cada fila.

- En el ejemplo: `id`.
- Sirve para distinguir aunque dos estudiantes se llamen igual.

---

## 3. Relaciones

### Uno a muchos

- Un curso tiene muchas alumnas.
- Ejemplo: `curso.id = 1` se asocia con muchas filas en `estudiantes`.

### Muchos a muchos

- Una alumna puede estar en varios cursos.
- Esto suele resolverse con una tabla intermedia (más adelante lo verás en SQL).

---

## 4. Operaciones CRUD (conceptual)

- **C**reate → agregar fila.
- **R**ead → consultar datos.
- **U**pdate → cambiar datos.
- **D**elete → eliminar fila.

Ya hiciste esto en Python con listas y pandas:

- `append` = Create.
- Filtrar con condicionales = Read.
- Reasignar valores = Update.
- `remove` o drop en pandas = Delete.

---

## 5. Pensamiento tabular

Cuando armes estructuras en Python (listas, diccionarios, DataFrames), pensalas como mini-bases de datos:

- ¿Cuál sería la clave primaria?
- ¿Qué relaciones existen?
- ¿Qué operaciones CRUD quiero hacer?

---

### 🧪 Mini-retos D

1.  Imaginá una tabla `libros` con columnas `id, titulo, autor, anio`.
2.  Decí cuál sería la clave primaria.
3.  Pensá: ¿qué relación habría entre `libros` y `autoras`?
4.  Escribí los cuatro pasos CRUD aplicados a tu tabla `libros`.

---

✨ Con este puente conceptual, ya estás lista para saltar del mundo Python/pandas al mundo SQL ✦

---

# 🌸 Parte E — Checklist final (fractal)

### 🎯 Objetivo

Verificar que tengas las habilidades mínimas para dar el salto al **tutorial SQL+Python (0–7)**.

---

## 1. Python base

- Sé crear variables (`nombre = "Miyu"`).
- Uso listas y puedo recorrerlas con `for`.
- Manejo diccionarios (`estudiante["curso"]`).
- Escribí funciones simples (`def promedio(notas): ...`).

---

## 2. Pensar datos

- Representé una **tabla** con lista de diccionarios.
- Entendí qué es un `id` como clave única.
- Practiqué cómo un curso puede tener muchas alumnas.

---

## 3. Pandas

- Creé un `DataFrame` con datos ficticios.
- Filtré filas (`df[df["nota"] >= 9]`).
- Agrupé y calculé promedios.
- Exporté a CSV.

---

## 4. Conceptos de bases de datos

- Entiendo la idea de tabla, fila y columna.
- Sé qué es una clave primaria.
- Puedo explicar relaciones uno-a-muchos y muchos-a-muchos.
- Relacioné operaciones CRUD con lo que ya hice en Python/pandas.

---

## 5. Ready check ✦

Si tildaste la mayoría, ya tenés:

- Bases de Python.
- Pensamiento tabular.
- Experiencia con pandas.
- Conceptos claros de bases de datos.

✨ Estás lista para comenzar el **tutorial SQL+Python (Partes 0–7)** y volverte maga de consultas ✦

---

# 🌸 Precuela Fractal ✦ Anexo de Mini‑retos Resueltos

Este anexo contiene las soluciones a todos los mini‑retos planteados en las Partes A–D.

---

# 🌸 Parte A — Python base kawaii ✦

### 🧪 Mini‑retos A

**1. Lista con 5 números y mostrarlos con un bucle `for`.**

```python
numeros = [1, 2, 3, 4, 5]
for n in numeros:
    print(n)

```

**2. Diccionario con `nombre, edad, hobby`.**

```python
persona = {"nombre": "Miyu", "edad": 22, "hobby": "dibujar"}
print(persona)

```

**3. Función `aprobada(nota)` que devuelva True si ≥ 7.**

```python
def aprobada(nota):
    return nota >= 7

print(aprobada(9))   # True
print(aprobada(6))   # False

```

---

# 🌸 Parte B — Pensar datos en Python ✦

### 🧪 Mini‑retos B

**1. Crear lista con 3 cursos, cada uno con su lista de alumnas.**

```python
cursos = [
    {"nombre": "Python", "alumnas": ["Miyu", "Kora"]},
    {"nombre": "SQL 101", "alumnas": ["Lyra", "Nia"]},
    {"nombre": "Data Science", "alumnas": ["Nova"]}
]

```

**2. Recorrer y mostrar: “Curso X tiene N alumnas”.**

```python
for curso in cursos:
    print(f"Curso {curso['nombre']} tiene {len(curso['alumnas'])} alumnas")

```

**3. Agregar una nueva alumna a un curso existente.**

```python
cursos[0]["alumnas"].append("Sibel")
print(cursos[0])

```

**4. Representar notas si una alumna está en varios cursos.**

```python
alumna = {
  "nombre": "Miyu",
  "notas": {"Python": 9.5, "SQL 101": 8.7}
}

```

---

# 🌸 Parte C — Pandas como juguete previo ✦

### 🧪 Mini‑retos C

**1. Crear un DataFrame con 5 alumnas.**

```python
import pandas as pd

df = pd.DataFrame({
  "nombre": ["Miyu", "Kora", "Lyra", "Nia", "Nova"],
  "curso": ["Python", "SQL 101", "Python", "Data Science", "SQL 101"],
  "nota": [9.5, 8.7, 9.1, 7.8, 10.0]
})
print(df)

```

**2. Filtrar todas con nota < 8.**

```python
print(df[df["nota"] < 8])

```

**3. Calcular promedio de notas por curso.**

```python
print(df.groupby("curso")["nota"].mean())

```

**4. Exportar a `alumnas.csv`.**

```python
df.to_csv("alumnas.csv", index=False)

```

**5. Agregar columna `aprobada = nota >= 7`.**

```python
df["aprobada"] = df["nota"] >= 7
print(df)

```

---

# 🌸 Parte D — Puente a bases de datos ✦

### 🧪 Mini‑retos D

**1. Imaginá tabla `libros` con columnas.**

```
libros(id, titulo, autor, anio)

```

**2. Clave primaria.**

- `id`

**3. Relación entre `libros` y `autoras`.**

- Un libro tiene una autora (1 a N).
- Una autora puede tener varios libros (1 a N).
- También podría ser muchos a muchos si hay coautoría.

**4. CRUD aplicado a `libros`.**

- **Create**: agregar libro nuevo a la tabla.
- **Read**: listar todos los libros de una autora.
- **Update**: corregir el `anio` de un libro.
- **Delete**: borrar un libro de la tabla.

---

✨ Con este anexo ya tenés ejemplos resueltos de todos los mini‑retos, para comparar con tus propias soluciones ✦

---

# 🌸 Precuela Fractal ✦ Anexo Visual

Este anexo muestra diagramas en texto para visualizar tablas, claves y relaciones.

---

## 1. Tabla simple (estudiantes)

```
┌────┬─────────┬───────────┬───────┐
│ id │ nombre  │ curso     │ nota  │
├────┼─────────┼───────────┼───────┤
│  1 │ Miyu    │ Python    │  9.5  │
│  2 │ Kora    │ SQL 101   │  8.7  │
│  3 │ Lyra    │ Python    │  9.1  │
└────┴─────────┴───────────┴───────┘

```

Clave primaria: `id`

---

## 2. Relación uno a muchos (Curso → Estudiantes)

```
┌───────────┐        ┌───────────────────┐
│   Curso   │ 1    N │    Estudiante     │
├───────────┤        ├───────────────────┤
│ id (PK)   │◄──────►│ curso_id (FK)     │
│ nombre    │        │ nombre, nota ...  │
└───────────┘        └───────────────────┘

```

- Un curso tiene muchas estudiantes.
- Una estudiante pertenece a un curso.

---

## 3. Relación muchos a muchos (Estudiantes ↔ Cursos)

Se necesita tabla intermedia:

```
┌───────────┐        ┌───────────────────┐        ┌───────────┐
│ Estudiante│        │ Estudiante_Curso  │        │   Curso   │
├───────────┤        ├───────────────────┤        ├───────────┤
│ id (PK)   │◄──────►│ estudiante_id (FK)│        │ id (PK)   │
│ nombre    │        │ curso_id (FK)     │◄──────►│ nombre    │
└───────────┘        └───────────────────┘        └───────────┘

```

- Una estudiante puede estar en varios cursos.
- Un curso puede tener varias estudiantes.

---

## 4. CRUD ilustrado

```
C = Create → INSERT fila
R = Read   → SELECT fila(s)
U = Update → UPDATE fila
D = Delete → DELETE fila

```

Ejemplo sobre `estudiantes`:

```
Create → Añadir (id=4, nombre="Nia", curso="SQL 101", nota=7.9)
Read   → Listar todas las estudiantes de Python
Update → Subir nota de Kora de 8.7 a 9.0
Delete → Eliminar a Nia

```

---

✨ Estos diagramas visuales ayudan a imaginar cómo se conectan las estructuras antes de pasar a SQL real ✦

---
