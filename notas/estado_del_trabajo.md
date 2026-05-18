# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Cierre de la pre-demo checklist y arranque del diseño específico de la demo. Las decisiones de diseño de plataforma están cerradas. Lo que queda antes de empezar a desarrollar es: ejecutar la pre-demo checklist (`notas/recordatorios.md`, sección "Antes de iniciar el desarrollo de la demo") y resolver las ocho decisiones de diseño específicas de la demo listadas más abajo.

## Decisiones de diseño pendientes

## Decisiones de diseño pendientes
 
No hay decisiones de diseño de plataforma pendientes. La etapa de cierre de decisiones de diseño previas al desarrollo de la demo queda completa salvo por la pre-demo checklist (ver `notas/recordatorios.md`, sección "Antes de iniciar el desarrollo de la demo").

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma. Incluye una decisión pendiente surgida durante el diseño de P-0014:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Simulación de AUTENTICAR en la demo. La demo no tiene acceso a credenciales reales de AUTENTICAR, lo cual exige algún mecanismo de simulación de la verificación de identidad. Decidir el alcance y la mecánica (lista de identidades ficticias, generación al vuelo u otra opción) y documentar la simulación en un ADR de demo (DP-XXXX).
6. Implementación de ZK en la demo. Decisión abierta. Implementarla cubre en la demo una parte sustantiva del sistema (verificación criptográfica de la legitimidad del `anon_id`) pero agrega complejidad significativa (trusted setup, integración con la simulación de AUTENTICAR, generación de pruebas en cada emisión). No implementarla simplifica la demo pero deja fuera una parte central del diseño. Si se decide no implementarla, la omisión debe documentarse en un DP-XXXX.
7. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **Análisis del supuesto conflicto P-0006 / P-0014 (pre-demo checklist).** Se verificó la sospecha de contradicción que planteaba el recordatorio "Revisión final de documentación de diseño". Dictamen: no hay contradicción. P-0006 decide el nivel de adversario del modelo de amenazas (adopta el modelo intermedio) y no toma ninguna decisión sobre ZK; la única mención de ZK en P-0006 es ilustrativa del modelo de adversario total descartado. P-0014 introduce ZK para un fin distinto —la auditabilidad criptográfica de la legitimidad del emisor, el problema del administrador malicioso— y lo hace dentro del modelo de amenazas intermedio de P-0006, sin contradecirlo. La relación de supersesión que sí correspondía (P-0014 supersede P-0013 Decisión 4) ya existía y estaba correctamente documentada. No se modificó ningún ADR: no hay redacción incorrecta que corregir ni supersesión que crear. El ejemplo de P-0006/P-0014 se quitó del recordatorio para no volver a inducir la confusión; la revisión completa de `design/` sigue pendiente.

- **Verificación léxica del repositorio (pre-demo checklist).** Dos verificaciones ejecutadas sobre todo el repo (excluido `README.md`, pendiente para cuando el proyecto esté completo). Verificación 1: grep de "AnonAgora" — sin correcciones fuera de `README.md`; todos los hits externos eran legítimos. Verificación 2: coherencia de vocabulario por capas — 13 reemplazos en 9 archivos. Correcciones: `demo/design/capa_de_identidad/adr/DP-0001` (1), `demo/design/capa_de_identidad/simulacion_autenticar.md` (5), `design/capa_de_identidad/adr/P-0002` (1), `design/capa_de_identidad/adr/P-0013` (1), `design/capa_de_identidad/identity_wordlists.md` (1), `notas/README.md` (1), `notas/propuesta_guia_de_instalacion.md` (2), `notas/recordatorios.md` (1). +Los usos de "la plataforma" en `docs/propuesta/` se identificaron pero no se corrigieron: 84 ocurrencias en 9 archivos (incluido el SVG de arquitectura conceptual), que quedan para la tarea separada de revisión de `docs/propuesta/`. Dos de los archivos corregidos en la verificación 2 son ADRs cerrados (P-0002, P-0013); los cambios fueron exclusivamente léxicos, sin alterar ninguna decisión.

