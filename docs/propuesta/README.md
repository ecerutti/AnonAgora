# Propuesta conceptual

Esta carpeta contiene la documentación conceptual del proyecto. Está orientada a lectores no técnicos — responsables institucionales, decisores políticos, asesores, personas interesadas en comprender el objetivo del proyecto. No registra decisiones técnicas ni especificaciones de implementación; esas viven en `design/` e `implementation/`.

Los documentos están organizados en un orden de lectura sugerido que va de la idea general a los detalles técnicos.

Esta carpeta describe una aplicación destino concreta: la plataforma de participación ciudadana. Esa aplicación está construida sobre una infraestructura de identidad anónima verificada reutilizable, que maneja la verificación de cada ciudadano y la emisión de su identidad anónima. Los documentos aquí no describen esa infraestructura; describen lo que el ciudadano puede hacer con ella.

## Documentos

| # | Documento | Propósito | Destinatario principal | Estilo |
|---|---|---|---|---|
| 01 | El Problema y la Idea | Qué problema resuelve la propuesta y cuál es la idea central. | Público general, decisores. | Narrativo, no técnico. |
| 02 | Fundamentos y Lógica de funcionamiento | Por qué el sistema podría funcionar sin perder anonimato ni orden. | Público general, decisores. | Narrativo, no técnico. |
| 03 | Cómo se usaría | Historia cotidiana de un ciudadano usando la plataforma. | Público general. | Narrativo, ilustrativo. |
| 04 | Riesgos, límites y preocupaciones razonables | Alcances y límites de la propuesta, respuestas a objeciones habituales. | Decisores, asesores. | Argumentativo, no técnico. |
| 05 | Cómo podría implementarse | Por qué la idea es técnicamente viable con herramientas actuales. | Asesores técnicos de decisores. | Semi-técnico, accesible. |
| 06 | Arquitectura técnica y desafíos | Componentes, riesgos técnicos y estrategias de mitigación. | Asesores técnicos, perfiles de ingeniería. | Técnico. |
| 00 | Resumen Ejecutivo | La propuesta en pocas páginas. | Decisores políticos, responsables institucionales. | No técnico, conciso. |

## Notas sobre el uso de esta documentación

Estos documentos son conceptuales y tienen cierta licencia narrativa. Cuando algo aquí descrito se diferencia de las decisiones técnicas registradas en `design/` o `implementation/`, prevalecen estas últimas. En caso de contradicción, ver la regla de prioridad de interpretación en `AGENTS.md`.
