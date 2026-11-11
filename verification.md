# Verificación del repositorio

Inspección estática de los archivos del checkout. Se validó que `docker compose config` interpreta la configuración; no se levantaron los contenedores, así que las respuestas HTTP y la disponibilidad real de los servicios siguen pendientes de ejecución.

## Instrucciones de Ejecución

### Levantar los servicios

Desde la raíz del repositorio, con Docker y el plugin Compose disponibles:

```bash
docker compose up --build -d
docker compose ps
```

Compose construye dos servicios: `frontend` y `backend` ([docker-compose.yml](docker-compose.yml)). El frontend ejecuta Vite en `0.0.0.0:5173` ([frontend/Dockerfile](frontend/Dockerfile)); el backend ejecuta Uvicorn en `0.0.0.0:8000` y `debugpy` en `0.0.0.0:5678` ([backend/Dockerfile](backend/Dockerfile)). Compose publica esos mismos puertos como `5173`, `8000` y `5678`, respectivamente. `5678` es el puerto del depurador, no una URL HTTP.

| Servicio | URL / puerto | Qué comprobar |
| --- | --- | --- |
| Dashboard | http://localhost:5173 | Página servida por Vite |
| API de salud | http://localhost:8000/health | Esperado desde el código: `{"status":"ok"}` |
| API interactiva | http://localhost:8000/docs | Swagger UI generado por FastAPI |
| Movimientos | http://localhost:8000/api/metrics | Lista de movimientos simulados |
| Puerto de depuración | `localhost:5678` | `debugpy`; no es endpoint web |

Los puertos publicados están definidos en [docker-compose.yml](docker-compose.yml), las direcciones de escucha en ambos Dockerfiles y la ruta `/health` y rutas `/api/metrics...` en [backend/app/routes.py](backend/app/routes.py). El README documenta las URLs de frontend/API y la ejecución con Compose ([README.es.md](README.es.md)). La comprobación de salud puede hacerse con:

```bash
curl -fsS http://localhost:8000/health
curl -fsS http://localhost:8000/api/metrics/summary
```

La UI obtiene `/api/metrics` a través del proxy definido en [frontend/vite.config.ts](frontend/vite.config.ts), cuyo destino `http://backend:8000` corresponde al nombre de servicio de Compose. Para abrir el dashboard, visita http://localhost:5173 y comprueba que cargan los indicadores y gráficas; el frontend muestra un mensaje si falla la petición.

### Pruebas y parada

Los comandos de prueba se basan en scripts y dependencias declarados: backend con `pytest` ([backend/requirements.txt](backend/requirements.txt), [backend/tests/test_routes.py](backend/tests/test_routes.py)) y frontend con el script `test` de npm ([frontend/package.json](frontend/package.json), [frontend/src/lib/financial-utils.test.ts](frontend/src/lib/financial-utils.test.ts)). Después de levantar Compose:

```bash
docker compose exec backend pytest
docker compose exec frontend npm test
docker compose exec frontend npm run lint
docker compose exec frontend npm run build
docker compose down
```

Los scripts `test`, `lint` y `build` están declarados en [frontend/package.json](frontend/package.json); `build` ejecuta TypeScript y Vite. [frontend/tsconfig.app.json](frontend/tsconfig.app.json) comprueba el código de `src`, mientras [frontend/tsconfig.node.json](frontend/tsconfig.node.json) aplica al archivo de configuración de Vite. Los alias `@/` están configurados en [frontend/vite.config.ts](frontend/vite.config.ts), con alias de componentes/UI descritos en [frontend/components.json](frontend/components.json). `docker compose config` sí se ejecutó y terminó correctamente. Los comandos de pruebas/build/lint anteriores y la disponibilidad de los servicios no se ejecutaron durante esta inspección.

## Resumen del Proyecto

Aplicación de panel financiero con frontend React + TypeScript y API FastAPI, conforme a los README ([README.md](README.md), [README.es.md](README.es.md)). La entrada del navegador monta React en [frontend/src/main.tsx](frontend/src/main.tsx). [frontend/src/App.tsx](frontend/src/App.tsx) pide `GET /api/metrics`, mantiene estados de carga/error y entrega los movimientos a [frontend/src/lib/financial-utils.ts](frontend/src/lib/financial-utils.ts) para calcular ingresos, egresos, beneficio, margen y agregados mensuales.

La pantalla se compone del encabezado y la fila de KPIs ([frontend/src/components/dashboard/dashboard-header.tsx](frontend/src/components/dashboard/dashboard-header.tsx), [frontend/src/components/dashboard/kpi-row.tsx](frontend/src/components/dashboard/kpi-row.tsx), [frontend/src/components/dashboard/kpi-card.tsx](frontend/src/components/dashboard/kpi-card.tsx)) y dos gráficas mensuales ([frontend/src/components/dashboard/income-outcome-chart.tsx](frontend/src/components/dashboard/income-outcome-chart.tsx), [frontend/src/components/dashboard/profit-percent-chart.tsx](frontend/src/components/dashboard/profit-percent-chart.tsx)); estas usan Recharts, declarado en [frontend/package.json](frontend/package.json). Los tipos compartidos del frontend están en [frontend/src/lib/financial-types.ts](frontend/src/lib/financial-types.ts), y componentes visuales básicos en `frontend/src/components/ui/`.

