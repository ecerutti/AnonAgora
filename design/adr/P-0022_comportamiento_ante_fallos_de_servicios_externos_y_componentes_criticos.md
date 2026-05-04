# P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos

**Estado:** Activo

## Contexto

El sistema depende de varios servicios y componentes para cumplir sus funciones. La capa de identidad se integra con AUTENTICAR para verificar identidad real (P-0013) y opera un componente de proving ZK para certificar la legitimidad de cada `anon_id` emitido (P-0014). La aplicación destino verifica las pruebas ZK contra el JWKS histórico mantenido por la capa (P-0014) y, en el caso de la aplicación de participación ciudadana, consulta el revisor de lenguaje de OpenAI antes de publicar cualquier propuesta (P-0011).

Cualquiera de estos servicios o componentes puede estar caído, lento o devolver errores. Sin una política explícita el comportamiento queda librado al criterio de cada implementador, lo que abre tres riesgos: experiencia inconsistente para el ciudadano, contradicción accidental con principios cerrados en otros ADRs, y aparición de modos de operación degradados que el resto del diseño no contempla.

Las preguntas de diseño que motivaron este ADR son:

- ¿Cómo se comporta el sistema cuando un servicio externo o componente crítico no responde, está lento o devuelve error?
- ¿Cuántos reintentos se realizan, con qué backoff, antes de declarar la operación fallida?
- ¿Qué se le comunica al ciudadano y con qué nivel de detalle?

Algunas políticas son derivación directa del diseño y no se discuten como decisión: la verificación de identidad real es prerrequisito de la emisión (P-0013), la prueba ZK es prerrequisito de la entrega de un `anon_id` (P-0014, P-0015), y el revisor de lenguaje es prerrequisito de la publicación de toda propuesta (P-0011). En todos esos casos el flujo es bloqueante por construcción: sin el componente, la operación no procede. Las decisiones que sí tienen alternativas reales son las relativas a reintentos, atomicidad de la emisión y comunicación al ciudadano.

Este ADR es transversal: aplica a la capa de identidad y a la aplicación destino. Se redacta en términos genéricos respecto del flujo concreto de entrega emisor → aplicación destino, que será objeto de un ADR posterior.

## Opciones consideradas

### Decisión 1 — Política ante fallos de la verificación de identidad

Los fallos cubiertos son:

- AUTENTICAR no responde o demora más allá de un umbral en cualquiera de sus endpoints (autorización, token, JWKS).
- AUTENTICAR responde con error en el endpoint de autorización (`server_error`, `temporarily_unavailable`, `access_denied`) o de token (`invalid_grant`, `invalid_client`, equivalentes).
- El JWT recibido por el emisor llega con `exp` ya pasado (token vencido en tránsito).
- El JWKS de AUTENTICAR no está disponible al momento de verificar la firma del JWT.

La degradación es bloqueante por construcción: sin verificación válida no hay emisión. Las decisiones reales son sobre reintentos y comunicación.

#### Opción A — Sin reintento automático

Cualquier fallo se propaga al ciudadano, que reinicia el flujo manualmente.

Ventajas

- Implementación trivial.
- Sin riesgo de amplificar carga sobre AUTENTICAR durante un incidente.

Desventajas

- Convierte cualquier fallo transitorio de red en error visible al ciudadano.
- La fricción al ciudadano es desproporcionada respecto del costo de implementar reintentos acotados.

#### Opción B — Reintento server-to-server con backoff acotado

En las llamadas server-to-server (intercambio de `code` por tokens, descarga del JWKS) el emisor reintenta con backoff exponencial dentro de un timeout total acotado. Tras agotar reintentos, la operación falla y se reporta al ciudadano. En las llamadas que ocurren en el navegador del ciudadano (redirect al endpoint de autorización, callback con error) el reintento corre por cuenta del ciudadano: el sistema solo le presenta el resultado.

El caso del token vencido en tránsito no admite reintento de la misma operación: el JWT ya está vencido y no puede revivirse. El ciudadano debe iniciar un flujo de autenticación nuevo.

Ventajas

