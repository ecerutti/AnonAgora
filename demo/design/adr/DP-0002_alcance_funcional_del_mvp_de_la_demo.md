# DP-0002 — Alcance funcional del MVP de la demo

**Estado:** Activo

## Contexto

La demo tiene como objetivo demostrar el funcionamiento de la plataforma participativa —propuestas, apoyos, identidades anónimas, límites operativos— en un entorno controlado (DP-0001). Antes de iniciar su desarrollo es necesario fijar qué parte del diseño de plataforma implementa la primera versión funcional (MVP).

El inventario de flujos (`notas/inventario_de_flujos_ui_ux.md`) identifica diez flujos con interfaz visible al ciudadano (F-CI-01 y F-AP-00 a F-AP-08), el comportamiento transversal de expiración de sesión y el modificador transversal de modo debug (M-DEBUG). A eso se suma una operación del operador con efecto visible para el ciudadano: el retiro de propuestas por causal legal (P-0023).

Por regla de herencia (`demo/README.md`), la demo adopta todo el diseño de plataforma salvo lo que un ADR `DP-XXXX` registre como simplificación. Cada función que el MVP deje afuera requiere entonces una justificación explícita.

La demo además se publica en un sitio accesible por cualquier persona, y la verificación de identidad es simulada (DP-0001). Esto plantea dos preguntas propias de la demo:

- La verificación simulada debe aportar un CUIT/CUIL, porque el emisor deriva de él el `anon_seed` y sobre el `anon_seed` opera el cool-down (P-0013, P-0015). ¿De dónde sale ese CUIL sin pedir datos reales?
- En el sistema real, la unicidad del CUIL verificado es la base de todos los mecanismos anti-abuso: una persona, una identidad; el cool-down y el cupo anual operan sobre esa unicidad. Con verificación simulada esa base desaparece. ¿Se agrega algún mecanismo anti-abuso propio de la demo?

## Opciones consideradas

### Decisión 1 — Alcance funcional del MVP

#### Opción A — MVP reducido al núcleo

El MVP implementa solo el recorrido central: registro, login, ranking, visualización de propuestas, apoyo y creación. Se omiten o simplifican la búsqueda con filtros estructurados, el modo debug, el retiro por causal legal y el revisor de lenguaje real.

Ventajas

- Menor esfuerzo de desarrollo; la demo estaría disponible antes.
- Menos dependencias externas.

Desventajas

- Cada omisión requiere un DP que la justifique, y varias son difíciles de justificar: la base de logs de P-0020 (granularidad, campos prohibidos, retención) debe implementarse igual, y el modo debug agrega poco sobre ella.
- Omitir el revisor de lenguaje real o el retiro legal elimina justamente las piezas que muestran cómo la plataforma modera la forma sin moderación editorial (P-0011, P-0023).
- La demo deja de mostrar el diseño tal como fue decidido, y comportamientos transversales quedan sin probar hasta un despliegue real.

#### Opción B — MVP con el diseño completo de la plataforma

El MVP implementa todos los flujos del inventario y los comportamientos transversales, con las únicas omisiones de DP-0001 (AUTENTICAR simulado, sin ZK).

Ventajas

- La demo muestra el diseño tal como está decidido, sin DPs de simplificación adicionales.
- Permite probar comportamientos transversales (modo debug, retiro legal, fallos del revisor) antes de cualquier despliegue real.

Desventajas

- Mayor esfuerzo de desarrollo.
- Dependencia externa del revisor de lenguaje (API de moderación y su clave).
- Obliga a resolver antes del desarrollo huecos de UX que el inventario dejaba abiertos (H-1, H-3).

### Decisión 2 — Origen del CUIL en la verificación simulada

#### Opción A — El usuario ingresa un CUIL libre

Ventajas

- Simple y cercano a la experiencia real.

Desventajas

- El usuario puede ingresar su CUIL real, a pesar de la advertencia. Su `anon_seed` quedaría persistido en el emisor de una demo pública.

#### Opción B — Lista fija de personas ficticias

La demo ofrece un conjunto cerrado de personas ficticias con CUIL asignado.

Ventajas

- Simple; imposible ingresar datos reales.

Desventajas

- En un sitio público, cada persona ficticia queda bloqueada por cool-down en cuanto alguien la usa. La lista se agota en poco tiempo.

#### Opción C — Sin CUIL

Cada verificación simulada se trata como una persona nueva.

Ventajas

- La más simple.

Desventajas

- El cool-down no puede demostrarse, y el emisor de la demo deja de reproducir el mecanismo del sistema real.

#### Opción D — CUIL ficticio con prefijo inexistente

La demo genera al azar un CUIL con un prefijo que no se asigna a CUIT/CUIL reales (`99-`) y solo acepta CUILs con ese prefijo. El usuario puede reutilizar un CUIL ficticio ya usado.

Ventajas

- Ingresar un CUIL real es imposible: la validación lo rechaza.
- El cool-down se demuestra reutilizando el mismo CUIL ficticio.
- El espacio de CUILs ficticios es prácticamente ilimitado, así que no se agota.
- El emisor opera exactamente como en el sistema real: deriva `anon_seed` con salt y persiste solo `{anon_seed, fecha_emision}`.

Desventajas

- Crear identidades es gratuito (ver Decisión 3).

### Decisión 3 — Mecanismos anti-abuso propios de la demo

#### Opción A — Reinicio periódico y rate limiting

La demo vuelve periódicamente (por ejemplo, cada noche) a su estado inicial, con aviso visible. Además, aplica rate limiting en memoria, sin persistir ni registrar direcciones IP.

