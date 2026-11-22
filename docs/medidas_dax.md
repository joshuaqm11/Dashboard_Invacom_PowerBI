# Medidas DAX — explicación para la defensa

El enunciado exige al menos 6 funciones DAX diferentes y que cada integrante explique técnicamente las suyas.
Cada responsable documenta aquí **sus** medidas: el código y una explicación en sus palabras (qué hace cada función y por qué se usó).

Convención: cada integrante tiene su propia tabla de medidas en el modelo.

| Tabla de medidas | Responsable |
|---|---|
| `Medidas Ventas` | Joshua |
| `Medidas Finanzas` | Fernanda |
| `Medidas Logística` | Fabio |
| `Medidas Clientes` | Joseph |

La tabla `Metas KPI` (metas y límites del semáforo de los 8 KPIs) es una tabla calculada con `DATATABLE`; no se relaciona con ninguna otra y las medidas la leen con `LOOKUPVALUE` por el código del KPI.

## Funciones DAX usadas (control del requisito ≥ 6)

| Función | Medida donde se usa | Responsable |
|---|---|---|
| `SUM` | Ventas válidas | Joshua |
| `CALCULATE` | Ventas válidas, Órdenes válidas, Ventas mes anterior, Órdenes mes anterior | Joshua |
| `DISTINCTCOUNT` | Órdenes válidas | Joshua |
| `COUNTROWS` | Meses con ventas | Joshua |
| `FILTER` | Meses con ventas | Joshua |
| `VALUES` | Meses con ventas | Joshua |
| `DIVIDE` | Ventas promedio mensuales, Órdenes promedio mensuales, Var VEN-01, Var VEN-02 | Joshua |
| `DATEADD` | Ventas mes anterior, Órdenes mes anterior | Joshua |
| `LOOKUPVALUE` | Meta VEN-01, Meta VEN-02, Color VEN-01, Color VEN-02 | Joshua |
| `VAR` / `RETURN` | Color VEN-01, Color VEN-02 | Joshua |
| `SWITCH` | Color VEN-01, Color VEN-02 | Joshua |
| `DATATABLE` | Tabla `Metas KPI` | Joshua |
| `KEEPFILTERS` | Órdenes con reseña, Reseñas 4 y 5, Reseñas negativas | Joseph |
| `REMOVEFILTERS` | % del total de reseñas negativas | Joseph |
| `TOTALYTD` | Cancelaciones acumuladas del año | Joseph |
| `ISBLANK` | Órdenes con reseña, Color CLI-01, Color CLI-02 | Joseph |
| `COUNTROWS` + `VALUES` | Total órdenes | Joseph |
| `AVERAGEX` | Días promedio de entrega | Fabio |
| `PREVIOUSMONTH` | % a tiempo mes anterior, Días mes anterior | Fabio |
| `USERELATIONSHIP` | Órdenes entregadas en el mes | Fabio |
| `MAX` | Días promedio de entrega | Fabio |
| `SAMEPERIODLASTYEAR` | Ticket año anterior, Flete año anterior | Fernanda |
| `SELECTEDVALUE` | Título Finanzas | Fernanda |
| `SUM`, `DISTINCTCOUNT`, `DIVIDE` | Ticket promedio, Peso del flete | Fernanda |
| `CALCULATE`, `DATEADD` | Peso del flete, Var FIN-01 y FIN-02 vs mes anterior | Fernanda |
| `LOOKUPVALUE`, `VAR`/`RETURN`, `SWITCH`, `ISBLANK` | Meta y Color FIN-01/FIN-02, Color Var FIN-01/FIN-02 | Fernanda |
| | | |

---

## Joshua — Medidas Ventas

Carpeta en el modelo: `Medidas Ventas → KPIs Ventas`.

Validación con 2017 (línea base aprobada): **Ventas promedio mensuales = 590,881 BRL** y **Órdenes promedio mensuales = 3,698**. Ambas coinciden.
Período meta (enero–agosto 2018): 1,074,142 BRL (verde, meta 1,000,000) y 6,691 órdenes (verde, meta 6,500).

### Ventas válidas
```dax
Ventas válidas =
CALCULATE (
    SUM ( FACT_VENTAS[MONTO_TOTAL] ),
    NOT DIM_ESTADO_ORDEN[ESTADO_ORDEN_ES] IN { "Cancelada", "No disponible" }
)
```
**Qué hace:** suma lo que pagaron los clientes (precio + flete) solo de las órdenes válidas.
**Funciones usadas y por qué:** `SUM` suma la columna `MONTO_TOTAL`. `CALCULATE` evalúa esa suma agregando un filtro: deja fuera los estados *Cancelada* y *No disponible*, porque esas órdenes no son venta real. La usa también Fernanda para el peso del flete.

