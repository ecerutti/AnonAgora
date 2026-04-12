# Notas

Esta carpeta contiene el material de trabajo del proyecto: documentos vivos que acompañan el proceso de diseño y desarrollo, pero que no forman parte de la documentación oficial del sistema ni de las decisiones ya tomadas.

Es el espacio donde las ideas viven antes de convertirse en documentos oficiales (ADRs, documentos de diseño, documentación conceptual), o donde persisten como apoyo al proceso de trabajo.

El contenido es mantenido por el humano con apoyo de los agentes cuando corresponda. Su autoría final siempre es del humano.

## Qué va acá

- Estado del trabajo en curso y decisiones pendientes.
- Recordatorios de cosas a hacer en momentos específicos del desarrollo.
- Ideas sueltas y anotaciones que todavía no tienen decisión asociada.
- Borradores conceptuales de documentos que aún no están maduros para su lugar definitivo.

## Qué NO va acá

- Decisiones ya tomadas: van a `design/adr/` (plataforma) o `design/demo/adr/` (demo).
- Descripciones de componentes o modelos del sistema: van a `design/`.
- Documentación conceptual de la propuesta: va a `docs/propuesta/`.
- Scratchpads o notas internas que los agentes generan durante una conversación para sí mismos: esos no pertenecen al repo, deben permanecer en el workspace interno del agente.

## Tipos de documentos

Los archivos de esta carpeta se pueden agrupar conceptualmente en dos tipos, aunque no se separan en subcarpetas:

### Documentos de proceso

Acompañan el trabajo del proyecto a lo largo del tiempo y se van actualizando. No tienen un horizonte de migración.

- `estado_del_trabajo.md` — estado actual del trabajo, decisiones cerradas recientemente y pendientes por abordar. Es el primer archivo que debe leer cualquier conversación nueva con un agente.
- `recordatorios.md` — cosas a recordar en momentos específicos del desarrollo, agrupadas por cuándo deben activarse.
- `anotaciones_e_ideas_de_trabajo.md` — cuaderno abierto de ideas sueltas, analogías, pensamientos en proceso y cuestiones pendientes de evaluación que todavía no ameritan una decisión formal.

### Documentos de gestación

Borradores conceptuales de documentos que eventualmente migrarán a su lugar definitivo en el repo (`design/`, `design/adr/`, `docs/`, etc.) cuando estén maduros. Cuando un documento de gestación migra, deja de existir en `notas/`.

- `autenticacion_autenticar.md` — borrador técnico sobre integración con AUTENTICAR para verificación de identidad. Su destino probable es un documento de diseño en `design/` o un insumo para un ADR futuro.
- `propuesta_guia_de_instalacion.md` — material en gestación para una futura guía de instalación y operación dirigida a administradores de despliegues de la plataforma.

## Regla práctica

Si una idea todavía tiene alternativas abiertas o dudas sin resolver, vive como nota en esta carpeta. Cuando las dudas se resuelven y solo falta redactar el resultado, salta directamente a su lugar oficial: un ADR en `design/adr/` si decide entre alternativas, un documento en `design/` si describe un modelo o componente.