- **`design/glosario.md` — Glosario del proyecto.** Entregable de la pre-demo checklist. Congela el vocabulario del proyecto con las decisiones de diseño de plataforma cerradas. Organizado en ocho categorías temáticas con referencias a los ADRs correspondientes y notas que distinguen términos confundibles. Normaliza la distinción "el sistema" (capa + aplicación destino del despliegue) vs "la plataforma" (forma abreviada de la aplicación de participación ciudadana), reconociendo el uso histórico ambiguo previo a P-0021. Aclara las relaciones entre `anon_seed`, `anon_id`, "identidad anónima" y "pseudónimo".

- **P-0023 — Moderación de contenido y retiro de propuestas.** ADR de la aplicación de participación ciudadana que formaliza tres cosas: (a) la ausencia de moderación humana de contenido más allá del revisor de lenguaje (P-0011); (b) la inmutabilidad del contenido de las propuestas publicadas, distinguida del retiro como operación distinta; (c) un mecanismo excepcional de retiro de propuestas activable solo por causales legales (catálogo configurable acotado a obligaciones legales del despliegue), ejecutable por autoridad judicial o por el operador ante contenido manifiestamente ilegal. La propuesta retirada conserva únicamente su `id`; el resto se reescribe a un tombstone con motivo categorizado. Los apoyos se eliminan, los vínculos salientes se eliminan, los vínculos entrantes se mantienen. La aplicación no admite retiro a pedido del autor (consecuencia de P-0001/P-0018/P-0015). El ADR deja registradas las capacidades y limitaciones del operador frente a requerimientos legales.

- **P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos.** Define la política de degradación, reintentos y comunicación al ciudadano ante fallos del verificador de identidad, del componente de proving ZK, del JWKS histórico y del revisor de lenguaje. Establece la atomicidad de la emisión respecto del cool-down (la tupla `{anon_seed, fecha_emision}` se persiste solo tras confirmación de la aplicación destino) y la política fail-closed para el revisor de lenguaje (publicación rechazada agotados los reintentos, sin persistencia del draft).

- **Gap 5 — Refactorización por arquitectura modular.** Cerrado completamente. Tres bloques de trabajo:
  - **Bloque 4** — reestructuración física del repositorio en subcarpetas por capa (capa de identidad / aplicaciones destino) y por ámbito (transversales / específicos).
  - **Bloque 4.5** — consolidación de la demo bajo `demo/` (incluyendo `demo/design/` y `demo/implementation/`), eliminando `design/demo/` e `implementation/demo/`. La regla de herencia de la demo respecto del sistema general queda documentada en `demo/README.md`.
  - **Bloque 5** — refactor de vocabulario en los 21 ADRs cerrados aplicando la regla "vocabulario y marco sí, decisiones no". 15 ADRs editados, 6 sin cambios.
  - **Bloque 6** — refactor de documentos descriptivos: `AGENTS.md`, `docs/architecture_overview.md`, READMEs de `design/`, `design/adr/`, `design/capa_de_identidad/` (incluyendo el contrato capa↔aplicación), READMEs minimales del Bloque 4 y los del refactor demo, `identity_model.md` (incorporación del principio "no sugerir memoria entre sesiones" y resolución de duplicación con P-0015), eliminación del solapamiento entre P-0004 y `identity_wordlists.md` preservando en P-0004 la decisión sobre normalización del pseudónimo en login. `identity_wordlists.md` movido a `design/capa_de_identidad/` junto con la carpeta `wordlists/`. Se creó `demo/design/adr/` (transversales DP) que faltaba en la estructura. Se saltó intencionalmente la revisión de `docs/propuesta/` por su naturaleza narrativa; cualquier ajuste se hará oportunamente bajo licencia descriptiva.

- **P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino.** Reconoce que el sistema está compuesto por una capa de identidad reutilizable y una aplicación destino, con el modelo "una capa ↔ una aplicación por despliegue". La aplicación de participación ciudadana queda como una aplicación destino entre otras posibles.