### Órdenes válidas
```dax
Órdenes válidas =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    NOT DIM_ESTADO_ORDEN[ESTADO_ORDEN_ES] IN { "Cancelada", "No disponible" }
)
```
**Qué hace:** cuenta las órdenes únicas válidas.
**Funciones usadas y por qué:** `FACT_VENTAS` tiene una fila por **ítem**: una orden con 3 productos tiene 3 filas. `COUNT` contaría 3; `DISTINCTCOUNT` cuenta cada `ORDER_ID` una sola vez. `CALCULATE` aplica el mismo filtro de estados que las ventas.

### Meses con ventas
```dax
Meses con ventas =
COUNTROWS (
    FILTER ( VALUES ( DIM_TIEMPO[ANIO_MES] ), [Ventas válidas] > 0 )
)
```
**Qué hace:** cuenta cuántos meses del filtro actual tienen ventas (12 en 2017, 8 en enero–agosto 2018, 1 si se mira un solo mes).
**Funciones usadas y por qué:** `VALUES` devuelve la lista de meses (`ANIO_MES`) visibles con los filtros actuales. `FILTER` deja solo los meses donde `[Ventas válidas] > 0`. `COUNTROWS` cuenta cuántos quedaron. Así el promedio mensual divide entre los meses reales y no entre meses vacíos.

### Ventas promedio mensuales (KPI-VEN-01)
```dax
Ventas promedio mensuales =
DIVIDE ( [Ventas válidas], [Meses con ventas] )
```
**Qué hace:** el KPI-VEN-01: venta promedio por mes del período elegido.
**Funciones usadas y por qué:** `DIVIDE` divide y, si el denominador es 0 o vacío, devuelve vacío en vez de un error. Es *lagging*: muestra el resultado final del negocio.

### Órdenes promedio mensuales (KPI-VEN-02)
```dax
Órdenes promedio mensuales =
DIVIDE ( [Órdenes válidas], [Meses con ventas] )
```
**Qué hace:** el KPI-VEN-02: cantidad promedio de órdenes por mes.
**Funciones usadas y por qué:** igual que la anterior, con `DIVIDE`. Es *leading*: si bajan las órdenes, después bajan las ventas (Ventas ≈ Órdenes × Ticket).

### Ventas mes anterior y Var VEN-01 vs mes anterior
```dax
Ventas mes anterior =
CALCULATE ( [Ventas promedio mensuales], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )

Var VEN-01 vs mes anterior =
DIVIDE ( [Ventas promedio mensuales] - [Ventas mes anterior], [Ventas mes anterior] )
```
**Qué hace:** calcula el KPI del mes anterior y el cambio porcentual contra el mes actual (comparación de períodos mensuales que pide el enunciado).
**Funciones usadas y por qué:** `DATEADD` corre las fechas del filtro un mes hacia atrás (por ejemplo, marzo → febrero). `CALCULATE` recalcula el KPI con esas fechas. Funciona porque `DIM_TIEMPO` está marcada como tabla de fechas. `DIVIDE` da el porcentaje de cambio: (actual − anterior) ÷ anterior.

### Órdenes mes anterior y Var VEN-02 vs mes anterior
```dax
Órdenes mes anterior =
CALCULATE ( [Órdenes promedio mensuales], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )

Var VEN-02 vs mes anterior =
DIVIDE ( [Órdenes promedio mensuales] - [Órdenes mes anterior], [Órdenes mes anterior] )
```
**Qué hace:** lo mismo para las órdenes. Según la propuesta, una caída mayor a 10 % contra el mes anterior dispara una revisión de la captación de clientes.
**Funciones usadas y por qué:** las mismas que la anterior (`CALCULATE`, `DATEADD`, `DIVIDE`).

### Meta VEN-01 y Meta VEN-02
```dax
Meta VEN-01 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-VEN-01" )
Meta VEN-02 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-VEN-02" )
```
**Qué hace:** trae la meta aprobada (1,000,000 BRL y 6,500 órdenes).
**Funciones usadas y por qué:** `LOOKUPVALUE` busca en la tabla `Metas KPI` la fila cuyo código es el del KPI y devuelve su meta. Así la meta no está escrita a mano en cada medida: si cambia, se cambia solo en la tabla.

### Color VEN-01 y Color VEN-02 (semáforo)
```dax
Color VEN-01 =
VAR valor = [Ventas promedio mensuales]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-VEN-01" )
VAR rojo  = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo],  'Metas KPI'[Codigo], "KPI-VEN-01" )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( valor ), BLANK (),
        valor >= verde, "#2E7D32",
        valor <  rojo,  "#C62828",
        "#F9A825"
    )
```
(`Color VEN-02` es igual, con `[Órdenes promedio mensuales]` y el código `KPI-VEN-02`.)

**Qué hace:** devuelve el color del semáforo: verde si cumple la meta, rojo si está debajo del límite rojo y amarillo en medio. Se usa en el formato condicional de los visuales (**fx → Valor del campo**).
**Funciones usadas y por qué:** `VAR` guarda el valor del KPI y los dos límites con un nombre, para no calcularlos varias veces y que se lea fácil; `RETURN` devuelve el resultado. `SWITCH(TRUE(), ...)` revisa las condiciones en orden y devuelve la primera que se cumple. `ISBLANK` evita pintar un color cuando no hay datos.

