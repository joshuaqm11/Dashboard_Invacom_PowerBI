# Dashboard de KPIs gerenciales — DW Invacom · Proyecto #2 Minería de Datos (3C 2026)

Universidad Técnica Nacional · ISW-911

Tablero de control en **Power BI** conectado al Data Warehouse `DW_INVACOM` construido en el Proyecto #1
(repo: [DataWarehouse_Invacom](https://github.com/joshuaqm11/DataWarehouse_Invacom)).
Muestra los 8 KPIs gerenciales aprobados por el profesor (2 por integrante).

## Equipo y KPIs

| Integrante | KPIs | Página del tablero | Requisito transversal |
|---|---|---|---|
| Joshua | KPI-VEN-01 Ventas promedio mensuales · KPI-VEN-02 Órdenes promedio mensuales | Ventas | Modelo semántico base, integración de PRs, visual AppSource #1 |
| Fernanda | KPI-FIN-01 Ticket promedio · KPI-FIN-02 Peso del flete | Finanzas | Tabla de metas, semáforo, visual AppSource #2, tema visual |
| Fabio | KPI-LOG-01 % entregas a tiempo · KPI-LOG-02 Días promedio de entrega | Logística | Mapa |
| Joseph | KPI-CLI-01 % reseñas 4–5 · KPI-CLI-02 Tasa de cancelación | Clientes | Portada ejecutiva, árbol de descomposición |

Detalle de cada KPI: `docs/kpis.md`. Medidas DAX explicadas: `docs/medidas_dax.md`.
Participación del equipo: `docs/participacion.md`. Actas de sprint: `docs/sprints/`.

**Kanban (Trello):** https://trello.com/b/1Hm2pCge/dashboard-invacom-proyecto-2-miner%C3%ADa-de-datos

## Estructura del repositorio

```
powerbi/      proyecto de Power BI en formato PBIP (Dashboard_Invacom.pbip + carpetas .SemanticModel y .Report)
temas/        tema visual de Power BI (JSON)
docs/         KPIs, medidas DAX, actas de sprint, participación y evidencias (capturas)
```

## Puesta en marcha (cada integrante)

Requisitos:
- DW del Proyecto #1 cargado en **su** Oracle local (`DW_INVACOM`, servicio `XEPDB1`).
- **Power BI Desktop** (Windows) actualizado.
- **Oracle Client for Microsoft Tools (OCMT) de 64 bits** (Power BI es de 64 bits; si Office es de 32 bits, el OCMT del P1 puede no servir).

Pasos:
1. `git clone https://github.com/joshuaqm11/Dashboard_Invacom_PowerBI.git` y luego `git pull` siempre antes de trabajar.
2. Abrir `powerbi/Dashboard_Invacom.pbip` con Power BI Desktop.
3. **Inicio → Actualizar**. Credenciales: *Base de datos*, usuario `DW_INVACOM`.
4. Si su servicio no es `XEPDB1`: **Transformar datos → Editar parámetros** y cambiar `pServidorOracle` (no subir ese cambio).

Los datos **no** se versionan: cada quien los carga desde su Oracle (el `.gitignore` excluye la caché `cache.abf`).

## Forma de trabajo en Git

1. Crear una rama por tarea: `git checkout -b <nombre>/<tarea>` (ej. `fabio/medidas-logistica`).
2. Trabajar **solo** en su tabla de medidas y su página.
3. Guardar en Power BI, cerrar Power BI, `git add`, `git commit`, `git push`.
4. Abrir un Pull Request a `main` usando la plantilla. Joshua revisa e integra.
5. Después de cada merge, todos hacen `git pull` en `main`.

Regla: nunca editar la tabla de medidas o la página de otro integrante sin avisar en el grupo.
