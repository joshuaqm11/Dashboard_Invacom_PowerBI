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

### [Nombre de la medida]
```dax
-- código
```
**Qué hace:**
**Funciones usadas y por qué:**

---

## Fabio — Medidas Logística

### [Nombre de la medida]
```dax
-- código
```
**Qué hace:**
**Funciones usadas y por qué:**

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