---

## Fernanda — Medidas Finanzas

Carpeta en el modelo: `Medidas Finanzas`.

Validación con 2017 (línea base aprobada): **Ticket promedio = 160.23 BRL** y **Peso del flete = 13.85 %**. Ambas coinciden (captura en `evidencias/validacion_kpis_finanzas_2017.png`).
Período meta (enero–agosto 2018): 160.74 BRL (amarillo, meta 170.00 BRL) y 14.57 % (rojo, meta ≤ 13.00 %).

### Ticket promedio (KPI-FIN-01)
```dax
Ticket promedio =
DIVIDE (
    SUM ( FACT_VENTAS[MONTO_TOTAL] ),
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] )
)
```
**Qué hace:** el KPI-FIN-01: cuánto paga en promedio un cliente por orden (precio + flete).
**Funciones usadas y por qué:** `SUM` suma lo cobrado en todas las filas. `DISTINCTCOUNT` cuenta cada `ORDER_ID` una sola vez, porque `FACT_VENTAS` tiene una fila por **ítem** y una orden con 3 productos tiene 3 filas; si se usara `COUNT`, el ticket saldría como un promedio por ítem y no por orden. `DIVIDE` evita el error cuando no hay órdenes.
**Por qué usa todas las órdenes (y las ventas no):** así quedó aprobado en la propuesta. El ticket describe el **comportamiento de compra** en el momento de ordenar, incluida la orden que luego se cancela; las ventas válidas miden **ingreso real**, y por eso excluyen canceladas y no disponibles. Las canceladas son menos del 0.5 % de las órdenes, así que casi no mueven el ticket. Es *lagging*: es el resultado de lo que ya compró cada cliente.

### Peso del flete (KPI-FIN-02)
```dax
Peso del flete =
VAR flete =
    CALCULATE (
        SUM ( FACT_VENTAS[FLETE] ),
        NOT DIM_ESTADO_ORDEN[ESTADO_ORDEN_ES] IN { "Cancelada", "No disponible" }
    )
VAR venta = [Ventas válidas]
RETURN
    DIVIDE ( flete, venta )
```
**Qué hace:** el KPI-FIN-02: qué parte de cada real vendido corresponde al flete. Menor es mejor.
**Funciones usadas y por qué:** `CALCULATE` suma el flete con el **mismo filtro de estados** que usa `[Ventas válidas]` de Joshua, para que numerador y denominador hablen de las mismas órdenes. `VAR` guarda el flete y la venta con un nombre: se lee como la fórmula de la propuesta (flete ÷ venta), cada parte se calcula una sola vez y se puede revisar por separado. `RETURN` devuelve la división y `DIVIDE` evita el error si no hay ventas. Reutilizar `[Ventas válidas]` evita escribir dos veces la misma lógica. Es *leading*: un flete caro desincentiva la compra y presiona el margen antes de que se note en las ventas.

### Ticket año anterior y Var FIN-01 vs año anterior
```dax
Ticket año anterior =
CALCULATE ( [Ticket promedio], SAMEPERIODLASTYEAR ( DIM_TIEMPO[FECHA] ) )

Var FIN-01 vs año anterior =
VAR actual   = [Ticket promedio]
VAR anterior = [Ticket año anterior]
RETURN
    DIVIDE ( actual - anterior, anterior )
```
**Qué hace:** compara el ticket con el **mismo período del año anterior** (enero–agosto 2018 contra enero–agosto 2017) y da el cambio en %. Es la variación que muestran las tarjetas de la página Finanzas: +1.03 % en 2018.
**Funciones usadas y por qué:** `SAMEPERIODLASTYEAR` corre las fechas del filtro **un año** hacia atrás: si el filtro es marzo 2018, calcula marzo 2017. `CALCULATE` recalcula el ticket con esas fechas. Funciona porque `DIM_TIEMPO` está marcada como tabla de fechas. `VAR`/`RETURN` dejan clara la fórmula (actual − anterior) ÷ anterior y `DIVIDE` evita el error si no hay año anterior (2016 está fuera del análisis por datos incompletos).

### Flete año anterior y Var FIN-02 vs año anterior
```dax
Flete año anterior =
CALCULATE ( [Peso del flete], SAMEPERIODLASTYEAR ( DIM_TIEMPO[FECHA] ) )

Var FIN-02 vs año anterior =
[Peso del flete] - [Flete año anterior]
```
**Qué hace:** lo mismo para el flete. En 2018 el peso del flete subió +0.75 puntos frente al mismo período de 2017.
**Por qué se resta y no se divide:** el peso del flete **ya es un porcentaje**. Restar dos porcentajes da la diferencia en **puntos porcentuales** (14.57 % − 13.82 % = +0.75 puntos), que es como se comunica el cambio de una tasa. Dividir daría un "porcentaje de un porcentaje" (+5.4 %), que se presta a confusión. Aquí una variación **negativa es buena** (baja el flete).

