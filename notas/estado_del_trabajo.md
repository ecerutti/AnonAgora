# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño del sistema: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

1. **P-0022 — Comportamiento ante fallos de servicios externos.** ADR que defina el comportamiento del sistema ante fallos o indisponibilidad de servicios externos: verificador de identidad caído, token vencido en tránsito, componente de proving ZK con demoras o error, API del revisor de lenguaje caída o con error. Debe cubrir principios generales de degradación, reintentos y comunicación al ciudadano.

2. **P-0023 — Moderación de contenido y retiro de propuestas.** ADR que formalice la ausencia de moderación de contenido más allá del revisor de lenguaje (P-0011), y que defina cómo se maneja el caso de propuestas que deben retirarse por orden judicial, contenido ilegal, o solicitudes de ciudadanos con implicancias legales (derecho al olvido). Debe dejar explícitas las limitaciones derivadas del diseño actual y las implicancias legales para el operador que despliega el sistema.

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

- **Gap 5 — Refactorización por arquitectura modular.** Cerrado completamente. Tres bloques de trabajo:
  - **Bloque 4** — reestructuración física del repositorio en subcarpetas por capa (capa de identidad / aplicaciones destino) y por ámbito (transversales / específicos).
  - **Bloque 4.5** — consolidación de la demo bajo `demo/` (incluyendo `demo/design/` y `demo/implementation/`), eliminando `design/demo/` e `implementation/demo/`. La regla de herencia de la demo respecto del sistema general queda documentada en `demo/README.md`.
  - **Bloque 5** — refactor de vocabulario en los 21 ADRs cerrados aplicando la regla "vocabulario y marco sí, decisiones no". 15 ADRs editados, 6 sin cambios.
  - **Bloque 6** — refactor de documentos descriptivos: `AGENTS.md`, `docs/architecture_overview.md`, READMEs de `design/`, `design/adr/`, `design/capa_de_identidad/` (incluyendo el contrato capa↔aplicación), READMEs minimales del Bloque 4 y los del refactor demo, `identity_model.md` (incorporación del principio "no sugerir memoria entre sesiones" y resolución de duplicación con P-0015), eliminación del solapamiento entre P-0004 y `identity_wordlists.md` preservando en P-0004 la decisión sobre normalización del pseudónimo en login. `identity_wordlists.md` movido a `design/capa_de_identidad/` junto con la carpeta `wordlists/`. Se creó `demo/design/adr/` (transversales DP) que faltaba en la estructura. Se saltó intencionalmente la revisión de `docs/propuesta/` por su naturaleza narrativa; cualquier ajuste se hará oportunamente bajo licencia descriptiva.

- **P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino.** Reconoce que el sistema está compuesto por una capa de identidad reutilizable y una aplicación destino, con el modelo "una capa ↔ una aplicación por despliegue". La aplicación de participación ciudadana queda como una aplicación destino entre otras posibles.