- Cubre fallos transitorios de red sin afectar al ciudadano.
- Acotado para no amplificar carga sobre AUTENTICAR.
- Comportamiento consistente con el resto de las llamadas server-to-server del sistema (revisor de lenguaje en Decisión 4, JWKS histórico en este mismo ADR).

Desventajas

- Leve complejidad adicional en la implementación.

Sobre la comunicación al ciudadano:

#### Opción C — Mensaje único genérico

Mismo mensaje ante cualquier fallo de la verificación.

Ventajas

- Implementación más simple.
- Cero riesgo de filtrar información operativa.

Desventajas

- El ciudadano no sabe si conviene reintentar ya, esperar, o hacer algo diferente.
- Confunde casos cuya solución correcta es distinta. Token vencido requiere iniciar flujo nuevo; servicio caído invita a esperar; rechazo del proveedor estatal no se resuelve reintentando.

#### Opción D — Mensaje diferenciado por categoría de fallo

Tres mensajes distintos, sin revelar detalles operativos:

- Servicio externo no disponible o con error transitorio.
- Sesión de verificación expirada.
- Solicitud rechazada por el sistema externo.

Ventajas

- El ciudadano sabe qué hacer.
- No filtra información operativa: los detalles internos del error van solo a logs según P-0020.

Desventajas

- Tres flujos de UX en lugar de uno.

### Decisión 2 — Política ante fallos del componente de proving ZK

Los fallos cubiertos son:

- El componente de proving ZK no responde o supera el timeout de generación.
- El componente devuelve error: input mal formado, constraint no satisfecho, fallo del runtime.

La degradación es bloqueante por construcción: sin prueba ZK válida no se entrega `anon_id`. La decisión real es sobre reintentos.

#### Opción A — Reintento uniforme ante cualquier error

El emisor reintenta N veces ante cualquier error del componente de proving.

Desventajas

- Los errores deterministas (constraint no satisfecho, input mal formado) van a fallar igual en todos los reintentos. El ciudadano espera de más para terminar fallando con la misma causa.
- Gasta recursos del emisor sin beneficio.

#### Opción B — Reintento solo ante fallo transitorio

El emisor distingue tipos de error en la integración con el componente de proving. Errores transitorios (timeout, fallo del runtime, condición de memoria) habilitan reintento. Errores deterministas (constraint no satisfecho, input mal formado) fallan inmediatamente.

Ventajas

- No reintenta lo que sabemos que va a fallar igual.
- Mejor experiencia para el ciudadano y menor carga para el emisor.

Desventajas

- Requiere distinguir tipos de error en la integración con el componente. Ese mapeo es trivial pero hay que hacerlo explícito.

Sobre la comunicación al ciudadano: distinguir tipos de error a nivel de mensaje no aporta nada útil. En ambos casos lo que el ciudadano puede hacer es reintentar y, si persiste, escalar al operador. Mensaje genérico.

### Decisión 3 — Atomicidad de la emisión respecto del cool-down

El emisor genera la identidad anónima en una secuencia: verifica el token de AUTENTICAR, calcula el `anon_seed`, verifica unicidad y cool-down, genera `nonce`, calcula `anon_id`, genera prueba ZK, entrega la tripla a la aplicación destino. P-0015 establece que la tupla `{anon_seed, fecha_emision}` es lo único que el emisor persiste, pero no fija el momento exacto de esa persistencia. Esa precisión queda dentro del alcance de este ADR porque determina qué pasa si la entrega a la aplicación destino falla.

#### Opción A — Emisión atómica: cool-down consumido tras confirmación de entrega

El emisor mantiene la información necesaria para completar la emisión en estado transitorio durante la generación. Solo persiste `{anon_seed, fecha_emision}` tras recibir confirmación de la aplicación destino de que la tripla `(pseudónimo, anon_id, prueba ZK)` fue recibida y aceptada. Si la entrega falla, el cool-down no queda consumido y el ciudadano puede reintentar el flujo.

Ventajas

- El ciudadano no queda colgado del período de cool-down (default 6 meses) por un fallo en la entrega que escapa a su control.
- Coherente con el criterio aplicado en P-0016: cuando un mecanismo puede dejar al ciudadano peor parado por un fallo que no controla, el costo se carga sobre el sistema, no sobre el ciudadano.

