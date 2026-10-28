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

Antes de una tarea no trivial, indicá en 2-3 líneas su complejidad, el costo de un error y si requiere razonamiento profundo, y qué nivel de modelo corresponde. Si no es el de la sesión, sugerí cambiarlo con `/model` y esperá la decisión del usuario. Niveles, de menor a mayor costo: **Haiku** (tareas mecánicas), **Sonnet** (default: edición de documentos, `contexto/`, código simple), **Opus** (ADRs, coherencia entre documentos, trade-offs, desarrollo de la demo), **Fable** (criptografía, modelo de amenazas, revisiones integrales). Ante la duda, elegí el menor si el error es barato de corregir (texto versionado) y el mayor si es caro de detectar (diseño, seguridad).
