# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Orden sugerido de abordaje:

1. **Evaluación de ZK para auditoría de legitimidad del emisor (decisión nueva).** La Decisión 4 de P-0013 adoptó temporalmente auditoría procedimental sin retención de metadatos, aceptando que no es posible detectar identidades ficticias fabricadas por un admin malicioso mediante auditoría forense sobre datos. La solución criptográfica a ese problema son las pruebas de conocimiento cero (ZK) aplicadas a tokens JWT (proyectos de referencia: zkLogin, zk-JWT). Esta evaluación requiere conversación propia con contexto limpio: analizar complejidad técnica, madurez de librerías disponibles, impacto en el modelo de amenazas de P-0006, y decidir si se adopta o no. Si se adopta, P-0013 Decisión 4 debe revisarse.

2. **Modelo de datos del emisor y ciclo de vida de identidades anónimas (P-0014).** Se trabaja después de cerrar o posponer explícitamente la evaluación de ZK, ya que P-0014 incluye la decisión de firma del emisor sobre cada `anon_id` para auditoría en la plataforma participativa, que es independiente de ZK y puede cerrarse sin esperarlo.

3. **Límite anual de propuestas por ciudadano.** Decidir la existencia del límite como propiedad del sistema (configurable, cómo se cuenta el año, qué pasa con propuestas derivadas, posibilidad de configurar 0 o desactivarlo). El valor concreto "2 por año" que aparece en la documentación conceptual es el default sugerido, no la decisión de fondo.

4. **Modelo de datos de propuestas.** Qué campos tiene una propuesta, longitud máxima, formato del cuerpo, si admite imágenes o links.

5. **Búsqueda de propuestas.** Tipo de búsqueda (texto plano, tags, categorías), si hay categorización temática.

6. **Vinculación entre propuestas.** Tipos de vínculo (deriva de, mejora a, integra a), direccionalidad, si requieren aceptación del autor original.

7. **Política de logs y retención de metadatos.** Concretar lo que P-0006 exige pero no fija: qué se registra, con qué granularidad temporal, por cuánto tiempo.

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

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **ADR P-0013 — Integración con AUTENTICAR.** Cerrado. Decisiones formalizadas:
  - Se usa AUTENTICAR como proveedor de verificación de identidad.
  - Se aceptan únicamente ARCA y ANSES como proveedores de identidad. La razón es que ambos entregan CUIT/CUIL, que es el mismo espacio de identificadores: la misma persona tiene el mismo número en ambos proveedores, lo que permite construir un `anon_seed` estable e independiente del proveedor usado en cada autenticación. Cualquier otro proveedor disponible en AUTENTICAR (ReNaPer, Mi Argentina, NIC.ar) usa identificadores en espacios distintos y rompe la unicidad.
  - Fórmula del `anon_seed`: `HASH(salt_del_sistema + CUIT/CUIL)`. El claim a extraer es `cuit`. No se usa `sub` (varía entre reinos) ni `preferred_username` (inestable entre proveedores).
  - Nivel mínimo requerido: nivel 2. El criterio es el nivel mínimo que garantice verificación de persona real con credenciales estatales activas.
  - No se retiene ningún metadato del token. La auditoría del emisor es procedimental hasta que se evalúe ZK. Esta es una decisión temporal explícita con limitaciones documentadas en el ADR.
  - La auditoría de legitimidad en la plataforma participativa se resuelve en P-0014 mediante firma del emisor sobre cada `anon_id`.
  - `notas/autenticacion_autenticar.md` fue migrado y debe eliminarse: la parte descriptiva de AUTENTICAR migró a `docs/autenticar.md` y las decisiones al ADR.
  - `docs/autenticar.md` fue creado como documento de referencia técnica agnóstico del sistema. Está pendiente de completar con información de implementación (estructura real del JWT, ejemplos de requests/responses, manejo de errores, scopes, ambiente de testing) mediante investigación con el plugin de Chrome usando el prompt preparado en esta conversación.

- **Modelo de datos del emisor y ciclo de vida de identidades anónimas (P-0014)** — pendiente de redacción. Se trabaja después de la evaluación de ZK o después de decidir explícitamente posponerla. Decisiones cerradas que esperan ser formalizadas en ese ADR:
  - Cool-down de 6 meses contado desde la fecha de emisión de la identidad anónima. Valor configurable por el operador, default 6 meses.
  - No se establece límite a la cantidad de renovaciones de por vida. El cool-down es el único freno.
  - El secuestro de identidad queda fuera del modelo de amenazas: el impacto es acotado y la mitigación técnica completa requeriría complejidad criptográfica incompatible con el modelo de amenazas intermedio de P-0006.
  - Firma del emisor sobre cada `anon_id` para habilitar auditoría de legitimidad en la plataforma participativa de forma independiente al emisor.

  Decisiones abiertas a discutir en la conversación de P-0014:
  - **Tupla del emisor.** Candidato a evaluar: `{anon_seed, anon_id, fecha_emision}` más el mecanismo de firma sobre `anon_id`. Definir exactamente qué campos entran y por qué, resolviendo explícitamente: auditoría de legitimidad, detección de identidades falsas en la plataforma participativa, y recuperación de pseudónimo olvidado.
  - **Pérdida de frase secreta vs. revocación voluntaria.** Evaluar si comparten el mismo mecanismo técnico en el emisor o si requieren flujos diferenciados.

  Consecuencias dependientes de la tupla final (a confirmar en P-0014):
  - Con tupla `{anon_seed, anon_id, fecha_emision}`: el emisor puede devolver la `anon_id` actual al ciudadano que olvidó su pseudónimo, autenticándose nuevamente vía AUTENTICAR. La identidad anónima es recuperable; la frase secreta no lo es.
  - La trazabilidad `anon_seed → anon_id` permite que la plataforma mantenga el historial completo de la identidad: propuestas, apoyos y contadores del límite anual no se pierden ni se resetean con una renovación.
  - A confirmar: si la plataforma puede o no distinguir identidades activas de renovadas, y qué implicaciones tiene eso.