Desventajas

- Introduce una superficie temporal en la que coexisten en el emisor el `anon_seed` recién calculado y el `anon_id` derivado, hasta que se confirma la entrega.
- Requiere algún mecanismo de confirmación entre emisor y aplicación destino, cuyo detalle queda diferido al ADR o documento de diseño que defina el flujo de entrega.

#### Opción B — Cool-down consumido al generar, sin esperar confirmación

El emisor persiste `{anon_seed, fecha_emision}` apenas calcula el `anon_seed` y descarta el resto del estado al entregar, sin importar si la aplicación destino confirmó.

Ventajas

- Implementación más simple en el emisor.
- Minimiza o elimina la superficie temporal en la que coexisten en el emisor el anon_seed y el anon_id.

Desventajas

- Si la entrega falla, el ciudadano queda con cool-down consumido pero sin identidad utilizable en la aplicación destino. Tiene que esperar el período completo para reintentar.

### Decisión 4 — Política ante fallos del revisor de lenguaje

Los fallos cubiertos son:

- La API del revisor no responde, supera el timeout, o devuelve error 5xx.
- La API devuelve error 4xx por rate limit, autenticación inválida o request mal formado.

A diferencia de las decisiones anteriores, acá la política de degradación sí tiene alternativas reales.

#### Opción A — Fail-closed: bloquear publicación hasta que el servicio se restablezca

La publicación se rechaza con un mensaje específico al ciudadano. El ciudadano debe reintentar más tarde.

Ventajas

- Respeta P-0011 íntegramente: toda propuesta pasa por el revisor antes de ser publicada.
- La aplicación de participación ciudadana no es de tiempo real. Cupos anuales, cool-downs de meses y un ranking calculado sobre ventanas largas hacen que demorar una publicación minutos u horas no rompa nada.
- No introduce estados nuevos en propuestas ni mecanismos accesorios.

Desventajas

- Una caída prolongada del proveedor del revisor traba publicaciones en toda la aplicación destino (la frecuencia real de caídas prolongadas en proveedores masivos como OpenAI es baja).

#### Opción B — Fail-open: publicar sin filtro durante la caída

La propuesta se publica sin pasar por el revisor.

Desventajas

- Contradice directamente P-0011, decisión cerrada. No hay forma honesta de declarar fail-open sin superseder P-0011.
- Una propuesta con lenguaje ofensivo se publica sin filtro y queda visible en la aplicación destino.

#### Opción C — Fallback local

Se aplica un mecanismo local de filtrado (lista de palabras o modelo local) durante la caída del revisor.

Desventajas

- P-0011 ya descartó explícitamente la lista de palabras (insuficiente frente a ofuscación) y el modelo local (complejidad operativa, capacidad de detección menor). Reintroducirlas como fallback las legitima por la puerta de atrás.
- El ciudadano que se topa con un rechazo del fallback no tiene forma de saber que el comportamiento es distinto al normal.

#### Opción D — Encolado y publicación diferida

La propuesta se acepta, se encola y se publica cuando el revisor vuelve a estar disponible.

Desventajas

- Introduce un estado nuevo de propuesta ("pendiente de revisión") que P-0018 no contempla.
- Rompe el loop de feedback con el ciudadano: la corrección iterativa pre-publicación es parte de la experiencia. Si la propuesta se encola y se rechaza horas después, el ciudadano ya no está delante.
- Requiere diseñar notificación, recuperación del draft, política de retención del encolado. Mucha tela para un caso de fallo.

Sobre reintentos: las llamadas a la API del revisor son server-to-server, igual que las de la Decisión 1. Aplica reintento con backoff acotado antes de declarar la operación fallida.

Sobre la comunicación al ciudadano: el mensaje debe identificar el problema como transitorio y orientar al ciudadano sobre cómo no perder su trabajo. La aplicación destino no garantiza persistencia del draft del ciudadano durante la caída del revisor: la responsabilidad de preservar la propuesta queda del lado del ciudadano, lo que es honesto y evita comprometer al sistema con un mecanismo de persistencia que excede el alcance de este ADR.

## Decisión

**Decisión 1:** Se adoptan las **Opciones B y D**.

