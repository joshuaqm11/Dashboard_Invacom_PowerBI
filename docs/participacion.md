# Participación de los integrantes · Proyecto #2

Universidad Técnica Nacional · ISW-911 Minería de Datos · 3C 2026

El equipo acordó asignar el **porcentaje máximo de participación (100 %)** a los cuatro integrantes. Cada uno desarrolló sus 2 KPIs de principio a fin (medidas DAX, metas, semáforo, página del tablero, validación 2017 y documentación) y además cumplió un requisito transversal del enunciado.

## Resumen

| Integrante | Rol | KPIs | Página | Participación |
|---|---|---|---|---|
| Joshua | Coordinador / Scrum Master | KPI-VEN-01 · KPI-VEN-02 | Ventas | **100 %** |
| Fernanda | Desarrolladora | KPI-FIN-01 · KPI-FIN-02 | Finanzas | **100 %** |
| Fabio | Desarrollador | KPI-LOG-01 · KPI-LOG-02 | Logística | **100 %** |
| Joseph | Desarrollador | KPI-CLI-01 · KPI-CLI-02 | Clientes y Portada | **100 %** |

## Aporte de cada integrante

### Joshua · 100 %
- Creación del repositorio, estructura inicial, plantilla de PR y proyecto PBIP con las 8 tablas del DW.
- Modelo semántico base: tipos de datos, tabla de fechas, claves ocultas y tablas de medidas (PR #1).
- Medidas de KPI-VEN-01 y KPI-VEN-02 con comparación mensual (PR #2).
- Tabla `Metas KPI` con metas y umbrales del semáforo de los 8 KPIs (PR #3).
- Página Ventas: tarjetas con semáforo, velocímetro **Tachometer** (visual de AppSource #1), línea 2017 vs 2018 con meta, Top 10 categorías y ventas por región (PR #4 y #19).
- Integración de los PRs del equipo, Kanban del proyecto y formato final de las páginas (rama `joshua/formato-paginas`).

### Fernanda · 100 %
- Tema visual Invacom (PR #11).
- Medidas de KPI-FIN-01 y KPI-FIN-02 con metas, semáforo, variaciones y validación 2017 (PR #12).
- Página Finanzas: **Bullet Chart** por región (visual de AppSource #2), ticket promedio por mes 2017 vs 2018 y peso del flete por estado (PR #14).
- Documentación: medidas de Finanzas, metodología Balanced Scorecard y fichas técnicas de los KPIs (PR #16).

### Fabio · 100 %
- Medidas de KPI-LOG-01 y KPI-LOG-02 con metas, colores y validación 2017 (PR #13).
- Columna `Estado (mapa)` en `DIM_CLIENTE` para ubicar los estados de Brasil (PR #15).
- Página Logística: tarjetas con semáforo, **mapa** de Azure Maps por estado, tendencia vs meta y días de entrega por región (PR #17).

### Joseph · 100 %
- Metodología S.M.A.R.T. y clasificación leading/lagging de los KPIs (PR #5).
- Columna `Entrega` en `FACT_VENTAS` y medidas de KPI-CLI-01 y KPI-CLI-02 (PR #6 y #7).
- Página Clientes: tarjetas KPI, **árbol de descomposición** y gráficos de contexto (PR #8).
- Portada ejecutiva: segmentadores sincronizados, navegador y matriz de los 8 KPIs con semáforo (PR #9, #10 y #18).

## Evidencia

- Historial de commits y Pull Requests del repositorio: https://github.com/joshuaqm11/Dashboard_Invacom_PowerBI/pulls?q=is%3Apr
- Tablero Kanban con las tareas de cada integrante: https://trello.com/b/1Hm2pCge/dashboard-invacom-proyecto-2-miner%C3%ADa-de-datos
- Validación de los 8 KPIs contra la línea base 2017: `docs/kpis.md` y capturas en `docs/evidencias/`.