FastAPI crea la aplicación y habilita CORS en [backend/app/main.py](backend/app/main.py); los modelos, la generación de movimientos y las rutas están en [backend/app/routes.py](backend/app/routes.py). La API contiene `GET /health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b` y `/api/metrics/b2c`. Los datos se generan en memoria con una semilla fija (`42`), 30 movimientos por mes (360 en total) y no provienen de una base de datos. El backend tiene pruebas de rutas en `backend/tests/`; el frontend prueba cálculos y formateadores.

## Tabla/Rastro de Verificación (✅ / ❌ / ❓)

| Estado | Afirmación | Evidencia / corrección |
| --- | --- | --- |
| ✅ Verificado | La aplicación se compone de frontend React/TypeScript y backend FastAPI. | README, dependencias de [frontend/package.json](frontend/package.json) y [backend/requirements.txt](backend/requirements.txt), y código de [backend/app/main.py](backend/app/main.py). |
| ✅ Verificado | Compose construye los servicios `frontend` y `backend` y publica `5173`, `8000` y `5678`. | [docker-compose.yml](docker-compose.yml); configuración también aceptada por `docker compose config`. No implica que los servicios se hayan iniciado. |
| ✅ Verificado | El contenedor frontend usa Vite; el backend usa Uvicorn con recarga y `debugpy`. | [frontend/Dockerfile](frontend/Dockerfile) y [backend/Dockerfile](backend/Dockerfile). |
| ✅ Verificado | FastAPI ofrece `/health`, documentación `/docs` y las rutas de métricas enumeradas arriba. | `/health` y rutas están en [backend/app/routes.py](backend/app/routes.py); FastAPI genera `/docs` desde [backend/app/main.py](backend/app/main.py). Hay pruebas de endpoints en [backend/tests/test_routes.py](backend/tests/test_routes.py). |
| ✅ Verificado | Los datos del endpoint son simulados, generados en backend, no datos persistidos. | `generate_mock_movements(seed=42)` y construcción de movimientos en [backend/app/routes.py](backend/app/routes.py); no hay servicio de base de datos en [docker-compose.yml](docker-compose.yml). |
| ✅ Verificado | La pantalla actual consulta `/api/metrics` y calcula métricas en el frontend. | `fetchFinancialData` en [frontend/src/App.tsx](frontend/src/App.tsx) y funciones en [frontend/src/lib/financial-utils.ts](frontend/src/lib/financial-utils.ts). |
| ✅ Verificado | Los tests frontend disponibles prueban cálculos y formateo; los tests backend cubren generación, filtros y endpoints. | [frontend/src/lib/financial-utils.test.ts](frontend/src/lib/financial-utils.test.ts) y [backend/tests/test_routes.py](backend/tests/test_routes.py). |
| ❌ Incorrecto / Corregido | El README español indica copiar `frontend/.env.example`, pero ese archivo no está presente en este checkout. | [README.es.md](README.es.md) lo referencia; la búsqueda de archivos `frontend/.env*` no encontró ninguno. La variable `VITE_API_BASE_URL` sí aparece leída en [frontend/src/App.tsx](frontend/src/App.tsx), pero no existe la plantilla prometida. |
| ❌ Incorrecto / Corregido | La etiqueta visible `2024 - Full Year` no acredita que los datos pertenezcan a 2024. | Se fija literalmente en [frontend/src/App.tsx](frontend/src/App.tsx) y [frontend/src/components/dashboard/dashboard-header.tsx](frontend/src/components/dashboard/dashboard-header.tsx). El backend genera meses relativos a la fecha actual mediante `_year_for_month` en [backend/app/routes.py](backend/app/routes.py). |
| ❌ Incorrecto / Corregido | La presencia de `mock-data.ts` no significa que la pantalla utilice ese arreglo. | [frontend/src/lib/mock-data.ts](frontend/src/lib/mock-data.ts) exporta `mockMovements`, pero [frontend/src/App.tsx](frontend/src/App.tsx) no lo importa: consulta la API. |
| ❓ Sin verificar | Que los servicios estén disponibles en las URLs anteriores y que pasen los tests en este entorno. | `docker compose config` solo valida la configuración; no se iniciaron contenedores ni se ejecutaron `curl`, `pytest` o `npm test`. Ejecutar los comandos de las instrucciones para verificarlo. |
| ❓ Sin verificar | Que endpoints del backend como comparación, alertas, facetas y B2B/B2C estén integrados a la interfaz. | Existen implementaciones y tests en [backend/app/routes.py](backend/app/routes.py) y [backend/tests/test_routes.py](backend/tests/test_routes.py), pero la petición de [frontend/src/App.tsx](frontend/src/App.tsx) solo usa `/api/metrics`. |
| ❓ Sin verificar | Que la configuración documentada de `VITE_API_BASE_URL` funcione en cada modo de ejecución. | El código lee la variable en [frontend/src/App.tsx](frontend/src/App.tsx), pero no hay `.env.example`; además, el proxy apunta al host de Compose `backend` en [frontend/vite.config.ts](frontend/vite.config.ts). Solo se comprobó la ruta Compose de forma estática. |