Las llamadas server-to-server del emisor a AUTENTICAR (intercambio de `code` por tokens, descarga del JWKS) y las llamadas server-to-server de la aplicación destino al JWKS histórico de la capa al verificar la prueba ZK reintentan con backoff exponencial. Defaults configurables: 3 reintentos, backoff 500 ms / 1 s / 2 s, timeout total 10 s. Las llamadas que ocurren en el navegador del ciudadano no se reintentan automáticamente.

Los mensajes al ciudadano se diferencian en tres categorías:

| Categoría | Casos | Mensaje (ejemplo) |
|---|---|---|
| Servicio externo no disponible o con error transitorio | AUTENTICAR no responde, AUTENTICAR responde `server_error` o `temporarily_unavailable`, fallo en token endpoint, JWKS no disponible, JWKS histórico inaccesible | "El sistema de verificación de identidad no está disponible en este momento. Probá nuevamente en unos minutos." |
| Sesión de verificación expirada | Token vencido en tránsito, AUTENTICAR responde `invalid_grant` | "Tu sesión de verificación expiró antes de completarse. Iniciá el flujo nuevamente." |
| Solicitud rechazada por el sistema externo | AUTENTICAR responde `access_denied` o errores equivalentes de cliente | "El sistema de verificación de identidad rechazó la solicitud." |

La redacción exacta y la realización visual corresponden al diseño de UX, no a este ADR. Los mensajes de la tabla son referencia.

**Decisión 2:** Se adopta la **Opción B**.

El emisor distingue, en la integración con el componente de proving ZK, errores transitorios de errores deterministas. Errores transitorios habilitan reintento. Errores deterministas fallan inmediatamente. Defaults configurables: 2 reintentos con backoff 1 s / 2 s, timeout total 60 s. Los detalles del error van a logs según P-0020.

Mensaje al ciudadano (genérico, ejemplo): "Hubo un problema al completar tu verificación. Probá nuevamente en unos minutos. Si el problema persiste, contactate con el operador del sistema."

**Decisión 3:** Se adopta la **Opción A**.

El emisor consume el cool-down únicamente al recibir confirmación de la aplicación destino de que la entrega de `(pseudónimo, anon_id, prueba ZK)` fue recibida y aceptada. La tupla `{anon_seed, fecha_emision}` se persiste solo en ese momento. Si la entrega falla, el cool-down no se consume y el ciudadano puede reintentar el flujo.

La mecánica concreta de confirmación y la política de reintentos de la entrega quedan diferidas al ADR posterior que defina el flujo emisor↔aplicación destino. Este ADR fija el principio: la emisión es atómica respecto del cool-down y solo se completa con confirmación de entrega exitosa.

Mensaje al ciudadano ante fallo de entrega (genérico, ejemplo): "Hubo un problema al registrar tu identidad anónima. Probá nuevamente en unos minutos."

**Decisión 4:** Se adopta la **Opción A**.

Ante fallo del revisor de lenguaje, la publicación se rechaza con un mensaje específico al ciudadano. La aplicación destino reintenta la llamada al revisor con backoff exponencial antes de declarar la operación fallida. Defaults configurables: 3 reintentos, backoff 500 ms / 1 s / 2 s, timeout total 10 s.

La aplicación destino no persiste el draft de la propuesta durante el fallo. La responsabilidad de preservar el texto queda del lado del ciudadano, indicada explícitamente en el mensaje.

Mensaje al ciudadano (ejemplo): "El servicio que revisa el lenguaje de las propuestas no está disponible en este momento. Para no perder tu propuesta, copiala y guardala por tu cuenta, y volvé a intentar publicarla en unos minutos."

## Justificación

El comportamiento ante fallos no es una decisión de implementación: es una propiedad observable del sistema que afecta directamente la experiencia del ciudadano y la coherencia con principios cerrados. Dejar este comportamiento sin definir invitaba a contradicciones accidentales con P-0011, P-0013, P-0014, P-0015 y al riesgo de que el sistema operara en modos degradados que el resto del diseño no contempla.

