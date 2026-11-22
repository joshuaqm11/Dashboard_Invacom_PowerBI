# Metodología: Balanced Scorecard y fichas técnicas de los KPIs

Parte de Fernanda (Finanzas) para el documento final. Se complementa con la sección de criterios S.M.A.R.T. e indicadores leading y lagging (Joseph, `metodologia_smart_leading_lagging.md`).
Fuente de los valores: *Propuesta de KPIs – DW Invacom* y `docs/kpis.md`. Línea base (X): año 2017. Meta (Y): enero–agosto 2018.

## 1. ¿Qué es el Balanced Scorecard?

El Balanced Scorecard (cuadro de mando integral) es un marco de gestión propuesto por Robert Kaplan y David Norton (1992) para traducir la estrategia de una organización en un conjunto equilibrado de indicadores. Su idea central es que los indicadores financieros, por sí solos, solo cuentan lo que ya pasó: son resultados del pasado. Para dirigir una empresa también hace falta medir los factores que **generan** esos resultados: lo que perciben los clientes, cómo funcionan los procesos internos y qué capacidades tiene la organización para mejorar (Kaplan & Norton, 1996).

Por eso el modelo organiza los indicadores en **cuatro perspectivas**, unidas por relaciones de causa y efecto: las capacidades mejoran los procesos, los buenos procesos satisfacen al cliente y un cliente satisfecho produce los resultados financieros (Kaplan & Norton, 2004).

## 2. Las cuatro perspectivas

| Perspectiva | Pregunta que responde | Qué suele medir |
|---|---|---|
| **Financiera** | ¿Cómo nos ven los dueños o accionistas? ¿La estrategia está dando resultados económicos? | Ingresos, crecimiento de ventas, rentabilidad, costos, margen. |
| **Clientes** | ¿Cómo nos ven los clientes? ¿Qué valor les entregamos? | Satisfacción, retención, quejas, cancelaciones, participación de mercado. |
| **Procesos internos** | ¿En qué procesos tenemos que ser excelentes para satisfacer al cliente? | Tiempos de ciclo, calidad, cumplimiento de entregas, productividad. |
| **Aprendizaje y crecimiento** | ¿Cómo mantenemos la capacidad de cambiar y mejorar? | Capacitación y competencias del personal, sistemas de información, cultura y clima. |

El término *balanced* (equilibrado) se refiere a que el tablero combina indicadores financieros y no financieros, internos y externos, y de causa (leading) y de efecto (lagging).

## 3. Ubicación de los 8 KPIs del tablero

| Código | KPI | Responsable | Perspectiva | Tipo | Justificación |
|---|---|---|---|---|---|
| KPI-VEN-01 | Ventas promedio mensuales | Joshua | **Financiera** | Lagging | Mide el ingreso: el resultado económico final del negocio. |
| KPI-VEN-02 | Órdenes promedio mensuales | Joshua | **Financiera** | Leading | Volumen de transacciones que genera el ingreso (ventas ≈ órdenes × ticket). |
| KPI-FIN-01 | Ticket promedio por orden | Fernanda | **Financiera** | Lagging | Valor económico de cada compra; con las órdenes explica las ventas. |
| KPI-FIN-02 | Peso del flete sobre la venta | Fernanda | **Financiera** | Leading | Costo logístico que trasladamos al cliente por cada real vendido; afecta precio final y margen. |
| KPI-LOG-01 | % de entregas a tiempo | Fabio | **Procesos internos** | Leading | Mide la ejecución del proceso de entrega frente a lo prometido. |
| KPI-LOG-02 | Días promedio de entrega | Fabio | **Procesos internos** | Leading | Mide la velocidad del proceso logístico. |
| KPI-CLI-01 | % de reseñas de 4 y 5 estrellas | Joseph | **Clientes** | Lagging | Satisfacción declarada por el cliente después de recibir el pedido. |
| KPI-CLI-02 | Tasa de cancelación de órdenes | Joseph | **Clientes** | Lagging | Pérdida de clientes en el proceso de compra. |

**Resumen:** 4 KPIs en la perspectiva financiera, 2 en procesos internos y 2 en clientes.

### ¿Por qué ninguno cae en Aprendizaje y crecimiento?

El Data Warehouse se construyó a partir de las **transacciones de venta** (órdenes, ítems, pagos, reseñas, clientes, vendedores y productos). No contiene datos sobre el personal, su capacitación, los sistemas internos ni el clima organizacional, que es lo que mide esta perspectiva. Proponer un KPI ahí obligaría a inventar datos o a usar una fuente que no existe, y eso rompería el criterio **Medible** de S.M.A.R.T.

Se documenta como **limitación del alcance** y como recomendación: si Invacom incorpora datos de recursos humanos o de sistemas (por ejemplo, horas de capacitación del equipo logístico o el porcentaje de vendedores con integración automática de envíos), esa perspectiva podría agregarse en una siguiente versión del tablero.

