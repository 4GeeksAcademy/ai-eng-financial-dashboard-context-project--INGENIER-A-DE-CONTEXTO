# Estado del proyecto

## Estado observado
El código contiene una aplicación frontend que muestra KPI y gráficos mensuales, y una API FastAPI con endpoints de movimientos y agregados. Hay pruebas backend con `TestClient` en `backend/tests/test_routes.py` y pruebas unitarias frontend para los cálculos y formatos en `frontend/src/lib/financial-utils.test.ts`.

La presencia de código y pruebas no demuestra que los contenedores arranquen ni que las suites pasen en el entorno actual; no se ejecutaron pruebas como parte de esta actualización del memory bank.

## Deuda técnica evidenciada
- La pantalla calcula KPI y series en el cliente, mientras el backend también expone agregados (`frontend/src/lib/financial-utils.ts`, `backend/app/routes.py`).
- Los endpoints generan datos simulados en memoria; no hay base de datos configurada.
- El periodo de la cabecera es fijo, aunque las fechas de los datos se generan respecto de la fecha actual (`frontend/src/App.tsx`, `backend/app/routes.py`).
- El frontend sustituye el fallo por un mensaje común y no ofrece reintento; las pruebas frontend existentes no cubren la petición ni los estados de `App.tsx`.
- `build_metrics_facets` indexa el primer y último elemento sin definir el caso de lista vacía (`backend/app/routes.py`).
- CORS está configurado con `allow_origins=["*"]` y `allow_credentials=True` (`backend/app/main.py`).
- README.es.md menciona `frontend/.env.example`, que no está en el repositorio; Compose no configura readiness y el frontend instala dependencias sin lockfile (`docker-compose.yml`, `frontend/Dockerfile`).

## Prioridades inmediatas derivadas de la evidencia
1. Elegir y documentar la capa responsable de cada métrica usada por el dashboard antes de ampliar agregaciones.
2. Hacer que la etiqueta de periodo corresponda con el rango real y definir recuperación visible ante fallos de carga.
3. Definir respuesta y prueba para facetas vacías; ampliar pruebas del flujo frontend para petición exitosa y errores HTTP/red.
4. Alinear la plantilla/documentación de `VITE_API_BASE_URL` con la ejecución dentro y fuera de Compose; decidir si se añade readiness.
5. Si se mantiene npm, versionar un lockfile y usar instalación reproducible; restringir CORS por entorno antes de publicar la API o añadir credenciales.