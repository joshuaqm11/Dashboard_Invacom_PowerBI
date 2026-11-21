# Metodología: criterios S.M.A.R.T. e indicadores leading y lagging

Parte de Joseph (Clientes) para el documento final. Se complementa con la sección de Balanced Scorecard y fichas de KPI (Fernanda).
Fuente de los valores: *Propuesta de KPIs – DW Invacom* y `docs/kpis.md`. Línea base (X): año 2017. Meta (Y): enero–agosto 2018.

## 1. Criterios S.M.A.R.T.

Un KPI bien definido cumple cinco criterios (Doran, 1981; Parmenter, 2015):

| Letra | Criterio | Qué exige | Cómo se aplicó en el proyecto |
|---|---|---|---|
| **S** | Específico (*Specific*) | El indicador mide una sola cosa, sin ambigüedad, con una fórmula y una fuente definidas. | Cada KPI tiene un código (KPI-VEN-01 … KPI-CLI-02), una fórmula única y las columnas exactas del DW (`FACT_VENTAS`, `DIM_ESTADO_ORDEN`, `DIM_TIEMPO`…). Ej.: CLI-01 cuenta **órdenes únicas** con `review_score ≥ 4` sobre órdenes con reseña, no filas de ítems. |
| **M** | Medible (*Measurable*) | Se calcula con datos disponibles y da un número con unidad. | Los 8 KPIs salen de una medida DAX y se validaron contra la línea base 2017 (por ejemplo 77.80 % y 0.458 % en Clientes). Unidades: BRL, órdenes, %, días. |
| **A** | Alcanzable (*Achievable*) | La meta es retadora pero realista frente al punto de partida. | Las metas se fijaron a partir de la línea base 2017: CLI-01 sube 2.2 puntos (77.80 % → 80.00 %); CLI-02 baja de 0.458 % a ≤ 0.400 %. El rango amarillo da margen entre la base y la meta. |
| **R** | Relevante (*Relevant*) | El indicador se relaciona con un objetivo estratégico y dispara decisiones. | Cada KPI pertenece a una perspectiva del Balanced Scorecard (financiera, procesos, clientes) y tiene un responsable y un semáforo que indica qué hacer (verde mantener, amarillo vigilar, rojo plan correctivo). |
| **T** | Temporal (*Time-bound*) | Tiene un período de medición y un plazo para cumplir la meta. | Periodicidad mensual (comparación con el mes anterior en cada página) y plazo fijo: enero–agosto 2018. Se excluyen 2016 y septiembre 2018 por datos incompletos (filtro `ANIO_MES` 2017-01 a 2018-08 en todo el informe). |

### Aplicación a los 8 KPIs

| Código | KPI | S (fórmula) | M (base 2017) | A (meta) | R (perspectiva) | T |
|---|---|---|---|---|---|---|
| KPI-VEN-01 | Ventas promedio mensuales | Ventas válidas ÷ meses con ventas | 590,881 BRL | 1,000,000 BRL | Financiera | Mensual; ene–ago 2018 |
| KPI-VEN-02 | Órdenes promedio mensuales | Órdenes válidas ÷ meses con ventas | 3,698 | 6,500 | Financiera | Mensual; ene–ago 2018 |
| KPI-FIN-01 | Ticket promedio por orden | Monto total ÷ órdenes distintas | 160.23 BRL | 170.00 BRL | Financiera | Mensual; ene–ago 2018 |
| KPI-FIN-02 | Peso del flete sobre la venta | Flete ÷ ventas válidas | 13.85 % | ≤ 13.00 % | Financiera | Mensual; ene–ago 2018 |
| KPI-LOG-01 | % de entregas a tiempo | Órdenes a tiempo ÷ órdenes entregadas | 94.35 % | 95.00 % | Procesos | Mensual; ene–ago 2018 |
| KPI-LOG-02 | Días promedio de entrega | Promedio de días por orden entregada | 12.54 días | ≤ 10.00 días | Procesos | Mensual; ene–ago 2018 |
| KPI-CLI-01 | % de reseñas de 4 y 5 estrellas | Órdenes con reseña ≥ 4 ÷ órdenes con reseña | 77.80 % | 80.00 % | Clientes | Mensual; ene–ago 2018 |
| KPI-CLI-02 | Tasa de cancelación de órdenes | Órdenes canceladas ÷ total de órdenes | 0.458 % | ≤ 0.400 % | Clientes | Mensual; ene–ago 2018 |

## 2. Indicadores leading y lagging

- **Leading (predictivos o de causa):** miden las actividades o condiciones que ocurren *antes* del resultado y lo influyen. Miran hacia adelante y permiten corregir el rumbo a tiempo, mientras el período todavía está abierto.
- **Lagging (históricos o de efecto):** miden el resultado ya consolidado al cerrar un período. Confirman si se logró el objetivo, pero cuando se conocen ya no se puede cambiar lo ocurrido.

Los dos tipos se usan juntos: los leading avisan temprano y los lagging confirman el resultado (Kaplan & Norton, 1996).

### Clasificación de los KPIs de la propuesta

| Código | KPI | Tipo | Por qué |
|---|---|---|---|
| KPI-VEN-01 | Ventas promedio mensuales | **Lagging** | Es el resultado financiero final del mes. |
| KPI-VEN-02 | Órdenes promedio mensuales | **Leading** | Las ventas ≈ órdenes × ticket: si bajan las órdenes, después bajan las ventas. |
| KPI-FIN-01 | Ticket promedio por orden | **Lagging** | Resultado de lo que ya compró cada cliente. |
| KPI-FIN-02 | Peso del flete sobre la venta | **Leading** | Un flete caro desincentiva la compra y presiona el margen antes de que se note en las ventas. |
| KPI-LOG-01 | % de entregas a tiempo | **Leading** | La puntualidad de la entrega anticipa la satisfacción del cliente (reseñas). |
| KPI-LOG-02 | Días promedio de entrega | **Leading** | Igual que LOG-01: es una condición del proceso que influye en la experiencia posterior. |
| KPI-CLI-01 | % de reseñas de 4 y 5 estrellas | **Lagging** | La reseña se escribe *después* de recibir el pedido: refleja el efecto de la entrega y del producto. |
| KPI-CLI-02 | Tasa de cancelación de órdenes | **Lagging** | La cancelación es el desenlace final de una orden que ya ocurrió. |

**Relación entre los KPIs de Clientes y Logística.** El árbol de descomposición de la página Clientes abre las reseñas negativas por estado de la entrega (`FACT_VENTAS[Entrega]`): la mayor parte de las reseñas de 1 y 2 estrellas viene de órdenes con atraso. Eso muestra la cadena causa → efecto: LOG-01 y LOG-02 (leading) se mueven primero y CLI-01 (lagging) refleja el resultado después.

## Referencias

Doran, G. T. (1981). There's a S.M.A.R.T. way to write management's goals and objectives. *Management Review, 70*(11), 35–36.

Kaplan, R. S., & Norton, D. P. (1996). *The balanced scorecard: Translating strategy into action*. Harvard Business School Press.

Parmenter, D. (2015). *Key performance indicators: Developing, implementing, and using winning KPIs* (3.ª ed.). John Wiley & Sons.

Universidad Técnica Nacional. (2026). *Minería de datos – 04 – KPIs* [Presentación de clase]. Curso ISW-911, Sede Regional de San Carlos.
