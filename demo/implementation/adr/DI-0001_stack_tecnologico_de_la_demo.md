# DI-0001 — Stack tecnológico de la demo

**Estado:** Activo

## Contexto

Con el alcance del MVP cerrado (DP-0002), la demo necesita un stack concreto: lenguaje, framework web, base de datos y bibliotecas para las piezas que el diseño ya fija. Varias decisiones de plataforma condicionan la elección:

- **P-0009**: el cliente normaliza la frase secreta y le aplica una función hash estándar antes de enviarla, así que el navegador necesita JavaScript al menos en registro y login. El servidor aplica Argon2id sobre el hash recibido.
- **P-0019**: búsqueda full-text en español con morfología y stopwords.
- **P-0021 y P-0006**: el emisor y la aplicación destino son componentes separados, cada uno con sus propios datos.
- **P-0020**: logs con granularidad temporal por tipo de evento, campos prohibidos y purga gestionada por cada componente.
- **P-0011**: llamada HTTP a una API externa de moderación con reintentos.
- **P-0014**: en producción, el proving ZK usa circom/snarkjs, del ecosistema JavaScript. La demo no lo implementa (DP-0001), pero el emisor real deberá convivir con él.
- **P-0005**: la interfaz nunca sugiere memoria entre sesiones.
- Accesibilidad WCAG nivel A, interfaz monolingüe es-AR (`demo/design/aplicaciones/participacion_ciudadana/i18n_y_a11y.md`).

El código de la demo será escrito y mantenido principalmente por agentes de IA; el conocimiento previo de un lenguaje por parte del equipo humano no es un criterio de elección.

## Opciones consideradas

### Decisión 1 — Lenguaje del servidor

#### Opción A — TypeScript sobre Node.js

Ventajas

- El mismo lenguaje en el navegador y en el servidor: la normalización de la frase secreta (P-0009) se escribe una sola vez y se usa en ambos lados, sin riesgo de divergencia.
- Alineado con circom/snarkjs para el futuro emisor con ZK.
- Ecosistema amplio y bien conocido por los agentes que escriben el código.

Desventajas

- Más dependencias de terceros que en lenguajes con biblioteca estándar más completa.

#### Opción B — Python

Ventajas

- Código muy legible; frameworks maduros (FastAPI, Django).

Desventajas

- La normalización de la frase debe implementarse dos veces (Python en el servidor para tests y validación, JavaScript en el navegador), con riesgo de divergencia.
- Sin afinidad con el ecosistema ZK de P-0014.

#### Opción C — Go

Ventajas

- Un único binario, eficiente y simple de desplegar.

Desventajas

- Menos ergonómico para plantillas HTML; misma duplicación de la normalización que la Opción B; sin afinidad con el ecosistema ZK.

### Decisión 2 — Arquitectura de la interfaz

#### Opción A — HTML renderizado en el servidor con JavaScript mínimo

Las páginas se generan en el servidor. El JavaScript del navegador se limita a lo indispensable: normalización y hash de la frase secreta, diálogos de confirmación del modo debug y del retiro de apoyo, y mejoras puntuales.

Ventajas

- Accesibilidad casi gratuita: las páginas son documentos HTML semánticos.
- Sin estado de la aplicación en el navegador, coherente con P-0005.
- Menos código y menos dependencias.

Desventajas

- Interacciones menos fluidas que en una aplicación de una sola página.

#### Opción B — Aplicación de una sola página (SPA)

Ventajas

- Interfaz más dinámica.

Desventajas

- Más complejidad, más estado en el cliente y más trabajo para alcanzar la accesibilidad requerida.

### Decisión 3 — Base de datos

#### Opción A — PostgreSQL

Ventajas

- Búsqueda full-text en español incluida (configuración `spanish`), suficiente para P-0019 sin motor de búsqueda adicional.
- Disponible en el entorno de despliegue de la demo.

Desventajas

- Requiere un servidor de base de datos.

#### Opción B — SQLite

Ventajas

- Sin servidor; operación más simple.

Desventajas

- Su búsqueda full-text no incluye morfología en español; habría que resolverla aparte.

## Decisiones

**Decisión 1:** TypeScript sobre Node.js (Opción A), en su versión LTS vigente (22).

**Decisión 2:** HTML renderizado en el servidor con JavaScript mínimo (Opción A). Si en el futuro se implementa la plataforma completa, la tecnología de interfaz se reevalúa en un ADR `I-XXXX`.

**Decisión 3:** PostgreSQL (Opción A).

Piezas concretas que completan el stack:

| Pieza | Elección | Motivo |
|---|---|---|
| Framework web | Hono (`@hono/node-server`) con plantillas JSX renderizadas en el servidor | Liviano; plantillas con tipos y escape de HTML por defecto, lo que previene inyección de código en el contenido escrito por usuarios. Alternativa descartada: Fastify con plantillas de texto, sin tipos ni escape por defecto. |
| Acceso a la base | Kysely, con su sistema de migraciones | Constructor de SQL con tipos que permite SQL directo donde hace falta (búsqueda full-text, ranking). Un ORM de alto nivel ocultaría justamente esas consultas. |
| Argon2id | `@node-rs/argon2` | Distribuido precompilado: el build no requiere compilar código nativo. |
| JavaScript del navegador | TypeScript empaquetado con esbuild; hash con Web Crypto | El módulo de normalización de la frase es el mismo archivo en el navegador y en los tests. |
| Tests | Vitest | Estándar actual del ecosistema TypeScript. |

## Justificación

El criterio dominante es la corrección del código con mantenimiento a cargo de agentes: un solo lenguaje en cliente y servidor elimina la duplicación de la normalización de la frase, que es una pieza crítica (si cliente y servidor normalizaran distinto, el login fallaría). Los tipos y el escape por defecto en las plantillas reducen errores y vulnerabilidades sin esfuerzo adicional. HTML renderizado en el servidor cumple accesibilidad y P-0005 por construcción. PostgreSQL resuelve la búsqueda de P-0019 de forma nativa.

## Consecuencias

- El código de la demo vive en `demo/src/` y requiere Node.js 22 para construirse y ejecutarse.
- La separación entre emisor y aplicación destino (uno o dos servicios, una o dos bases) se decide en el ADR de persistencia y despliegue.
- La función hash del cliente y el algoritmo de normalización de la frase deben especificarse como protocolo fijo antes del desarrollo (P-0009 lo exige; hasta ahora solo menciona SHA-256 como ejemplo).
- La adopción de bibliotecas auxiliares sin alternativas relevantes se documenta en `demo/implementation/` sin ADR.

## Referencias

- DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo
- DP-0002 — Alcance funcional del MVP de la demo
- P-0005 — Sesiones temporales
- P-0006 — Modelo de amenazas y confianza
- P-0009 — Hash de la frase en el cliente y Argon2id en el servidor
- P-0014 — Auditoría ZK de legitimidad del emisor
- P-0019 — Búsqueda y filtros
- P-0020 — Política de logs
- P-0021 — Arquitectura modular
