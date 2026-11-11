# Contexto del producto

## Visión
El repositorio presenta un panel de métricas financieras. El README lo describe como un dashboard; la interfaz lo titula `Financial Overview` y `Executive metrics dashboard` (`frontend/src/components/dashboard/dashboard-header.tsx`). No hay en el repositorio una especificación adicional de usuarios o requisitos de producto.

## Funcionalidad implementada
- `frontend/src/App.tsx` solicita movimientos a `GET /api/metrics` y calcula en el navegador los KPI y los datos mensuales.
- La fila de KPI muestra ingresos totales, egresos totales, beneficio (ingresos menos egresos) y margen de beneficio (`frontend/src/components/dashboard/kpi-row.tsx`).
- El dashboard incluye una serie mensual de ingresos frente a egresos y otra de margen de beneficio (`frontend/src/components/dashboard/income-outcome-chart.tsx`, `frontend/src/components/dashboard/profit-percent-chart.tsx`). Ambos gráficos muestran carga y un estado vacío cuando no hay datos.
- La cabecera muestra un periodo fijo `2024 - Full Year`; el frontend no lo calcula a partir de las fechas devueltas por la API.
- El backend implementa endpoints adicionales para facetas, resúmenes, categorías principales, comparaciones, alertas y movimientos B2B/B2C (`backend/app/routes.py`). La pantalla actual solo solicita `/api/metrics`.

## Origen de datos
Los movimientos son simulados por `generate_mock_movements` en `backend/app/routes.py`. La función crea 360 movimientos y los endpoints la invocan con `seed=42`; no se observa almacenamiento persistente configurado en el repositorio. Estos datos no deben describirse como transacciones reales.