### Var FIN-01 y Var FIN-02 vs mes anterior
```dax
Var FIN-01 vs mes anterior =
VAR actual   = [Ticket promedio]
VAR anterior = CALCULATE ( [Ticket promedio], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )
RETURN
    DIVIDE ( actual - anterior, anterior )

Var FIN-02 vs mes anterior =
[Peso del flete] - CALCULATE ( [Peso del flete], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )
```
**Qué hace:** la comparación mensual que pide el enunciado; la usan la tarjeta y la matriz de la Portada.
**Funciones usadas y por qué:** `DATEADD(..., -1, MONTH)` corre las fechas del filtro **un mes** atrás (marzo 2018 → febrero 2018).
**Diferencia con `SAMEPERIODLASTYEAR`:** `SAMEPERIODLASTYEAR` siempre retrocede **un año** (marzo 2018 → marzo 2017) y sirve para comparar sin el efecto de la temporada; equivale a `DATEADD(..., -1, YEAR)`. `DATEADD` permite elegir el intervalo y la cantidad (−1 mes, −3 meses, −1 trimestre…); con −1 mes muestra la tendencia inmediata.

### Meta FIN-01 y Meta FIN-02
```dax
Meta FIN-01 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-FIN-01" )
Meta FIN-02 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-FIN-02" )
```
**Qué hace:** trae la meta aprobada (170.00 BRL y 13.00 %). Se muestra en las tarjetas y es el **valor de destino** del Bullet Chart.
**Funciones usadas y por qué:** `LOOKUPVALUE` busca en `Metas KPI` la fila del código y devuelve la meta; si la meta cambia, se cambia solo en la tabla.

### Color FIN-01 y Color FIN-02 (semáforo)
```dax
Color FIN-01 =                       -- mayor es mejor
VAR valor = [Ticket promedio]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-FIN-01" )
VAR rojo  = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo],  'Metas KPI'[Codigo], "KPI-FIN-01" )
RETURN
    SWITCH ( TRUE (),
        ISBLANK ( valor ), BLANK (),
        valor >= verde, "#2E7D32",
        valor <  rojo,  "#C62828",
        "#F9A825" )

Color FIN-02 =                       -- menor es mejor
VAR valor = [Peso del flete]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-FIN-02" )
VAR rojo  = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo],  'Metas KPI'[Codigo], "KPI-FIN-02" )
RETURN
    SWITCH ( TRUE (),
        ISBLANK ( valor ), BLANK (),
        valor <= verde, "#2E7D32",
        valor >  rojo,  "#C62828",
        "#F9A825" )
```
**Qué hace:** el color del semáforo. FIN-01: verde ≥ 170, rojo < 155. FIN-02: verde ≤ 13.00 %, rojo > 14.50 %. Lo de en medio es amarillo. Se usa en el formato condicional (**fx → Valor del campo**) de las tarjetas y de las barras por estado.
**Funciones usadas y por qué:** `SWITCH(TRUE(), ...)` es la plantilla del semáforo: compara `TRUE()` contra cada condición **en orden** y devuelve el resultado de la **primera** que se cumple; si ninguna se cumple, devuelve el último valor (amarillo). Por eso el orden importa: primero el vacío, luego verde, luego rojo. Cuando **menor es mejor** (FIN-02) solo se invierten las comparaciones: `<=` para verde y `>` para rojo. `ISBLANK` evita pintar un color cuando no hay datos.

### Color Var FIN-01 y Color Var FIN-02
```dax
Color Var FIN-01 =
VAR v = [Var FIN-01 vs mes anterior]
RETURN
    SWITCH ( TRUE (), ISBLANK ( v ), BLANK (), v >= 0, "#2E7D32", "#C62828" )

Color Var FIN-02 =
VAR v = [Var FIN-02 vs mes anterior]
RETURN
    SWITCH ( TRUE (), ISBLANK ( v ), BLANK (), v <= 0, "#2E7D32", "#C62828" )
```
**Qué hace:** pinta la variación contra el mes anterior en la Portada: en el ticket, subir es verde; en el flete, **bajar** es verde.

### Título Finanzas (título dinámico)
```dax
Título Finanzas =
"Finanzas · " & SELECTEDVALUE ( DIM_TIEMPO[ANIO_MES], "período seleccionado" )
```
**Qué hace:** el título de la página cambia según el filtro: con un solo mes elegido muestra, por ejemplo, "Finanzas · 2018-03"; con varios meses, "Finanzas · período seleccionado".
**Funciones usadas y por qué:** `SELECTEDVALUE` devuelve el valor de la columna si hay **exactamente uno** en el contexto; si hay varios o ninguno, devuelve el texto alternativo. `&` une los textos.

