# Sistema de contexto para agentes

Esta carpeta contiene el conocimiento del proyecto **destilado e indexado** para que las conversaciones nuevas con agentes alcancen una comprensión profunda sin releer el repositorio completo. Fue construida a partir de una lectura íntegra del repositorio (junio 2026) y se mantiene actualizada como parte del cierre de cada tarea.

## Por qué existe

Leer el repositorio completo para reconstruir contexto es costoso y casi nunca necesario. Este sistema reemplaza esa lectura por tres niveles de carga:

| Nivel | Archivo | Cuándo se carga | Costo aproximado |
|---|---|---|---|
| 0 — Bootstrap | `CLAUDE.md` (raíz) y/o `AGENTS.md` | Automático al iniciar la conversación | Mínimo |
| 1 — Ruteo | `contexto/mapa_de_contexto.md` | Apenas se entiende el pedido | Bajo |
| 2 — Comprensión global | `contexto/sintesis_del_proyecto.md` | Solo para tareas que requieren visión completa (diseñar, decidir, revisar) | Medio |
| 3 — Fuentes | ADRs y documentos del repo | Solo los que el mapa indica para la tarea | Variable |

La regla de oro: **cargar lo justo**. Si la tarea es puntual, el nivel 1 alcanza. Los documentos del nivel 3 son siempre la fuente de verdad; ante cualquier contradicción con la síntesis o el mapa, **prevalecen los ADRs** (regla de prioridad en `AGENTS.md`).

## Contenido

- `mapa_de_contexto.md` — ruteo por tipo de tarea + índice de una línea por documento del repo. Responde "¿qué tengo que leer para esta tarea?".
- `sintesis_del_proyecto.md` — la comprensión profunda destilada: propósito, arquitectura, mecanismos, principios, limitaciones aceptadas y hoja de ruta. Responde "¿de qué se trata todo esto y por qué es así?".

El **estado actual del trabajo no vive acá**: vive en `notas/estado_del_trabajo.md`, que ya cumple ese rol y se mantiene como siempre. La síntesis solo lo referencia.

## Reglas de mantenimiento (paso final de cada tarea)

Toda conversación que modifique el repositorio debe, antes de cerrar, verificar si su trabajo afecta este sistema y actualizarlo. El humano revisa el diff antes de commitear.

| Si la tarea... | Actualizar |
|---|---|
| Creó, modificó o supersedió un ADR | La línea de ese ADR en el índice de `mapa_de_contexto.md` (y el estado del ADR supersedido). |
| Agregó, movió o eliminó un documento del repo | El índice de documentos y, si corresponde, la tabla de ruteo de `mapa_de_contexto.md`. |
| Cambió la arquitectura, los principios, los mecanismos centrales o la etapa del proyecto | La sección correspondiente de `sintesis_del_proyecto.md` (y su fecha de actualización). |
| Cambió el estado del trabajo, decisiones pendientes o recordatorios | `notas/estado_del_trabajo.md` y/o `notas/recordatorios.md`, como siempre (no esta carpeta). |
| Cambió el protocolo de arranque mismo | `CLAUDE.md`, `AGENTS.md` y este README, manteniéndolos coherentes. |

Criterios:

- **No duplicar**: la síntesis condensa y explica; no copia texto de ADRs. Si un detalle solo importa a veces, va una línea en el mapa apuntando a la fuente, no un párrafo en la síntesis.
- **Sesgo a la brevedad**: cada línea de estos archivos se carga en futuras conversaciones. Antes de agregar, preguntarse si el mapa no lo resuelve con un puntero.
- **Vocabulario del glosario**: "el sistema" = capa + aplicación destino; "la plataforma" = la aplicación de participación ciudadana. El nombre provisorio del repositorio no se usa en la documentación (ver `AGENTS.md`).
