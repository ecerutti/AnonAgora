# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Orden sugerido de abordaje:

1. **Límite anual de propuestas por ciudadano.** Decidir la existencia del límite como propiedad del sistema (configurable, cómo se cuenta el año, qué pasa con propuestas derivadas, posibilidad de configurar 0 o desactivarlo). El valor concreto "2 por año" que aparece en la documentación conceptual es el default sugerido, no la decisión de fondo.

2. **Modelo de datos de propuestas.** Qué campos tiene una propuesta, longitud máxima, formato del cuerpo, si admite imágenes o links.

3. **Búsqueda de propuestas.** Tipo de búsqueda (texto plano, tags, categorías), si hay categorización temática.

4. **Vinculación entre propuestas.** Tipos de vínculo (deriva de, mejora a, integra a), direccionalidad, si requieren aceptación del autor original.

5. **Política de logs y retención de metadatos.** Concretar lo que P-0006 exige pero no fija: qué se registra, con qué granularidad temporal, por cuánto tiempo.

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Manejo de la identidad anónima en la demo (simulación de la verificación de unicidad).
6. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
7. Autenticación administrativa para gestión técnica de la demo.
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/autenticacion_autenticar.md` — borrador técnico sobre integración con AUTENTICAR para verificación de identidad y emisión de identidades anónimas. Cuando esté maduro, este documento debe migrar desde `notas/` a su lugar definitivo: probablemente como documento de diseño en `design/` (sobre integración con proveedores de verificación) o como insumo de un ADR futuro. Cuando se haga esa migración, el archivo debe dejar de existir en `notas/`.

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores. No es un entregable del proyecto todavía, pero acumula material para cuando llegue el momento.

## Últimas decisiones cerradas

- **Ciclo de vida de identidades anónimas** — decisiones de fondo cerradas, pendiente redacción de ADR (la próxima conversación decide si va uno o dos documentos). Decisiones:
  - El emisor guarda únicamente la tupla `{anon_seed, fecha_emision}` por ciudadano. Sin pseudónimo, sin historial, sin estado, sin contador.
  - Cool-down de 6 meses contado desde `fecha_emision`. Cuando llega un pedido de nueva identidad, el emisor compara la fecha guardada contra el momento actual; si pasaron 6 meses o más, emite nueva identidad y reemplaza la fila con la nueva fecha. Valor configurable por el operador, default 6 meses.
  - No se establece límite a la cantidad de veces que un ciudadano puede obtener nuevas identidades a lo largo de su vida. Se podría haber implementado con un campo extra en la tupla; se decidió explícitamente no hacerlo. El cool-down es el único freno.
  - Pérdida de frase secreta y revocación voluntaria comparten el mismo mecanismo técnico. Un único flujo en el emisor, aunque la UX pueda diferenciarlos.

  Consecuencias (a documentar en la sección correspondiente del ADR):
  - No hay recuperación del pseudónimo olvidado. Perder frase o pseudónimo deja la identidad irrecuperable (simetría con P-0008).
  - Propuestas y apoyos de identidades anteriores quedan huérfanos: permanecen, cuentan, no pueden modificarse ni retirarse.
  - Los contadores del límite anual se reinician de facto con cada nueva identidad.
  - La plataforma participativa no puede distinguir identidades activas de huérfanas.
  
- P-0012 — Mecanismo de apoyo a propuestas (apoyo binario retractable, apoyo automático del autor, conteo público).
- README de `design/adr/` actualizado con criterio claro para determinar cuándo corresponde un ADR.
- `AGENTS.md` actualizado con tabla consolidada de carpetas y reconocimiento de `notas/` como material de trabajo del proyecto.
