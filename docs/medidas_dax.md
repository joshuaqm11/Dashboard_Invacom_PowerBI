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

### [Nombre de la medida]
```dax
-- código
```
**Qué hace:**
**Funciones usadas y por qué:**
