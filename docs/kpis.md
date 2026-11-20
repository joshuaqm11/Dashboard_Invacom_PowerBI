# KPIs gerenciales aprobados — DW Invacom

Fuente: *Propuesta de KPIs – DW Invacom* (aprobada por el profesor).
Línea base (X): año 2017 completo. Meta (Y): enero–agosto 2018 (fuera 2016 y septiembre 2018 por datos incompletos). Periodicidad: mensual.
Perspectivas del Balanced Scorecard: financiera, procesos y clientes.

| Código | KPI | Responsable | Perspectiva | Tipo | Línea base 2017 | Meta 2018 | Verde | Amarillo | Rojo | Mejor si |
|---|---|---|---|---|---|---|---|---|---|---|
| KPI-VEN-01 | Ventas promedio mensuales | Joshua | Financiera | Lagging | 590,881 BRL | 1,000,000 BRL | ≥ 1,000,000 | 800,000 – 999,999 | < 800,000 | Mayor |
| KPI-VEN-02 | Órdenes promedio mensuales | Joshua | Financiera | Leading | 3,698 | 6,500 | ≥ 6,500 | 5,500 – 6,499 | < 5,500 | Mayor |
| KPI-FIN-01 | Ticket promedio por orden | Fernanda | Financiera | Lagging | 160.23 BRL | 170.00 BRL | ≥ 170 | 155 – 169.99 | < 155 | Mayor |
| KPI-FIN-02 | Peso del flete sobre la venta | Fernanda | Financiera | Leading | 13.85 % | ≤ 13.00 % | ≤ 13.00 % | 13.01 – 14.50 % | > 14.50 % | Menor |
| KPI-LOG-01 | % de entregas a tiempo | Fabio | Procesos | Leading | 94.35 % | 95.00 % | ≥ 95 % | 92 – 94.99 % | < 92 % | Mayor |
| KPI-LOG-02 | Días promedio de entrega | Fabio | Procesos | Leading | 12.54 días | ≤ 10.00 días | ≤ 10 | 10.01 – 12.50 | > 12.50 | Menor |
| KPI-CLI-01 | % de reseñas de 4 y 5 estrellas | Joseph | Clientes | Lagging | 77.80 % | 80.00 % | ≥ 80 % | 76 – 79.99 % | < 76 % | Mayor |
| KPI-CLI-02 | Tasa de cancelación de órdenes | Joseph | Clientes | Lagging | 0.458 % | ≤ 0.400 % | ≤ 0.40 % | 0.41 – 0.50 % | > 0.50 % | Menor |

## Fórmulas aprobadas

| Código | Fórmula | Columnas del DW |
|---|---|---|
| KPI-VEN-01 | Suma de `monto_total` (órdenes no canceladas ni no disponibles) ÷ número de meses | `FACT_VENTAS[monto_total]`, `DIM_ESTADO_ORDEN`, `DIM_TIEMPO` |
| KPI-VEN-02 | Órdenes distintas no canceladas ÷ número de meses | `FACT_VENTAS[order_id]`, `DIM_ESTADO_ORDEN`, `DIM_TIEMPO` |
| KPI-FIN-01 | Suma de `monto_total` ÷ órdenes distintas (todas las órdenes) | `FACT_VENTAS[monto_total]`, `FACT_VENTAS[order_id]` |
| KPI-FIN-02 | Suma de `flete` ÷ suma de `monto_total` × 100 (no canceladas) | `FACT_VENTAS[flete]`, `FACT_VENTAS[monto_total]`, `DIM_CLIENTE[region]` |
| KPI-LOG-01 | Órdenes con `entregado_a_tiempo = 1` ÷ órdenes entregadas × 100 | `FACT_VENTAS[entregado_a_tiempo]`, `DIM_CLIENTE` |
| KPI-LOG-02 | Promedio de `dias_entrega` por orden entregada | `FACT_VENTAS[dias_entrega]` |
| KPI-CLI-01 | Órdenes con `review_score ≥ 4` ÷ órdenes con reseña × 100 | `FACT_VENTAS[review_score]`, `DIM_PRODUCTO` |
| KPI-CLI-02 | Órdenes con estado *Cancelada* ÷ total de órdenes × 100 | `FACT_VENTAS[order_id]`, `DIM_ESTADO_ORDEN[estado_orden_es]` |

## Validación contra la línea base

Cada responsable filtra el año 2017 en Power BI y anota el valor que da su medida. Debe coincidir con la línea base.

| Código | Línea base aprobada | Valor en Power BI (2017) | ¿Coincide? | Captura |
|---|---|---|---|---|
| KPI-VEN-01 | 590,881 BRL | 590,881 BRL | Sí | `evidencias/validacion_kpis_ventas_2017.png` |
| KPI-VEN-02 | 3,698 | 3,698 | Sí | `evidencias/validacion_kpis_ventas_2017.png` |
| KPI-FIN-01 | 160.23 BRL | | | |
| KPI-FIN-02 | 13.85 % | | | |
| KPI-LOG-01 | 94.35 % | | | |
| KPI-LOG-02 | 12.54 días | | | |
| KPI-CLI-01 | 77.80 % | | | |
| KPI-CLI-02 | 0.458 % | | | |