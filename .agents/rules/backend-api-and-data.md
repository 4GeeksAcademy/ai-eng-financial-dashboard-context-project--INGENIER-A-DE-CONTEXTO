# Backend: API y datos

## Nombre
Convenciones de API, dominio y datos financieros

## Alcance
Aplica a `backend/app/`, especialmente a los modelos, cálculos, filtros y endpoints de `backend/app/routes.py`, así como a la configuración común de la aplicación en `backend/app/main.py`.

## Justificación
Actualmente `routes.py` define modelos Pydantic, genera movimientos simulados, filtra y agrega datos, y también contiene handlers HTTP. Además, el backend expone agregados mientras el dashboard calcula sus propios KPIs y series. Sin un dueño explícito de cada métrica, las definiciones pueden divergir. Los datos se generan en memoria y no se persisten.

## Guía específica del proyecto
- Define una sola capa como responsable de cada métrica consumida por una pantalla. Antes de extender un cálculo existente, decide si el dashboard consumirá los agregados de `/api/metrics/summary` y rutas relacionadas o si seguirá agregando los movimientos de `/api/metrics`. No dupliques una métrica sin documentar el motivo y probar que ambos contratos coinciden.
- Conserva los handlers HTTP como adaptadores. `routes.py` actualmente mezcla `generate_mock_movements`, `summarize_movements` y los endpoints; si se incorpora persistencia o crece el dominio, mueve generación/acceso a datos y cálculos a módulos propios y deja los handlers encargados de validar parámetros y delegar.
- Identifica explícitamente los datos simulados. `generate_mock_movements(seed=42)` produce 360 movimientos para los tests, mientras la API usa generación en memoria; no describas esos datos como persistidos ni asumas que cambios sobreviven a otra solicitud.
- Define el resultado para entradas vacías antes de indexar listas. `build_metrics_facets` usa `ordered[0]` y `ordered[-1]`; si el origen puede estar vacío, responde con una forma vacía acordada o un error de dominio claro, en vez de permitir un `IndexError` incidental.
- Configura CORS por entorno antes de desplegar o habilitar autenticación. `main.py` combina `allow_origins=["*"]` y `allow_credentials=True`; usa orígenes explícitos para cada entorno y no combines credenciales con una política abierta.
- Mantén y prueba el contrato HTTP. Por ejemplo, `test_metrics_summary_by_month_returns_balances` verifica que el resumen mensual exponga `period`, `income`, `outcome` y `net`; actualiza consumidores y pruebas en el mismo cambio si ese contrato cambia.

Ejemplo real de cálculo en `backend/app/routes.py`:

```python
MetricsSummaryItem(
    period=period,
    income=round(values["income"], 2),
    outcome=round(values["outcome"], 2),
    net=round(values["income"] - values["outcome"], 2),
)
```