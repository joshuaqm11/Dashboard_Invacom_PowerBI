# Sprint 1 · Arranque del Proyecto #2

**Scrum Master:** Joshua
**Fechas:** 16/11/2026 al 20/11/2026
**Objetivo:** repo listo, Power BI conectado al DW en las 4 máquinas y modelo semántico base subido a `main`.
**Kanban:** https://trello.com/b/1Hm2pCge/dashboard-invacom-proyecto-2-miner%C3%ADa-de-datos

## Sprint Backlog

| Tarjeta | Responsable | Estado |
|---|---|---|
| Crear repo, subir estructura, Kanban y proteger `main` | Joshua | Hecho (repo y estructura; el Kanban quedó en Trello) |
| Instalar Power BI Desktop y OCMT de 64 bits; probar conexión a `DW_INVACOM` | Todos | Hecho |
| Crear el proyecto `.pbip`, importar tablas y crear relaciones | Joshua | Hecho (PR #1) |
| Marcar `DIM_TIEMPO` como tabla de fechas y ordenar `nombre_mes` por `mes` | Joshua | Hecho (PR #1) |
| Crear tabla de metas (8 KPIs, umbrales del semáforo) | Fernanda | Hecho por Joshua (PR #3) |
| Investigación de la metodología para diseñar KPIs (Enunciado I-a) | Fernanda y Joseph | Hecho (PR #5 y #16) |
| Probar que el visual de mapa funciona en su máquina | Fabio | Hecho (PR #15) |
| Probar instalar un visual de AppSource (gauge) | Joshua | Hecho (Tachometer, PR #19) |

## Daily (asíncrono)
Registrado en WhatsApp.

## Cierre del sprint (Sprint Review)
- **Completado:** repositorio con estructura y plantilla de PR; proyecto PBIP conectado a `DW_INVACOM` en las 4 máquinas; modelo base con tabla de fechas, claves ocultas y tablas de medidas (PR #1); medidas de Ventas, tabla `Metas KPI` y primera versión de la página Ventas (PR #2, #3 y #4).
- **Pendiente / bloqueado:** ningún bloqueo. Las páginas de Finanzas, Logística y Clientes pasaron al Sprint 2.
- **Decisiones tomadas:**
  - Una rama por tarea y PR a `main` con la plantilla; cada integrante trabaja solo en su tabla de medidas y su página.
  - La tabla `Metas KPI` se creó en el modelo base para que los 4 integrantes usen las mismas metas y umbrales del semáforo.
  - Los datos no se versionan: cada quien actualiza desde su Oracle local.
