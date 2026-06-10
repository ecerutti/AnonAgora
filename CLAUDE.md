# Arranque de conversación

Este proyecto diseña una **capa de identidad anónima verificada** y, sobre ella, una **aplicación de participación ciudadana** ("termómetro social"). Es un repositorio principalmente documental; las reglas completas para agentes viven en `AGENTS.md`.

**Protocolo de arranque (obligatorio):**

1. **No leas el repositorio completo.** El conocimiento ya está destilado e indexado para carga a demanda.
2. Cuando entiendas qué te están pidiendo, leé `contexto/mapa_de_contexto.md` y cargá **solo** los documentos que el mapa indica para ese tipo de tarea.
3. Si la tarea requiere comprensión global del proyecto (diseñar, decidir, revisar), leé además `contexto/sintesis_del_proyecto.md`.
4. Antes de **modificar** el repositorio, leé `AGENTS.md` (reglas y estructura) y `notas/estado_del_trabajo.md` (estado actual).
5. Al cerrar una tarea que cambie decisiones, documentos o estado del proyecto, actualizá el sistema de contexto según `contexto/README.md`.

Para preguntas puntuales que el mapa resuelve directo (un dato, un archivo, un fix), no cargues la síntesis: el mapa alcanza.

## Evaluación de modelo antes de tareas no triviales

Al recibir una tarea **no trivial**, antes de empezar, evaluá brevemente y de forma visible para el usuario (2-3 líneas): **complejidad de la tarea**, **riesgo/impacto de un error** y **necesidad de razonamiento profundo**. Con eso determiná el modelo óptimo según la guía de abajo. Si el modelo actual de la sesión no es el óptimo, decilo y **sugerí cambiarlo con `/model` antes de continuar**, esperando la decisión del usuario. Si el modelo actual ya es el adecuado, indicálo en una línea y seguí. Las tareas triviales (un typo, una pregunta directa, un dato del mapa) se ejecutan directo, sin esta evaluación.

Guía de selección (costo por millón de tokens entrada/salida, junio 2026):

| Modelo | Costo | Usar para |
|---|---|---|
| Haiku 4.5 | $1 / $5 | Tareas mecánicas y acotadas: typos, renombres, búsquedas simples, formateo, cambios de una línea. |
| Sonnet 4.6 | $3 / $15 | El default para tareas estándar: edición de documentos, fixes multi-archivo, actualizaciones de `contexto/`, código simple sin decisiones de diseño. |
| Opus 4.8 | $5 / $25 | Razonamiento sólido: redactar o revisar ADRs, verificar coherencia entre documentos, decisiones con trade-offs, debugging complejo, desarrollo de la demo. |
| Fable 5 | $10 / $50 | Solo lo que lo justifica: arquitectura crítica, criptografía y modelo de amenazas, revisiones integrales del repositorio, tareas largas y autónomas de gran alcance. |

Criterio general: ante la duda entre dos niveles, elegí el menor si el error es barato de corregir (texto, documentación versionada) y el mayor si el error es caro de detectar (decisiones de diseño, seguridad, criptografía).