### Mapa de causa y efecto

```
Procesos internos              Clientes                      Financiera
LOG-01 Entregas a tiempo  ──►  CLI-01 Reseñas 4 y 5  ──►     VEN-02 Órdenes ──┐
LOG-02 Días de entrega    ──►  CLI-02 Cancelaciones  ──►     FIN-01 Ticket  ──┼──► VEN-01 Ventas
                                                             FIN-02 Flete   ──┘
```

- Si el proceso de entrega mejora (LOG), el cliente queda más satisfecho y cancela menos (CLI). El árbol de descomposición de la página Clientes lo confirma: la mayor parte de las reseñas de 1 y 2 estrellas viene de órdenes entregadas con atraso.
- Clientes satisfechos vuelven a comprar y recomiendan: más órdenes (VEN-02).
- Las ventas (VEN-01) son el resultado de las órdenes por el ticket (FIN-01). El peso del flete (FIN-02) afecta cuánto paga el cliente y, por lo tanto, su disposición a comprar.

## 4. Fichas técnicas de los KPIs

Cada ficha sigue los campos de la presentación de clase *Ficha técnica de un KPI* (nombre y código, línea base y meta, fórmula, semáforo, fuente y responsable), más la perspectiva, el tipo y la frecuencia. La periodicidad es **mensual** para los 8 KPIs: cada página compara el mes contra el anterior, y la página Finanzas además contra el mismo período del año anterior.

### KPI-VEN-01 · Ventas promedio mensuales
| Campo | Detalle |
|---|---|
| Objetivo | Medir el ingreso mensual real de la empresa. |
| Perspectiva / tipo | Financiera · Lagging |
| Fórmula | Suma de `monto_total` de órdenes no canceladas ni no disponibles ÷ meses con ventas |
| Fuente | `FACT_VENTAS[MONTO_TOTAL]`, `DIM_ESTADO_ORDEN`, `DIM_TIEMPO` (DW Invacom) |
| Unidad | BRL |
| Línea base 2017 / Meta ene–ago 2018 | 590,881 BRL / 1,000,000 BRL |
| Semáforo | Verde ≥ 1,000,000 · Amarillo 800,000 – 999,999 · Rojo < 800,000 |
| Frecuencia | Mensual |
| Responsable | Joshua |

### KPI-VEN-02 · Órdenes promedio mensuales
| Campo | Detalle |
|---|---|
| Objetivo | Medir el volumen de compras que genera el ingreso. |
| Perspectiva / tipo | Financiera · Leading |
| Fórmula | Órdenes distintas no canceladas ni no disponibles ÷ meses con ventas |
| Fuente | `FACT_VENTAS[ORDER_ID]`, `DIM_ESTADO_ORDEN`, `DIM_TIEMPO` |
| Unidad | Órdenes |
| Línea base 2017 / Meta ene–ago 2018 | 3,698 / 6,500 |
| Semáforo | Verde ≥ 6,500 · Amarillo 5,500 – 6,499 · Rojo < 5,500 |
| Frecuencia | Mensual |
| Responsable | Joshua |

### KPI-FIN-01 · Ticket promedio por orden
| Campo | Detalle |
|---|---|
| Objetivo | Medir el valor promedio que paga un cliente por cada orden. |
| Perspectiva / tipo | Financiera · Lagging |
| Fórmula | Suma de `monto_total` ÷ órdenes distintas (todas las órdenes) |
| Fuente | `FACT_VENTAS[MONTO_TOTAL]`, `FACT_VENTAS[ORDER_ID]` |
| Unidad | BRL |
| Línea base 2017 / Meta ene–ago 2018 | 160.23 BRL / 170.00 BRL |
| Semáforo | Verde ≥ 170 · Amarillo 155 – 169.99 · Rojo < 155 |
| Resultado ene–ago 2018 | 160.74 BRL (amarillo), +1.03 % frente a ene–ago 2017 |
| Frecuencia | Mensual |
| Responsable | Fernanda |

### KPI-FIN-02 · Peso del flete sobre la venta
| Campo | Detalle |
|---|---|
| Objetivo | Medir qué parte de cada real vendido corresponde al costo de envío. |
| Perspectiva / tipo | Financiera · Leading |
| Fórmula | Suma de `flete` ÷ suma de `monto_total` × 100 (órdenes no canceladas ni no disponibles) |
| Fuente | `FACT_VENTAS[FLETE]`, `FACT_VENTAS[MONTO_TOTAL]`, `DIM_ESTADO_ORDEN`, `DIM_CLIENTE[REGION]` |
| Unidad | % (menor es mejor) |
| Línea base 2017 / Meta ene–ago 2018 | 13.85 % / ≤ 13.00 % |
| Semáforo | Verde ≤ 13.00 % · Amarillo 13.01 – 14.50 % · Rojo > 14.50 % |
| Resultado ene–ago 2018 | 14.57 % (rojo), +0.75 puntos frente a ene–ago 2017; solo SP está por debajo de la meta |
| Frecuencia | Mensual |
| Responsable | Fernanda |