Ventajas

- Acota en el tiempo el efecto del spam y del vandalismo.
- Limita la creación masiva automatizada de identidades.

Desventajas

- Las identidades y propuestas de los usuarios de la demo se pierden en cada reinicio.
- El rate limiting implica tratar direcciones IP, aunque sea en memoria; es un terreno sensible bajo P-0020.
- Más esfuerzo de desarrollo y operación.

#### Opción B — Solo reinicio periódico

Ventajas

- Acota el efecto del abuso sin tratar direcciones IP.

Desventajas

- Las identidades y propuestas se pierden en cada reinicio.
- No limita el abuso dentro de cada período.

#### Opción C — Ningún mecanismo propio de la demo

Ventajas

- Sin esfuerzo adicional; la demo se comporta igual que el sistema real salvo por la verificación simulada.
- Las identidades y propuestas de los usuarios de la demo persisten.

Desventajas

- Cualquiera puede crear identidades ilimitadas, publicar spam (hasta el límite que imponga el revisor de lenguaje) e inflar apoyos.
- El ranking de la demo no es representativo del comportamiento de una población real.

## Decisiones

**Decisión 1:** el MVP implementa el diseño completo de la plataforma (Opción B). En concreto:

- **F-CI-01** con AUTENTICAR simulado y sin ZK (DP-0001), según `demo/design/capa_de_identidad/simulacion_autenticar.md`.
- **F-AP-00 a F-AP-08** completos, incluidos el tutorial y los términos y condiciones de F-AP-01 y la expiración de sesión.
- **Visitantes sin sesión** en modo solo lectura: ranking, búsqueda y visualización de propuestas. Apoyar, crear propuestas y el filtro "apoyadas / no apoyadas por mí" requieren sesión (resolución de H-1 en el inventario de flujos).
- **Búsqueda** full-text en español con los siete filtros estructurados de P-0019.
- **Revisor de lenguaje** con la API de moderación real (P-0011), fail-closed (P-0022). La clave de acceso se provee por configuración del despliegue.
- **Modo debug** completo según P-0020 en ambos componentes, con la resolución de H-3 del inventario de flujos.
- **Retiro por causal legal** (P-0023) como herramienta de línea de comando del operador (P-0024). Los datos iniciales incluyen al menos una propuesta retirada.
- **Parámetros configurables** con los valores por defecto definidos en los ADRs de plataforma: cool-down de 6 meses (P-0015), cupo de 2 propuestas por año móvil (P-0017), expiración de sesión a 1 hora (P-0005), entre otros.

**Decisión 2:** la verificación simulada usa CUILs ficticios con prefijo `99-`, generados al azar por la demo y reutilizables (Opción D).

**Decisión 3:** la demo no agrega mecanismos anti-abuso propios (Opción C).

## Justificación

El valor de la demo está en mostrar el diseño tal como fue decidido. Las piezas que un MVP reducido dejaría afuera son baratas en relación con lo que muestran: el modo debug se apoya en la infraestructura de logs que P-0020 exige de todos modos, y el revisor de lenguaje y el retiro legal son los mecanismos que explican cómo la plataforma funciona sin moderación editorial.

Los valores de producción de los parámetros no traban la demo. Como crear una identidad es gratuito, ningún usuario queda bloqueado por el cool-down o el cupo, y ambos siguen siendo demostrables: reutilizar un CUIL ficticio muestra el rechazo por cool-down, y publicar dos propuestas muestra el agotamiento del cupo.

El prefijo ficticio resuelve a la vez la imposibilidad de ingresar datos reales y la demostración del cool-down, sin agotarse como una lista fija.

La ausencia de mecanismos anti-abuso se acepta porque el objetivo de la demo es mostrar la experiencia, no resistir abuso. La demo usa solo datos ficticios, el revisor de lenguaje filtra los contenidos más graves, y tanto el reinicio periódico como el rate limiting pueden agregarse después sin impacto estructural.

## Consecuencias

- Todo lo que este ADR incluye en el MVP se rige por los ADRs de plataforma correspondientes; la demo no registra decisiones propias sobre esas funciones.
- Quedan por definir, antes o durante el desarrollo: stack tecnológico, persistencia y despliegue, datos iniciales, guía visual, contrato concreto de entrega emisor↔aplicación destino, manejo de la sesión durante redacciones largas, y el texto del tutorial y de los términos y condiciones.
- El cupo y el cool-down funcionan, pero cualquiera puede eludirlos creando nuevas identidades. La demo no demuestra la resistencia del sistema al abuso, que en producción depende de la unicidad del CUIL verificado.
- Si el abuso degrada la demo, la Decisión 3 se revisa mediante un nuevo DP.
- La demo depende de un servicio externo (la API de moderación). Si no está disponible, la creación de propuestas falla de forma controlada según P-0022.

## Referencias

- DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo
- P-0011 — Revisor de lenguaje
- P-0013, P-0015 — Derivación del `anon_seed` y cool-down
- P-0017 — Cupo anual de propuestas
- P-0019 — Búsqueda y filtros
- P-0020 — Política de logs y modo debug
- P-0022 — Comportamiento ante fallos de servicios externos
- P-0023 — Retiro de propuestas por causal legal
- P-0024 — Ausencia de perfil administrativo
- `notas/inventario_de_flujos_ui_ux.md` — flujos de UI/UX y resolución de H-1 y H-3
- `demo/design/capa_de_identidad/simulacion_autenticar.md`
