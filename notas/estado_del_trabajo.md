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

- **ADR de integración con AUTENTICAR (P-0013)** — pendiente de redacción. Es el próximo ADR a trabajar (en conversación nueva).
  El documento `notas/autenticacion_autenticar.md` está maduro y listo para migrar. Decisiones clave a formalizar en ese ADR:
  - Restringir proveedores aceptados a ARCA y ANSES únicamente. La razón es que ambos entregan CUIT/CUIL, que es el mismo espacio de identificadores: la misma persona tiene el mismo número en ambos proveedores. Esto permite construir un `anon_seed` estable e independiente del proveedor usado en cada autenticación.
  - Fórmula del `anon_seed`: `HASH(salt_del_sistema + CUIT/CUIL)`. El proveedor no entra al hash porque el identificador resultante es el mismo independientemente de si el ciudadano usó ARCA o ANSES.Usar cualquier otro proveedor (ReNaPer con DNI, MiArgentina con pasaporte) rompería la unicidad porque sus identificadores son distintos al CUIT/CUIL.
  - No usar el `sub` del token como base del `anon_seed`: el `sub` varía entre reinos de Keycloak y no es confiable como identificador universal entre proveedores.
  - Registrar el `jti` del token de AUTENTICAR en el emisor para auditoría: permite correlacionar cada identidad emitida con un evento de autenticación real sin almacenar identidad real.
  - Verificación offline de firmas mediante JWKS público de AUTENTICAR (endpoint por reino confirmado en producción).
  Al redactar P-0013, migrar `notas/autenticacion_autenticar.md` dividiéndolo en dos destinos: la parte descriptiva (qué es AUTENTICAR, endpoints, claims, JWKS, flujo OAuth2/OIDC) va a `docs/autenticar.md`; las decisiones van al cuerpo del ADR. El archivo en `notas/` se elimina una vez migrado.

- **Modelo de datos del emisor y ciclo de vida de identidades anónimas
  (P-0014)** — pendiente de redacción. Se trabaja después de P-0013, ya que se apoya en decisiones que ese ADR formaliza.

  Decisiones cerradas:
  - Cool-down de 6 meses contado desde la fecha de emisión de la identidad anónima. Valor configurable por el operador, default 6 meses.
  - No se establece límite a la cantidad de renovaciones de por vida. El cool-down es el único freno.
  - El secuestro de identidad queda fuera del modelo de amenazas: el impacto es acotado (el ladrón solo puede hacer lo que haría
    cualquier ciudadano) y la mitigación técnica completa requeriría complejidad criptográfica incompatible con el modelo de amenazas
    intermedio de P-0006.

  Decisiones abiertas a discutir en la conversación de P-0014:
  - **Tupla del emisor.** La tupla mínima `{anon_seed, fecha_emision}` fue considerada inicialmente por sus ventajas: máxima minimización de datos (coherente con el principio de minimización de P-0006), superficie de correlación mínima en el emisor si es comprometido de forma aislada, y simplicidad de implementación. Fue descartada porque no resuelve auditoría de legitimidad de identidades, no permite detectar identidades falsas generadas por un admin malicioso, y no permite recuperar el pseudónimo olvidado. Candidato a evaluar: `{anon_seed, anon_id, fecha_emision}` más, a evaluar, información de las firmas entregadas por AUTENTICAR (por ejemplo el `jti`) para habilitar auditoría sin cruzar sistemas. Definir exactamente qué campos entran y por qué, resolviendo explícitamente: auditoría de legitimidad, detección de identidades falsas, y recuperación de pseudónimo olvidado.
  - **Pérdida de frase secreta vs. revocación voluntaria.** Evaluar si comparten el mismo mecanismo técnico en el emisor o si requieren flujos diferenciados. Discutir las alternativas de UX y sus implicaciones técnicas antes de cerrar.

  Consecuencias dependientes de la tupla final (a confirmar en P-0014):
  - Con tupla extendida `{anon_seed, anon_id, fecha_emision}`: el emisor puede devolver la `anon_id` actual al ciudadano que olvidó su pseudónimo, autenticándose nuevamente vía AUTENTICAR. La identidad anónima es recuperable; la frase secreta no lo es.
  - La trazabilidad `anon_seed → anon_id` permite que la plataforma mantenga el historial completo de la identidad: propuestas, apoyos y contadores del límite anual no se pierden ni se resetean con una renovación.
  - Propuestas y apoyos de identidades anteriores no quedan huérfanos si la trazabilidad se preserva.
  - A confirmar: si la plataforma puede o no distinguir identidades activas de renovadas, y qué implicaciones tiene eso.
