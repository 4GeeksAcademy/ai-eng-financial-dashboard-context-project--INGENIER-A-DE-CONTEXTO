# Contexto tecnológico

## Frontend
- React y React DOM `19.2.x`, TypeScript `~6.0.2` y Vite `^8.0.4` (`frontend/package.json`).
- Visualización con Recharts `^3.8.1`; Tailwind CSS `^4.2.2` y el plugin `@tailwindcss/vite`; iconos con `lucide-react`.
- Pruebas frontend con Vitest `^4.1.4`; scripts disponibles: `dev`, `build`, `lint`, `test`, `test:watch` y `test:coverage`.

## Backend
- Python `3.13` en la imagen (`backend/Dockerfile`), FastAPI y Uvicorn (`backend/requirements.txt`).
- La API usa modelos Pydantic en `backend/app/routes.py`. Para depuración incluye `debugpy`; las dependencias de pruebas incluyen `pytest`, `pytest-cov` y `httpx`.

## Persistencia
El repositorio no declara ni configura una base de datos: `docker-compose.yml` define solo los servicios `frontend` y `backend`, `backend/requirements.txt` no lista un controlador o ORM de base de datos y la API genera movimientos simulados en memoria. Esto describe la configuración versionada, no sistemas externos que pudieran existir fuera del repositorio.

## Docker y ejecución
- `docker-compose.yml` publica el frontend en `5173`, el backend en `8000` y el puerto de depuración `5678`; monta el código local en los contenedores.
- La imagen frontend usa `node:24-alpine`, instala dependencias con `npm install` y ejecuta Vite (`frontend/Dockerfile`). No hay lockfile npm versionado en `frontend/`.
- La imagen backend usa `python:3.13-slim`, instala `requirements.txt` y ejecuta Uvicorn con recarga y `debugpy` (`backend/Dockerfile`).
- El comando documentado para iniciar ambos servicios es `docker compose up --build` (`README.es.md`).