### KPI-LOG-01 · % de entregas a tiempo
| Campo | Detalle |
|---|---|
| Objetivo | Medir el cumplimiento de la fecha de entrega prometida. |
| Perspectiva / tipo | Procesos internos · Leading |
| Fórmula | Órdenes con `entregado_a_tiempo = 1` ÷ órdenes entregadas × 100 |
| Fuente | `FACT_VENTAS[ENTREGADO_A_TIEMPO]`, `DIM_CLIENTE` |
| Unidad | % |
| Línea base 2017 / Meta ene–ago 2018 | 94.35 % / 95.00 % |
| Semáforo | Verde ≥ 95 % · Amarillo 92 – 94.99 % · Rojo < 92 % |
| Frecuencia | Mensual |
| Responsable | Fabio |

### KPI-LOG-02 · Días promedio de entrega
| Campo | Detalle |
|---|---|
| Objetivo | Medir la velocidad del proceso logístico. |
| Perspectiva / tipo | Procesos internos · Leading |
| Fórmula | Promedio de `dias_entrega` por orden entregada |
| Fuente | `FACT_VENTAS[DIAS_ENTREGA]` |
| Unidad | Días (menor es mejor) |
| Línea base 2017 / Meta ene–ago 2018 | 12.54 días / ≤ 10.00 días |
| Semáforo | Verde ≤ 10 · Amarillo 10.01 – 12.50 · Rojo > 12.50 |
| Frecuencia | Mensual |
| Responsable | Fabio |

### KPI-CLI-01 · % de reseñas de 4 y 5 estrellas
| Campo | Detalle |
|---|---|
| Objetivo | Medir la satisfacción del cliente con su compra. |
| Perspectiva / tipo | Clientes · Lagging |
| Fórmula | Órdenes con `review_score ≥ 4` ÷ órdenes con reseña × 100 |
| Fuente | `FACT_VENTAS[REVIEW_SCORE]`, `DIM_PRODUCTO` |
| Unidad | % |
| Línea base 2017 / Meta ene–ago 2018 | 77.80 % / 80.00 % |
| Semáforo | Verde ≥ 80 % · Amarillo 76 – 79.99 % · Rojo < 76 % |
| Frecuencia | Mensual |
| Responsable | Joseph |

### KPI-CLI-02 · Tasa de cancelación de órdenes
| Campo | Detalle |
|---|---|
| Objetivo | Medir la proporción de órdenes que el cliente o la empresa terminan cancelando. |
| Perspectiva / tipo | Clientes · Lagging |
| Fórmula | Órdenes con estado *Cancelada* ÷ total de órdenes × 100 |
| Fuente | `FACT_VENTAS[ORDER_ID]`, `DIM_ESTADO_ORDEN[ESTADO_ORDEN_ES]` |
| Unidad | % (menor es mejor) |
| Línea base 2017 / Meta ene–ago 2018 | 0.458 % / ≤ 0.400 % |
| Semáforo | Verde ≤ 0.40 % · Amarillo 0.41 – 0.50 % · Rojo > 0.50 % |
| Frecuencia | Mensual |
| Responsable | Joseph |

## 5. Cómo se lee el semáforo

Todos los KPIs usan los mismos tres colores (verde `#2E7D32`, amarillo `#F9A825`, rojo `#C62828`), calculados en DAX a partir de la tabla `Metas KPI`:

- **Verde:** se cumple la meta → mantener las acciones actuales.
- **Amarillo:** entre la meta y el límite de alerta → vigilar y revisar causas.
- **Rojo:** fuera del límite → activar un plan correctivo con el responsable del KPI.

Cuando **mayor es mejor** (VEN-01, VEN-02, FIN-01, LOG-01, CLI-01), verde es valor ≥ meta y rojo es valor < límite rojo. Cuando **menor es mejor** (FIN-02, LOG-02, CLI-02), las comparaciones se invierten: verde es valor ≤ meta y rojo es valor > límite rojo.

## Referencias

Kaplan, R. S., & Norton, D. P. (1992). The balanced scorecard—Measures that drive performance. *Harvard Business Review, 70*(1), 71–79.

Kaplan, R. S., & Norton, D. P. (1996). *The balanced scorecard: Translating strategy into action*. Harvard Business School Press.

Kaplan, R. S., & Norton, D. P. (2004). *Strategy maps: Converting intangible assets into tangible outcomes*. Harvard Business School Press.

Parmenter, D. (2015). *Key performance indicators: Developing, implementing, and using winning KPIs* (3.ª ed.). John Wiley & Sons.

Universidad Técnica Nacional. (2026). *Minería de datos – 04 – KPIs* [Presentación de clase]. Curso ISW-911, Sede Regional de San Carlos.