### Página Finanzas: qué muestra
- **Tarjetas** de KPI-FIN-01 y KPI-FIN-02 con semáforo, meta y variación contra el año anterior (verde/rojo según si mejora).
- **Bullet Chart (AppSource)**: peso del flete por región contra la meta de 13 %, con el mismo eje para comparar regiones.
- **Columnas**: ticket promedio por mes, 2017 contra 2018. Se quitó la interacción con el segmentador de Año (**Editar interacciones → Ninguno**) para que siempre compare los dos años.
- **Barras**: peso del flete por estado, coloreadas con `Color FIN-02`.

**Hallazgo:** en 2018 solo **São Paulo (SP)** cumple la meta de flete (≈ 12 %); los estados del norte y nordeste superan el 18 %. Como SP concentra la mayor parte de las ventas, sostiene el promedio nacional en 14.57 %. Para cumplir el KPI-FIN-02, el foco está en el costo de los envíos fuera del sudeste.

---

## Fabio — Medidas Logística

Carpeta en el modelo: `Medidas Logística → KPIs Logística`.

**Lo que hay que entender primero:** `FACT_VENTAS` tiene una fila por **ítem**, pero la entrega es un dato de la **orden**: una orden con 3 ítems repite su `ENTREGADO_A_TIEMPO` y sus `DIAS_ENTREGA` 3 veces. Por eso todo se cuenta y promedia **por orden** (`ORDER_ID`). Las órdenes no entregadas tienen esos campos vacíos y quedan fuera.

Validación con 2017 (línea base aprobada): **% entregas a tiempo = 94.35 %** (40,973 de 43,426 órdenes entregadas) y **Días promedio de entrega = 12.54**. Ambas coinciden.
Período meta (enero–agosto 2018): 92.27 % (amarillo, meta 95 %) y 11.69 días (amarillo, meta ≤ 10).

### Órdenes entregadas / Órdenes a tiempo
```dax
Órdenes entregadas =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    FILTER ( FACT_VENTAS, NOT ISBLANK ( FACT_VENTAS[ENTREGADO_A_TIEMPO] ) )
)

Órdenes a tiempo =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    FILTER ( FACT_VENTAS, FACT_VENTAS[ENTREGADO_A_TIEMPO] = 1 )
)
```
**Qué hace:** cuenta órdenes únicas con dato de entrega y órdenes únicas entregadas a tiempo.
**Funciones usadas y por qué:** `FILTER` recorre fila por fila la tabla que ya está en el contexto (año, región…) y deja solo las filas que cumplen la condición; `CALCULATE` usa ese resultado como filtro. `DISTINCTCOUNT` cuenta órdenes, no filas: una orden de 3 ítems cuenta 1 vez. `ISBLANK` descarta las órdenes no entregadas.

### KPI-LOG-01 · % entregas a tiempo
```dax
% entregas a tiempo = DIVIDE ( [Órdenes a tiempo], [Órdenes entregadas] )
```
**Qué hace:** órdenes a tiempo ÷ órdenes entregadas. Formato `0.00%`.
**Funciones usadas y por qué:** `DIVIDE` devuelve vacío si el divisor es 0, en vez de error.

### KPI-LOG-02 · Días promedio de entrega
```dax
Días promedio de entrega =
AVERAGEX (
    VALUES ( FACT_VENTAS[ORDER_ID] ),
    CALCULATE ( MAX ( FACT_VENTAS[DIAS_ENTREGA] ) )
)
```
**Qué hace:** promedia los días de entrega **una vez por orden**. Formato `0.00`.
**Funciones usadas y por qué:** `VALUES` da la lista de órdenes únicas del contexto. `AVERAGEX` es un iterador: evalúa la expresión para cada orden y promedia los resultados, ignorando los vacíos (órdenes sin entregar). `CALCULATE ( MAX (…) )` convierte la orden actual en filtro (transición de contexto) y toma su valor de días; `MAX` sirve porque todas las filas de la orden tienen el mismo valor. Un `AVERAGE` simple promediaría ítems: una orden de 5 ítems pesaría 5 veces más.

### Mes anterior y variación
```dax
% a tiempo mes anterior = CALCULATE ( [% entregas a tiempo], PREVIOUSMONTH ( DIM_TIEMPO[FECHA] ) )
Var LOG-01 vs mes anterior = [% entregas a tiempo] - [% a tiempo mes anterior]

Días mes anterior = CALCULATE ( [Días promedio de entrega], PREVIOUSMONTH ( DIM_TIEMPO[FECHA] ) )
Var LOG-02 vs mes anterior = [Días promedio de entrega] - [Días mes anterior]
```
**Qué hace:** compara el KPI con el mes anterior (ej.: marzo 2018 = 81.04 %, −4.82 pts vs febrero; 15.87 días, −0.64 vs febrero).
**Funciones usadas y por qué:** `PREVIOUSMONTH` devuelve todas las fechas del mes anterior al primer día del contexto; funciona porque `DIM_TIEMPO` está marcada como tabla de fechas. A diferencia de `DATEADD ( …, -1, MONTH )`, que **desplaza** todo el período un mes, `PREVIOUSMONTH` siempre devuelve **un mes completo**. Con un solo mes en el contexto dan lo mismo; con un rango, no. Por eso la variación se lee por mes (gráfico por mes o segmentador de mes).

