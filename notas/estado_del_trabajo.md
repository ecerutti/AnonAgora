# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Todas las pendientes son consecuencia del checkpoint realizado sobre el repositorio. Deben abordarse en conversaciones dedicadas en este orden:

1. **Gap 5 — Alcance y generalización del diseño.** Decisión de fondo: el proyecto se acota al ámbito argentino (descartado multi-idioma y multi-verificador como objetivos inmediatos), pero se incorpora la intención de hacer la plataforma lo suficientemente modular para que el componente de destino pueda intercambiarse a futuro dentro de ese mismo ámbito. Ejemplo: hoy la aplicación final es la plataforma de participación ciudadana, mañana podrían agregarse aplicaciones como una plataforma de denuncias anónimas que reutilice la infraestructura de identidad anónima verificada. La conversación debe revisar la documentación y el diseño existentes para evaluar qué ajustes son necesarios para habilitar esa modularidad desde el diseño, sin rediseñar lo ya decidido. Se trata antes que los siguientes dos ADRs porque sus decisiones pueden afectar cómo se redactan.

2. **P-0022 — Comportamiento ante fallos de servicios externos.** ADR que defina el comportamiento del sistema ante fallos o indisponibilidad de servicios externos: verificador de identidad caído, token vencido en tránsito, componente de proving ZK con demoras o error, API del revisor de lenguaje caída o con error. Debe cubrir principios generales de degradación, reintentos y comunicación al ciudadano.

3. **P-0021 — Moderación de contenido y retiro de propuestas.** ADR que formalice la ausencia de moderación de contenido más allá del revisor de lenguaje (P-0011), y que defina cómo se maneja el caso de propuestas que deben retirarse por orden judicial, contenido ilegal, o solicitudes de ciudadanos con implicancias legales (derecho al olvido). Debe dejar explícitas las limitaciones derivadas del diseño actual y las implicancias legales para el operador que despliega el sistema.

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
