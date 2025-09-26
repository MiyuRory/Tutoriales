# 🌸 Prompt Próxima Sesión ✦ API CRUD con Auth & Swagger

> **Contexto base**
> Ya tengo:
>
> - Precuela A–E (Python, pensamiento de datos, pandas, puente a BD).
> - Compendio SQL+Python Partes 0–7 (SQLite, pandas, JOIN/GROUP BY, índices, SQLAlchemy, Postgres/MySQL, mini-proyecto + FastAPI).
> - Anexos con mini-retos resueltos (Partes 1–6) y ampliación de Parte 7 (API `/reporte`, `/top/{n}`, `/curso/{nombre}`), más **CLI** para consumir endpoints.

---

## 🎯 Objetivo de esta sesión

Diseñar e implementar **una API CRUD completa** (altas, bajas, modificaciones, lecturas) con **autenticación** y **documentación Swagger**, lista para correr localmente y extensible a producción.

---

## 1. Stack sugerido

- **FastAPI** (docs Swagger/OpenAPI automáticas).
- **SQLAlchemy ORM** + **Pydantic** para esquemas.
- **SQLite** en dev (archivo `data/kawaii.db`).
- **JWT** para auth (login, refresh opcional).
- Tests básicos con `pytest` o `httpx`.

---

## 2. Modelos ORM + Schemas Pydantic

- `Curso {id, nombre (único), descripcion, duracion_horas}`.
- `Estudiante {id, nombre, email (único), nota, curso_id (FK)}`.
- `User {id, email (único), password_hash, rol}` (para auth).

---

## 3. Endpoints CRUD (con `tags`)

- `/cursos`: `POST`, `GET (lista + paginación)`, `GET/{id}`, `PUT/{id}`, `DELETE/{id}`.
- `/estudiantes`: CRUD + filtros (`?curso=…&min_nota=…`), orden y paginación.
- `/auth/login` (JWT), `/auth/me` (perfil actual), `/auth/refresh` (opcional).

---

## 4. Seguridad

- Hash de contraseñas (`passlib[bcrypt]`).
- JWT con expiración.
- Proteger rutas de escritura a rol `admin`.
- Validaciones: unicidad (`email`, `curso.nombre`), errores `404/409/422`.

---

## 5. Docs & DX

- Swagger/OpenAPI en `/docs`, ReDoc en `/redoc`.
- Ejemplos en Pydantic.
- Respuestas tipadas (`ResponseModel`), códigos HTTP correctos.

---

## 6. Scripts de arranque

- `make dev` → `uvicorn app.main:app --reload`.
- Seed inicial (cursos + estudiantes).
- `.env` con: `SECRET_KEY`, `ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `DATABASE_URL`.

---

## 7. Arquitectura de carpetas

```
app/
  main.py
  core/    (config, seguridad/JWT)
  db/      (session, base, init, seed)
  models/  (ORM)
  schemas/ (Pydantic)
  api/     (routers: auth, cursos, estudiantes)
  services/(opcional)
  tests/

data/
  kawaii.db
.env.example
requirements.txt
```

---

## 8. Criterios de “listo”

- CRUD completo funcionando.
- Auth JWT operativa.
- Índices/constraints en DB.
- Swagger con ejemplos.
- Documentación de ejecución y prueba.

---

## Entradas y decisiones

- Si falta algo, proponer opciones y avanzar.
- Tipos estrictos en Pydantic (emails, nota 0–10).
- Paginación con `limit`/`offset`.

---

## Entregables

1. **Estructura de proyecto** creada.
2. Código de `db/`, `models/`, `schemas/`.
3. `api/auth.py`, `api/cursos.py`, `api/estudiantes.py`.
4. Config JWT en `core/security.py`.
5. `main.py` con routers, CORS y metadata Swagger.
6. `README.md` corto con instalación, `.env`, cómo correr y probar.

---

## Datos iniciales (seed sugerido)

- Users: `admin@example.com / admin123` (rol admin), `miyu@example.com / miyu123` (rol user).
- Cursos: SQL 101 (20h), Python (30h), Data Science (40h).
- Estudiantes: 5–10 ejemplos variados.

---

## Al final quiero

- Bloque de comandos para levantar server y probar 3 requests felices y 2 de error (ej: email duplicado → `409 Conflict`).
- Mini checklist de verificación (auth, CRUD, docs, validaciones).

---

## .env.example

```
SECRET_KEY=super_kawaii_secret
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
DATABASE_URL=sqlite:///data/kawaii.db
```

---

## requirements.txt sugerido

```
fastapi
uvicorn
sqlalchemy
passlib[bcrypt]
python-jose[cryptography]
pydantic[email]
python-dotenv
pandas
httpx
pytest
```

---

✨ Dale Shiro, arrancá con el **scaffolding** y después metemos **auth + CRUD** ✦
