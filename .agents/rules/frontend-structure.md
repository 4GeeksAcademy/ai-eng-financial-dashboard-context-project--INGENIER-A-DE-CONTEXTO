# Frontend: estructura y datos

## Nombre
Convenciones de estructura y estados del dashboard

## Alcance
Aplica a `frontend/src/`, en particular `App.tsx`, `lib/financial-utils.ts`, los tipos financieros y los componentes del dashboard.

## Justificación
`App.tsx` solicita movimientos a `/api/metrics` y `financial-utils.ts` calcula KPIs y datos mensuales, aunque el backend también ofrece endpoints agregados. El periodo visible está fijado como `2024 - Full Year`, aunque el backend genera fechas respecto de la fecha actual. Además, el flujo actual reemplaza cualquier causa del fallo por el mismo mensaje y no permite reintentar.

## Guía específica del proyecto
- Mantén la petición y la coordinación de estados en la capa de aplicación, y los cálculos puros en `src/lib/financial-utils.ts`. Por ejemplo, `App.tsx` llama `fetchFinancialData()` y entrega los movimientos a `computeKPIs()` y `computeMonthlyData()`; no copies esas fórmulas en componentes de gráficos.
- Antes de añadir cálculos, elige una fuente de verdad: los movimientos de `/api/metrics` agregados en cliente o los endpoints agregados del backend. La pantalla actual usa la primera opción; evita mezclar resultados de ambas sin una regla definida.
- Deriva el periodo mostrado de las fechas que recibe o calcula la aplicación, o márcalo claramente como dato de muestra. No uses `period="2024 - Full Year"` para datos generados dinámicamente que no estén filtrados a 2024.
- Conserva información suficiente para diagnosticar fallos de red y HTTP y presenta un estado recuperable. En `App.tsx`, el `catch` actual muestra un mensaje común y el efecto no ofrece reintento; separa el mensaje de usuario de la causa técnica y permite volver a solicitar los datos.
- Usa `import.meta.env.VITE_API_BASE_URL` para el origen configurable, como ya hace `App.tsx`, y coordínalo con el proxy `/api` de Vite. El valor predeterminado vacío funciona cuando el proxy resuelve el backend; no presupongas que el host `backend` es accesible al ejecutar Vite fuera de Compose.

Ejemplo real de cálculo puro en `frontend/src/lib/financial-utils.ts`:

```ts
const profit = totalIncome - totalOutcome;
const profitPercent = totalIncome > 0 ? (profit / totalIncome) * 100 : 0;

return { totalIncome, totalOutcome, profit, profitPercent };
```