### Órdenes entregadas en el mes (por fecha de entrega)
```dax
Órdenes entregadas en el mes =
CALCULATE (
    [Órdenes entregadas],
    USERELATIONSHIP ( FACT_VENTAS[SK_FECHA_ENTREGA], DIM_TIEMPO[SK_TIEMPO] )
)
```
**Qué hace:** cuenta las órdenes según el mes en que se **entregaron**, no el mes en que se compraron.
**Funciones usadas y por qué:** entre dos tablas solo puede haber una relación activa. La de fecha de compra es la activa porque los KPIs se miden por mes de compra; la de entrega está inactiva. `USERELATIONSHIP` la activa solo dentro de esta medida, sin cambiar las demás.

### Metas y colores
```dax
Meta LOG-01 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-LOG-01" )
Meta LOG-02 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-LOG-02" )

Color LOG-01 =
VAR valor = [% entregas a tiempo]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-LOG-01" )
VAR rojo = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo], 'Metas KPI'[Codigo], "KPI-LOG-01" )
RETURN
    SWITCH ( TRUE (), ISBLANK ( valor ), BLANK (), valor >= verde, "#2E7D32", valor < rojo, "#C62828", "#F9A825" )

Color LOG-02 =
VAR valor = [Días promedio de entrega]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-LOG-02" )
VAR rojo = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo], 'Metas KPI'[Codigo], "KPI-LOG-02" )
RETURN
    SWITCH ( TRUE (), ISBLANK ( valor ), BLANK (), valor <= verde, "#2E7D32", valor > rojo, "#C62828", "#F9A825" )

Color Var LOG-01 =
VAR v = [Var LOG-01 vs mes anterior]
RETURN SWITCH ( TRUE (), ISBLANK ( v ), BLANK (), v >= 0, "#2E7D32", "#C62828" )

Color Var LOG-02 =
VAR v = [Var LOG-02 vs mes anterior]
RETURN SWITCH ( TRUE (), ISBLANK ( v ), BLANK (), v <= 0, "#2E7D32", "#C62828" )
```
**Qué hace:** lee meta y límites de `Metas KPI` y devuelve el color del semáforo para el formato condicional (fx → Valor del campo). LOG-01: mayor es mejor. LOG-02: menor es mejor (por eso `<=` y una variación negativa es verde).
**Funciones usadas y por qué:** `LOOKUPVALUE` busca por código del KPI porque `Metas KPI` no tiene relaciones. `VAR / RETURN` guarda cada valor una sola vez. `SWITCH ( TRUE (), … )` evalúa las condiciones en orden, como un `if / else if`.

**Por qué los dos KPIs son leading:** miden el proceso de entrega, que ocurre **antes** de los resultados: entregas tarde o lentas anticipan reseñas malas, cancelaciones y menos recompra (KPIs lagging). Se pueden corregir dentro del mes.


### Título Logística
```dax
Título Logística = "Logística · " & SELECTEDVALUE ( DIM_TIEMPO[ANIO_MES], "período seleccionado" )
```
**Qué hace:** título dinámico de la página: muestra el mes elegido en los segmentadores o «período seleccionado» si hay varios meses.
**Funciones usadas y por qué:** `SELECTEDVALUE` devuelve el valor de la columna si hay uno solo en el contexto y, si no, el texto alternativo.

### Columna calculada `Estado (mapa)` en `DIM_CLIENTE`
```dax
Estado (mapa) =
VAR nombre = SWITCH ( DIM_CLIENTE[ESTADO], "AC", "Acre", "AL", "Alagoas", … , "TO", "Tocantins", BLANK () )
RETURN IF ( ISBLANK ( nombre ), BLANK (), nombre & ", Brasil" )
```
**Qué hace:** convierte la sigla del estado (SP, RJ…) en «São Paulo, Brasil», «Rio de Janeiro, Brasil», etc.
**Por qué:** para Azure Maps una sigla de 2 letras es ambigua (puede ser un estado o provincia de otro país) y la ubica mal. Con el nombre completo y el país, cada burbuja cae en su estado. Es columna (no medida) porque se usa como **ubicación** del mapa, igual que una categoría. El cliente con estado «NA» queda en blanco y no se dibuja.

