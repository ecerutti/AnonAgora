# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Orden sugerido de abordaje:

1. **Modelo de datos del emisor y ciclo de vida de identidades anónimas (P-0015).** Decisiones cerradas que esperan ser formalizadas en ese ADR:
   - Cool-down de 6 meses contado desde la fecha de emisión de la identidad anónima. Valor configurable por el operador, default 6 meses.
   - No se establece límite a la cantidad de renovaciones de por vida. El cool-down es el único freno.
   - El secuestro de identidad queda fuera del modelo de amenazas: el impacto es acotado y la mitigación técnica completa requeriría complejidad criptográfica incompatible con el modelo de amenazas intermedio de P-0006.
   - Firma del emisor sobre cada `anon_id` para habilitar auditoría de legitimidad en la plataforma participativa de forma independiente al emisor.

   Decisiones abiertas a discutir en la conversación de P-0015:
   - **Tupla del emisor.** Candidato a evaluar: `{anon_seed, anon_id, fecha_emision, prueba_zk}` más el mecanismo de firma sobre `anon_id`. Definir exactamente qué campos entran y por qué, resolviendo explícitamente: auditoría de legitimidad, detección de identidades falsas en la plataforma participativa, y recuperación de pseudónimo olvidado. La inclusión de la prueba ZK en la tupla debe considerarse a la luz de P-0014.
   - **Pérdida de frase secreta vs. revocación voluntaria.** Evaluar si comparten el mismo mecanismo técnico en el emisor o si requieren flujos diferenciados.

   Consecuencias dependientes de la tupla final (a confirmar en P-0015):
   - Con tupla `{anon_seed, anon_id, fecha_emision}`: el emisor puede devolver la `anon_id` actual al ciudadano que olvidó su pseudónimo,
