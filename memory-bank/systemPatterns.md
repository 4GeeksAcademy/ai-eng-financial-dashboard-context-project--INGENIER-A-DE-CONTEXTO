# Patrones del sistema

## Arquitectura actual
El repositorio separa una SPA React/TypeScript en `frontend/` y una API FastAPI en `backend/`. `backend/app/main.py` crea la aplicación, configura CORS e incluye el router de `backend/app/routes.py`. En ese router conviven modelos, generación de datos, funciones de filtrado/agregación y handlers HTTP.

## Flujo de datos del dashboard
1. `frontend/src/App.tsx` ejecuta `fetchFinancialData()` al montar el componente.
2. La función solicita `GET ${VITE_API_BASE_URL}/api/metrics` (el prefijo es vacío por defecto) y convierte la respuesta JSON a movimientos financieros.
3. El frontend pasa esos movimientos a `computeKPIs()` y `computeMonthlyData()` de `frontend/src/lib/financial-utils.ts`.
4. `KPIRow` y los componentes `IncomeOutcomeChart` y `ProfitPercentChart` presentan las métricas y series mensuales.

## API y entorno
- `backend/app/routes.py` implementa `/health` y rutas bajo `/api/metrics`, incluidas métricas, facetas, resúmenes, comparación, categorías, alertas y segmentos B2B/B2C.
- Los handlers invocan `generate_mock_movements(seed=42)` y aplican filtros o agregaciones en memoria; no hay una capa de persistencia configurada.
- El backend también implementa varios agregados que no consume el dashboard actual, que agrega `/api/metrics` en el cliente. Por tanto, las dos ubicaciones existen; el repositorio no establece una única fuente compartida para todos los consumidores.
- `frontend/vite.config.ts` redirige `/api` a `http://backend:8000`, nombre de host de la red de Compose. `App.tsx` permite sustituir el origen mediante `VITE_API_BASE_URL`.
- `docker-compose.yml` configura `depends_on` del frontend respecto del backend, pero no define `healthcheck`; el orden de arranque no expresa una comprobación de disponibilidad HTTP.