### Página Logística
- **Tarjetas:** `% entregas a tiempo` y `Días promedio de entrega`, color del valor con `Color LOG-01` / `Color LOG-02`; referencias con la meta y la variación vs mes anterior (color con `Color Var LOG-01` / `Color Var LOG-02`).
- **Azure Maps:** ubicación `Estado (mapa)`, tamaño `Órdenes entregadas`, color de burbuja fx → `Color LOG-01`; información sobre herramientas con los dos KPIs.
- **Líneas:** `% entregas a tiempo` por mes, 2017 vs 2018, con línea constante en `Meta LOG-01`. El segmentador de Año no filtra este visual, para que siempre compare los dos años.
- **Barras:** `Días promedio de entrega` por región, color fx → `Color LOG-02`.
- **Lectura (2018, ene–ago):** solo el Sudeste cumple la meta de ≤ 10 días (9.95); Norte (23.26) y Nordeste (19.05) son los más lentos. En 2018 el % a tiempo cae fuerte en febrero–marzo (≈ 81 %).

---

## Joseph — Medidas Clientes

Carpeta en el modelo: `Medidas Clientes → KPIs Clientes`.

**Lo que hay que entender primero:** `FACT_VENTAS` tiene una fila por **ítem**, pero la reseña (`REVIEW_SCORE`) y el estado de la orden son datos de la **orden**: se repiten en cada ítem. Por eso todo se cuenta como órdenes únicas (`DISTINCTCOUNT` / `COUNTROWS(VALUES(...))` sobre `ORDER_ID`).

Validación con 2017 (línea base aprobada): **% reseñas 4 y 5 = 77.80 %** y **Tasa de cancelación = 0.458 %**.
Período meta (enero–agosto 2018): 77.43 % (amarillo, meta 80.00 %) y 0.454 % (amarillo, meta ≤ 0.400 %).

### Órdenes con reseña, Reseñas 4 y 5 y % reseñas 4 y 5 (KPI-CLI-01)
```dax
Órdenes con reseña =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    KEEPFILTERS ( NOT ISBLANK ( FACT_VENTAS[REVIEW_SCORE] ) )
)

Reseñas 4 y 5 =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    KEEPFILTERS ( FACT_VENTAS[REVIEW_SCORE] >= 4 )
)

% reseñas 4 y 5 =
DIVIDE ( [Reseñas 4 y 5], [Órdenes con reseña] )
```
**Qué hace:** el KPI-CLI-01: de las órdenes que tienen reseña, qué porcentaje recibió 4 o 5 estrellas.
**Funciones usadas y por qué:** `DISTINCTCOUNT` cuenta cada orden una sola vez. `CALCULATE` agrega la condición sobre la reseña. `KEEPFILTERS` hace que esa condición se **sume** a los filtros que ya existen sobre `REVIEW_SCORE` en vez de reemplazarlos: si un visual ya filtra, por ejemplo, reseñas = 1, sin `KEEPFILTERS` el `CALCULATE` pisaría ese filtro y contaría las de 4 y 5 de todas formas; con `KEEPFILTERS` el resultado respeta la intersección (da 0). `ISBLANK` excluye las órdenes sin reseña del denominador. `DIVIDE` evita el error de división entre cero. Es *lagging*: la reseña se escribe después de recibir el pedido.

### Total órdenes, Órdenes canceladas y Tasa de cancelación (KPI-CLI-02)
```dax
Total órdenes =
COUNTROWS ( VALUES ( FACT_VENTAS[ORDER_ID] ) )

Órdenes canceladas =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    DIM_ESTADO_ORDEN[ESTADO_ORDEN_ES] = "Cancelada"
)

Tasa de cancelación =
DIVIDE ( [Órdenes canceladas], [Total órdenes] )
```
**Qué hace:** el KPI-CLI-02: porcentaje de órdenes que terminan canceladas (formato `0.000 %` porque el valor es menor a 1 %).
**Funciones usadas y por qué:** `VALUES` devuelve la lista de `ORDER_ID` distintos visibles en el contexto y `COUNTROWS` cuenta las filas de esa lista: por eso da exactamente lo mismo que `DISTINCTCOUNT` (los dos cuentan órdenes únicas). `CALCULATE` filtra la dimensión de estado a *Cancelada*; el filtro viaja por la relación hasta `FACT_VENTAS`. Es *lagging*: la cancelación es el desenlace final de la orden.

### Reseñas negativas y % del total de reseñas negativas (para el árbol)
```dax
Reseñas negativas =
CALCULATE (
    DISTINCTCOUNT ( FACT_VENTAS[ORDER_ID] ),
    KEEPFILTERS ( FACT_VENTAS[REVIEW_SCORE] <= 2 )
)

% del total de reseñas negativas =
DIVIDE (
    [Reseñas negativas],
    CALCULATE ( [Reseñas negativas], REMOVEFILTERS ( DIM_CLIENTE ), REMOVEFILTERS ( DIM_PRODUCTO ) )
)
```
**Qué hace:** cuenta las órdenes con 1 o 2 estrellas (es la medida que analiza el árbol de descomposición) y calcula qué parte del total cae en la región, estado o categoría que se está viendo.
**Funciones usadas y por qué:** `KEEPFILTERS` es clave en el árbol: cada nivel del árbol agrega filtros y la condición `<= 2` tiene que convivir con ellos. `REMOVEFILTERS` quita **solo en el denominador** los filtros de cliente y de producto, así el denominador es el total de reseñas negativas del período y el numerador la parte de esa región o categoría. Diferencia: `KEEPFILTERS` **conserva** los filtros existentes y agrega uno; `REMOVEFILTERS` **borra** los filtros de una tabla o columna.

