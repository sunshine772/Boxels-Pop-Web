# Boxels-Pop-Server

POP backend service. A FastAPI REST API backed by a MySQL database via SQLAlchemy, providing the data and authentication layer for the POP application.

## Stack
- Python / FastAPI (ASGI, served with Uvicorn)
- SQLAlchemy ORM with PyMySQL driver (MySQL)
- Alembic for database migrations
- JWT authentication (python-jose / PyJWT) with passlib[bcrypt] password hashing
- Pydantic schemas, python-dotenv for configuration

## Layout
- `app/api/v1` — API entry point (`main.py`) and routers
- `app/models`, `app/schemas`, `app/crud` — ORM models, Pydantic schemas, data access
- `app/auth`, `app/dependencies.py` — authentication and shared dependencies
- `app/database`, `app/utils` — DB session setup and helpers

## Run
```
pip install -r requirements.txt
uvicorn app.api.v1.main:app --reload
```
Interactive API docs are served at `http://localhost:8000/docs`.

## Status
Active early-stage backend. Last pushed 2024-05-30.