La política de reintento server-to-server con backoff acotado es la respuesta estándar a fallos transitorios de red en sistemas distribuidos. Su valor en este contexto es que cubre el caso esperado (latencia o glitch puntual) sin afectar al ciudadano, mientras que ante una caída prolongada falla rápido y le da al ciudadano el feedback que necesita para reintentar más tarde. Los defaults son conservadores: tres reintentos en diez segundos no amplifican carga sobre el servicio caído ni demoran al ciudadano más de lo razonable.

La distinción entre errores transitorios y deterministas en el componente de proving ZK es una afinación pequeña con beneficio claro: evita que el ciudadano espere por reintentos que sabemos que van a fallar igual. La complejidad agregada es mínima.

La atomicidad de la emisión respecto del cool-down es la decisión de mayor peso del ADR. Surge de aplicar el mismo criterio que P-0016 estableció para la invalidación de identidades: cuando un mecanismo puede dejar al ciudadano peor parado por un fallo que no controla, la asimetría de costos es decisiva. El costo de la atomicidad es operativo (un mecanismo de confirmación entre emisor y aplicación destino, una superficie temporal de estado en el emisor); el costo de la alternativa es cargar al ciudadano con un cool-down completo por un fallo de comunicación. El primer costo es resoluble; el segundo es inaceptable.

La elección de fail-closed para el revisor de lenguaje preserva la coherencia con P-0011. Cualquier alternativa requería contradecir un ADR cerrado o introducir complejidad estructural significativa: estados nuevos en propuestas, fallback local que P-0011 ya descartó, encolado con notificación diferida. La asimetría de costos vuelve a ser clara: el costo de fail-closed es que el ciudadano reintente más tarde; el costo de cualquier otra opción es contradicción de principio o complejidad estructural. Si en algún momento aparece un escenario realista donde el revisor cae por períodos prolongados, eso ameritará revisitar P-0011 directamente, no parchearlo desde acá.

La diferenciación de mensajes al ciudadano es coherente con P-0020: los detalles operativos del error quedan en logs (modo debug si hace falta), mientras que el ciudadano recibe la información mínima que le permite saber qué hacer. La decisión de no prometer persistencia del draft durante el fallo del revisor es una aplicación del mismo criterio: el sistema solo declara lo que efectivamente garantiza.

## Consecuencias

- El emisor implementa reintentos server-to-server con backoff exponencial en sus llamadas al token endpoint y al JWKS de AUTENTICAR. Los valores son configurables por el operador, con los defaults definidos en este ADR.
- El emisor distingue tipos de error en la integración con el componente de proving ZK. Errores transitorios habilitan reintento; errores deterministas fallan inmediatamente. Esa distinción se documenta en la integración concreta del componente.
- El emisor no consume el cool-down hasta recibir confirmación de la aplicación destino de que la entrega fue aceptada. La tupla `{anon_seed, fecha_emision}` se persiste solo tras esa confirmación.
- El ADR (o documento de diseño) posterior que defina el flujo de entrega emisor↔aplicación destino debe incluir el mecanismo concreto de confirmación y la política de reintentos de la entrega, respetando el principio de atomicidad establecido acá.
- La aplicación destino implementa reintentos server-to-server con backoff exponencial en sus llamadas al JWKS histórico de la capa y al revisor de lenguaje. Los valores son configurables por el operador.
- Ante fallo del revisor de lenguaje agotados los reintentos, la publicación se rechaza. La aplicación destino no persiste el draft del ciudadano durante el fallo.
- Los mensajes al ciudadano se diferencian en categorías según el tipo de fallo, sin filtrar información operativa que pertenece a logs (P-0020). La redacción exacta y el diseño visual corresponden a la capa de UX.
- La guía de instalación y operación debe documentar los parámetros configurables introducidos por este ADR (cantidades de reintentos, backoffs, timeouts) y los criterios para ajustarlos al contexto de despliegue.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0011 — Revisor automático de lenguaje en propuestas
- P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0016 — Invalidación de identidades anónimas en la plataforma participativa
- P-0020 — Política de logs y retención de metadatos
- P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino
- `design/threat_model.md` — Modelo de amenazas
- `design/capa_de_identidad/README.md` — Contrato capa↔aplicación destino
- `docs/autenticar.md` — Referencia técnica de AUTENTICAR