### Cancelaciones acumuladas del año
```dax
Cancelaciones acumuladas del año =
TOTALYTD ( [Órdenes canceladas], DIM_TIEMPO[FECHA] )
```
**Qué hace:** suma las cancelaciones desde el 1 de enero hasta el mes que se está viendo (en marzo muestra enero + febrero + marzo).
**Funciones usadas y por qué:** `TOTALYTD` es una función de inteligencia de tiempo: necesita una columna de fechas continua y sin nulos, por eso `DIM_TIEMPO` está marcada como **tabla de fechas** (columna `FECHA`) y se quitó la fila `-1`.

### Var CLI-01 y Var CLI-02 vs mes anterior
```dax
Var CLI-01 vs mes anterior =
[% reseñas 4 y 5]
    - CALCULATE ( [% reseñas 4 y 5], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )

Var CLI-02 vs mes anterior =
[Tasa de cancelación]
    - CALCULATE ( [Tasa de cancelación], DATEADD ( DIM_TIEMPO[FECHA], -1, MONTH ) )
```
**Qué hace:** la comparación mensual que pide el enunciado. Como los dos KPIs ya son porcentajes, se **restan** (diferencia en puntos) en vez de dividir.
**Funciones usadas y por qué:** `DATEADD(..., -1, MONTH)` corre las fechas del filtro un mes atrás y `CALCULATE` recalcula el KPI con esas fechas. En CLI-01 una variación positiva es buena; en CLI-02 una negativa es buena (bajan las cancelaciones). Los colores `Color Var CLI-01` y `Color Var CLI-02` aplican esa regla en el formato condicional.

### Meta CLI-01, Meta CLI-02, Color CLI-01 y Color CLI-02
```dax
Meta CLI-01 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-CLI-01" )
Meta CLI-02 = LOOKUPVALUE ( 'Metas KPI'[Meta], 'Metas KPI'[Codigo], "KPI-CLI-02" )

Color CLI-01 =                       -- mayor es mejor
VAR valor = [% reseñas 4 y 5]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-CLI-01" )
VAR rojo  = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo],  'Metas KPI'[Codigo], "KPI-CLI-01" )
RETURN
    SWITCH ( TRUE (), ISBLANK ( valor ), BLANK (),
        valor >= verde, "#2E7D32", valor < rojo, "#C62828", "#F9A825" )

Color CLI-02 =                       -- menor es mejor
VAR valor = [Tasa de cancelación]
VAR verde = LOOKUPVALUE ( 'Metas KPI'[LimiteVerde], 'Metas KPI'[Codigo], "KPI-CLI-02" )
VAR rojo  = LOOKUPVALUE ( 'Metas KPI'[LimiteRojo],  'Metas KPI'[Codigo], "KPI-CLI-02" )
RETURN
    SWITCH ( TRUE (), ISBLANK ( valor ), BLANK (),
        valor <= verde, "#2E7D32", valor > rojo, "#C62828", "#F9A825" )
```
**Qué hace:** la meta aprobada (80.00 % y 0.400 %) y el color del semáforo: CLI-01 verde ≥ 80 %, rojo < 76 %; CLI-02 verde ≤ 0.40 %, rojo > 0.50 %; lo de en medio amarillo. Se usan en el formato condicional (**fx → Valor del campo**).
**Funciones usadas y por qué:** `LOOKUPVALUE` lee la meta y los límites de la tabla `Metas KPI` por el código. `SWITCH(TRUE(), ...)` evalúa las condiciones en orden y devuelve la primera que se cumple. En CLI-02 las comparaciones van al revés porque **menor es mejor**.

### Columna calculada Entrega (FACT_VENTAS)
```dax
Entrega =
SWITCH (
    FACT_VENTAS[ENTREGADO_A_TIEMPO],
    1, "A tiempo",
    0, "Con atraso",
    "Sin entregar"
)
```
**Qué muestra en el tablero (ene 2017 – ago 2018):** las órdenes con atraso son solo el 6.7 % de las órdenes (6,532 de 98,353), pero aportan el 28.4 % de las reseñas negativas (4,134 de 14,556): el 63 % de las órdenes con atraso termina con 1 o 2 estrellas, contra el 9.7 % de las que llegan a tiempo. El % de reseñas 4 y 5 cae de 82.68 % (a tiempo) a 26.70 % (con atraso). En números absolutos hay más reseñas negativas en órdenes a tiempo (8,737) porque son muchísimas más órdenes; por eso se lee la **tasa**, no el conteo.

**Qué hace:** convierte el indicador 1/0/vacío en un texto legible para el árbol de descomposición y el gráfico de columnas. Es columna (no medida) porque se usa como **eje / categoría** para partir los datos.
