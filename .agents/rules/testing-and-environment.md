# Pruebas y entorno

## Nombre
Convenciones de pruebas, configuración y ejecución

## Alcance
Aplica a `backend/tests/`, las pruebas del frontend, `docker-compose.yml`, `frontend/Dockerfile`, la configuración de Vite y la documentación de ejecución del proyecto.

## Justificación
Las pruebas backend ejercitan rutas con `TestClient` y las pruebas frontend cubren cálculos puros, pero no el ciclo de petición/estados de `App.tsx`. Compose ordena iniciar el backend antes del frontend, sin comprobar disponibilidad. El README español menciona `frontend/.env.example`, archivo que no existe; Vite apunta al host de Compose `backend`. La imagen del frontend ejecuta `npm install` y no hay lockfile versionado.

## Guía específica del proyecto
- Mantén pruebas unitarias para cálculos y pruebas de contrato para la API. El patrón backend existente crea `client = TestClient(app)` y verifica, por ejemplo, los campos de `/api/metrics/summary`; conserva aserciones sobre la forma y los filtros que usan los consumidores.
- Para el flujo del dashboard, cubre de forma acotada respuesta válida, respuesta HTTP fallida y error de red, incluyendo los estados de carga y error. Las pruebas actuales de `financial-utils.test.ts` validan cálculos y formato, pero no reemplazan pruebas de `fetchFinancialData()` o de la integración de `App.tsx`.
- No supongas que `depends_on` significa que la API está lista. `docker-compose.yml` solo ordena el inicio de `backend`; si el frontend necesita disponibilidad al arrancar, agrega una comprobación de salud/readiness y una estrategia de reintento coherente.
- Mantén sincronizados README, variables y configuración de Vite. `App.tsx` lee `VITE_API_BASE_URL` y `vite.config.ts` redirige `/api` a `http://backend:8000`, válido en la red de Compose. Si se documenta `frontend/.env.example`, versiona esa plantilla con valores no secretos y explica por separado cómo apuntar al backend fuera de Compose.
- Si se adopta npm, versiona `frontend/package-lock.json` y cambia la instalación de la imagen a `npm ci`; actualmente `frontend/Dockerfile` usa `npm install` y no hay lockfile, por lo que las dependencias transitivas pueden variar entre builds.

Ejemplo real de prueba backend en `backend/tests/test_routes.py`:

```python
response = client.get("/api/metrics/summary", params={"group_by": "month"})

assert response.status_code == 200
payload = response.json()
assert set(payload[0].keys()) == {"period", "income", "outcome", "net"}
```

Ejemplo real de proxy para desarrollo en `frontend/vite.config.ts`:

```ts
proxy: {
  "/api": {
    target: "http://backend:8000",
    changeOrigin: true,
  },
},
```