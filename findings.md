# Hallazgos de arquitectura y calidad

Revisión estática del frontend, backend, pruebas y configuración disponible. Las prioridades indican riesgo en el estado actual o al evolucionar el proyecto; no representan una auditoría de seguridad ni validan comportamiento en ejecución.

## Arquitectura

### A1. La lógica de agregación tiene dos dueños potenciales — Media

[frontend/src/App.tsx](frontend/src/App.tsx) descarga los movimientos de `/api/metrics` y [frontend/src/lib/financial-utils.ts](frontend/src/lib/financial-utils.ts) calcula KPIs y series mensuales en el navegador. A la vez, [backend/app/routes.py](backend/app/routes.py) implementa `/api/metrics/summary`, comparación, categorías principales y alertas. La pantalla actual no consume esos endpoints agregados.

**Riesgo:** pueden divergir las definiciones de margen, periodos o filtros entre consumidores y API; también se mantiene lógica de agregación no usada desde esta vista.

**Regla asociada:** cada métrica debe tener una única capa responsable. Elegir explícitamente entre agregación de servidor o cliente y probar el contrato y los resultados en esa capa; evitar mantener rutas agregadas paralelas sin consumidor.

### A2. La capa de rutas acumula responsabilidades — Media

[backend/app/routes.py](backend/app/routes.py) declara modelos Pydantic, genera movimientos aleatorios, filtra, agrega y expone handlers HTTP en el mismo módulo. Los datos se construyen en memoria con una semilla fija; [docker-compose.yml](docker-compose.yml) no configura una base de datos.

**Riesgo:** al sustituir los datos simulados o ampliar los casos de negocio, lógica de dominio y transporte quedan acoplados; además, el prototipo no conserva cambios entre solicitudes.

**Regla asociada:** mantener los handlers como adaptadores HTTP y separar generación/acceso a datos y cálculos de dominio cuando se añada persistencia o crezca la API; documentar explícitamente si un flujo usa datos simulados o persistidos.

### A3. CORS permite cualquier origen junto con credenciales — Media

[backend/app/main.py](backend/app/main.py) configura `allow_origins=["*"]` y `allow_credentials=True` para la aplicación completa.

**Riesgo:** si la API se expone fuera del desarrollo local o se incorporan cookies/credenciales, la política queda más amplia de lo necesario.

**Regla asociada:** definir orígenes permitidos mediante configuración por entorno y no habilitar credenciales junto a una política abierta; revisar CORS cuando se publique la API o se añada autenticación.

## Naming

### N1. El periodo mostrado no se deriva de los datos — Media

[frontend/src/App.tsx](frontend/src/App.tsx) pasa `"2024 - Full Year"` al encabezado. En cambio, [backend/app/routes.py](backend/app/routes.py) genera fechas según la fecha actual mediante `_year_for_month`; no aplica un filtro que limite los datos a 2024.

**Riesgo:** el nombre visible del periodo puede describir datos distintos a los que realmente presentan las cifras.

**Regla asociada:** etiquetas de periodo deben derivarse del rango recibido/calculado o identificarse expresamente como datos de muestra; evitar fijar un año que el origen no garantiza.

## Error Handling

### E1. El frontend descarta el detalle del fallo y no ofrece recuperación — Media

[frontend/src/App.tsx](frontend/src/App.tsx) convierte cualquier rechazo de `fetchFinancialData` en el mismo mensaje y no conserva el error original ni permite reintentar. En [docker-compose.yml](docker-compose.yml), `depends_on` solo ordena el inicio del backend; no hay `healthcheck` que espere a que la API responda.

**Riesgo:** un error HTTP, un fallo de red o el arranque tardío de la API resultan indistinguibles para diagnóstico; el intento único puede dejar el dashboard sin datos.

**Regla asociada:** preservar la causa técnica en logs de desarrollo, presentar un estado de error recuperable en UI y definir reintento o readiness cuando el cliente dependa de un servicio que puede arrancar más tarde.

### E2. El cálculo de facetas presupone una lista no vacía — Media

En [backend/app/routes.py](backend/app/routes.py), `build_metrics_facets` indexa `ordered[0]` y `ordered[-1]` para establecer fechas. La generación actual del endpoint crea 360 registros, pero la función no define qué hacer con una fuente vacía.

**Riesgo:** si el origen de datos cambia o devuelve cero registros, la función falla con `IndexError` en vez de representar un resultado vacío de forma controlada.

**Regla asociada:** validar precondiciones de colecciones antes de indexarlas y acordar una respuesta vacía explícita; cubrir esa rama con una prueba unitaria.

## Testing

### T1. Las pruebas del frontend no ejercitan el flujo de la aplicación — Media

[frontend/src/lib/financial-utils.test.ts](frontend/src/lib/financial-utils.test.ts) prueba KPIs, agregación mensual y formato, pero no la petición de [frontend/src/App.tsx](frontend/src/App.tsx), sus estados de carga/error ni la integración con las rutas del backend. [backend/tests/test_routes.py](backend/tests/test_routes.py) sí cubre endpoints mediante `TestClient`.

**Riesgo:** regresiones en URL/proxy, estados de UI o forma de la respuesta pueden pasar aunque las funciones matemáticas sigan aprobadas.

**Regla asociada:** mantener unit tests para cálculos y añadir pruebas de integración acotadas para la solicitud HTTP, respuesta válida, rechazo HTTP y error de red; validar al menos el contrato que consume el dashboard.

## DX

### D1. La configuración de entorno descrita en el README no está completa — Media

[README.es.md](README.es.md) instruye copiar `frontend/.env.example`, pero ese archivo no está presente en el repositorio. [frontend/src/App.tsx](frontend/src/App.tsx) sí lee `VITE_API_BASE_URL`; [frontend/vite.config.ts](frontend/vite.config.ts) configura el proxy `/api` hacia `http://backend:8000`, nombre de host propio de la red de Compose.

**Riesgo:** la alternativa de configuración indicada por la documentación no se puede seguir tal cual; ejecutar Vite fuera de Compose tampoco resuelve automáticamente el host `backend`.

**Regla asociada:** toda variable de entorno documentada debe tener plantilla y valor de ejemplo no secreto; documentar por separado cómo conectar el proxy al backend dentro y fuera de Compose, y verificar ambos comandos.

### D2. La imagen instala dependencias sin lockfile — Baja

[frontend/Dockerfile](frontend/Dockerfile) ejecuta `npm install`; no hay `package-lock.json`, `npm-shrinkwrap.json`, `pnpm-lock.yaml` ni `yarn.lock` en `frontend/`.

**Riesgo:** la resolución de versiones transitivas puede variar entre builds, reduciendo reproducibilidad y dificultando comparar fallos entre máquinas.

**Regla asociada:** elegir un gestor de paquetes, versionar su lockfile y usar instalación determinista (`npm ci` si se adopta npm con `package-lock.json`) en imágenes y CI.
