# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Todas las pendientes son consecuencia del checkpoint realizado sobre el repositorio. Deben abordarse en conversaciones dedicadas en este orden:

1. **Refactorización por arquitectura modular (Gap 5 en curso).** El ADR P-0021 está cerrado y formaliza la estructura del sistema en dos capas: capa de identidad (infraestructura de identidad anónima verificada) y aplicación destino. La reestructuración física del repositorio (Bloque 4) está hecha; los ADRs y documentos descriptivos viven en sus nuevas carpetas. Pendientes:
   - Refactor de vocabulario en ADRs cerrados: aplicar la regla de "vocabulario y marco sí, decisiones no" para distinguir capa, aplicación y sistema. Casos especiales: el principio estructural de P-0005 ("no sugerir memoria entre sesiones") se absorbe en `identity_model.md`; P-0020 se clarifica por capa.
   - Refactor de documentos descriptivos: `AGENTS.md` (Propósito y tabla de estructura del repo), `docs/architecture_overview.md` (reescritura), `design/README.md`, `design/adr/README.md`, READMEs nuevos creados en Bloque 4, y los READMEs preexistentes en `implementation/` (preservados sin tocar durante Bloque 4). Documentar el contrato capa↔aplicación en `design/capa_de_identidad/README.md`.
   - Ajustes menores en `docs/propuesta/` para enmarcar la propuesta como descripción de una aplicación destino específica.
   - Cierre: grep de vocabulario residual, validación de coherencia.

2. **P-0022 — Comportamiento ante fallos de servicios externos.** ADR que defina el comportamiento del sistema ante fallos o indisponibilidad de servicios externos: verificador de identidad caído, token vencido en tránsito, componente de proving ZK con demoras o error, API del revisor de lenguaje caída o con error. Debe cubrir principios generales de degradación, reintentos y comunicación al ciudadano.

3. **P-0023 — Moderación de contenido y retiro de propuestas.** ADR que formalice la ausencia de moderación de contenido más allá del revisor de lenguaje (P-0011), y que defina cómo se maneja el caso de propuestas que deben retirarse por orden judicial, contenido ilegal, o solicitudes de ciudadanos con implicancias legales (derecho al olvido). Debe dejar explícitas las limitaciones derivadas del diseño actual y las implicancias legales para el operador que despliega el sistema.

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma. Incluye una decisión pendiente surgida durante el diseño de P-0014:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Manejo de la identidad anónima en la demo (simulación de la verificación de unicidad). La demo no usa AUTENTICAR real ni implementa ZK; ambas omisiones deben quedar documentadas en un ADR de demo (DP-XXXX) que explique por qué no aplican en ese contexto.
6. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
7. Autenticación administrativa para gestión técnica de la demo.
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino.** Cierra Gap 5 a nivel decisión. Reconoce que el sistema está compuesto por una capa de identidad reutilizable y una aplicación destino, con el modelo "una capa ↔ una aplicación por despliegue". La aplicación de participación ciudadana queda como una aplicación destino entre otras posibles. Las consecuencias estructurales (refactor del repositorio, refactor de vocabulario en ADRs y documentos) están pendientes de ejecución (ver decisiones pendientes).
