# P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas

**Estado:** Activo

## Contexto

El emisor es el componente responsable de verificar que un ciudadano corresponde a una persona real y de emitir la identidad anónima (`anon_id`) que ese ciudadano usará dentro de la plataforma participativa. Las decisiones previas P-0013 y P-0014 definieron cómo se integra el emisor con AUTENTICAR y cómo se audita criptográficamente la legitimidad de las identidades emitidas.

Este ADR cierra las decisiones pendientes sobre tres aspectos interrelacionados: qué datos almacena el emisor y con qué propósito, cómo se deriva el `anon_id` y qué garantías ofrece esa derivación, y cuál es el ciclo de vida de una identidad anónima: emisión, renovación y pérdida.

Durante el diseño surgió una tensión central: para garantizar unicidad y auditoría de legitimidad, el sistema necesita alguna forma de asociar cada `anon_id` con un ciudadano real. Pero cualquier asociación persistente entre identificadores crea riesgo de correlación. Este ADR documenta cómo se resolvió esa tensión.

Las preguntas de diseño que motivaron este ADR son:

- ¿Qué almacena el emisor y qué no?
- ¿Cómo se deriva el `anon_id` de forma que sea auditable pero no reversible ni vinculable al ciudadano real?
- ¿Qué pasa cuando un ciudadano pierde sus credenciales?
- ¿Cuánto tiempo debe transcurrir entre emisiones sucesivas y por qué?

## Opciones consideradas

### Decisión 1 — Período mínimo entre emisiones (cool-down)

El sistema debe garantizar que un ciudadano no pueda solicitar nuevas identidades anónimas de forma indefinida. Sin un control sobre la frecuencia de emisión, un ciudadano podría acumular múltiples identidades activas simultáneas y usarlas para inflar apoyos a propuestas o para superar el límite anual de propuestas por ciudadano. La pregunta es si existe ese control y cómo se configura.

#### Opción A — Sin período mínimo

El ciudadano puede solicitar una nueva identidad en cualquier momento.

Ventajas

- Sin fricción para el ciudadano que perdió sus credenciales.

Desventajas

- Un ciudadano puede solicitar identidades sucesivas de forma indefinida. Las identidades anteriores siguen siendo funcionales en la plataforma, lo que permite acumular identidades activas simultáneas para inflar apoyos o superar el límite anual de propuestas.

#### Opción B — Período mínimo configurable con valor por defecto

Se establece un período mínimo entre emisiones, configurable por el operador, con un valor por defecto de 6 meses contados desde la `fecha_emision` de la identidad activa. No se establece límite a la cantidad de renovaciones de por vida.

Ventajas

- Limita la acumulación de identidades activas simultáneas sin imponer una restricción permanente.
- El operador puede ajustar el valor según el contexto de despliegue.
- Sin límite de por vida, no penaliza al ciudadano que legítimamente renovó muchas veces a lo largo de los años.

Desventajas

- Si el ciudadano pierde sus credenciales, debe esperar el período mínimo para obtener una nueva identidad. Este costo es aceptable porque es precisamente lo que desincentiva el abuso.

### Decisión 2 — Datos almacenados por el emisor

El emisor necesita almacenar información suficiente para verificar unicidad y aplicar el cool-down, pero cualquier dato adicional aumenta la superficie de correlación ante un atacante con acceso al emisor. La pregunta es cuál es el conjunto mínimo.

#### Opción A — Tupla extendida

El emisor almacena `{anon_seed, anon_id, pseudonimo, fecha_emision}` y mantiene la asociación entre `anon_seed` y `anon_id` para permitir operaciones posteriores sobre la identidad.

Ventajas

- Permite al emisor responder consultas sobre identidades emitidas.
- Facilita auditorías retrospectivas en el propio emisor.

Desventajas

- Un atacante con acceso al emisor puede vincular `anon_seed` con `anon_id`, colapsando la separación entre las dos capas de identificadores.
- Contradice el principio de minimización de datos de P-0006.

#### Opción B — Tupla mínima

El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena el `anon_id` ni ninguna asociación entre ambos identificadores.

Ventajas

- Minimiza la información disponible ante un atacante con acceso al emisor.
- Elimina la posibilidad de recuperación de identidad desde el emisor, lo cual refuerza la percepción del ciudadano de que el sistema no puede identificarlo.
- Coherente con el principio de minimización de datos de P-0006.

Desventajas

- El emisor no puede responder consultas sobre identidades emitidas después del momento de emisión.
- Si un ciudadano pierde sus credenciales, la identidad queda irrecuperable incluso con ayuda del emisor.

### Decisión 3 — Derivación del anon_id y auditoría de legitimidad

El `anon_id` es el identificador que el ciudadano usa dentro de la plataforma. La pregunta es cómo derivarlo de forma que sea auditable —demostrable que corresponde a un ciudadano real— sin que esa demostración permita vincular el `anon_id` con el `anon_seed` ni con el CUIT.

Durante el diseño se evaluaron cuatro familias de solución.

#### Opción A — anon_id aleatorio sin auditoría criptográfica

El emisor asigna un pseudónimo aleatorio del espacio disponible según P-0002 y P-0003. No existe mecanismo de auditoría de legitimidad autónomo en la plataforma.

Ventajas

- Implementación simple.

Desventajas

- La plataforma no puede verificar de forma autónoma que un `anon_id` fue emitido legítimamente. Un auditor externo con acceso solo a la plataforma no puede detectar identidades fabricadas.
- Un administrador malicioso con acceso a la plataforma puede insertar `anon_ids` falsos sin que sea detectable.

#### Opción B — anon_id aleatorio con firma independiente del emisor

El emisor asigna un pseudónimo aleatorio y firma cada `anon_id` con una clave privada propia. La plataforma verifica la firma contra la clave pública del emisor. Un auditor con acceso solo a la plataforma puede verificar que cada `anon_id` tiene firma válida del emisor sin cruzar datos con él.

Ventajas

- Auditoría de legitimidad autónoma en la plataforma: cualquier `anon_id` sin firma válida del emisor es detectable.
- Implementación más simple que las opciones criptográficas más avanzadas.

Desventajas

- La raíz de confianza es una clave privada bajo control del operador. En el escenario más probable de despliegue, emisor y plataforma comparten infraestructura y un mismo administrador tiene acceso a ambos. Ese administrador puede usar la clave privada para firmar `anon_ids` fabricados que la plataforma considerará legítimos.
- La firma no resiste el escenario de administrador malicioso con acceso a la infraestructura compartida, que es el escenario operativo habitual del sistema.

#### Opción C — anon_id derivado determinísticamente del CUIT

El `anon_id` interno se calcula como `HASH(salt_del_sistema + CUIT)`, colapsando `anon_seed` y `anon_id` en un único identificador. La prueba ZK certifica directamente ese identificador.

Ventajas

- La prueba ZK cubre directamente el identificador usado en la plataforma.
- Sin asociación separada que almacenar.

Desventajas

- El identificador que vive en la plataforma es función directa del CUIT. Un atacante con acceso a la base de datos de la plataforma, al salt del sistema y a una lista de CUITs puede ejecutar un ataque de diccionario para identificar ciudadanos, dado que el espacio de CUITs es finito y semi-público.
- Incompatible con el principio de minimización de correlaciones de P-0006.

#### Opción D — anon_id derivado del anon_seed y un nonce aleatorio generado por el emisor

El `anon_id` se calcula como `HASH(anon_seed + nonce)`, donde `anon_seed = HASH(salt_del_sistema + CUIT)` según P-0013 y `nonce` es un valor aleatorio generado por el emisor en el momento de la emisión. El nonce es el ingrediente que rompe la relación determinista entre `anon_seed` y `anon_id`, evitando que un atacante con acceso al emisor pueda calcular el `anon_id` de un ciudadano a partir de su `anon_seed`.

El flujo de emisión ocurre en una única interacción del ciudadano con el emisor. El ciudadano presenta su JWT de AUTENTICAR. El emisor calcula el `anon_seed`, verifica unicidad y cool-down; si el `anon_seed` ya existe y no expiró el cool-down, rechaza la solicitud. Si procede, el emisor genera un `nonce` aleatorio, calcula el `anon_id`, genera la prueba ZK, y descarta el `nonce` inmediatamente sin almacenarlo.

La prueba ZK demuestra: "existe un JWT válido de AUTENTICAR cuyo CUIT produce este `anon_seed`, y existe un `nonce` que combinado con ese `anon_seed` produce este `anon_id`", sin revelar ninguno de los dos valores privados. El `anon_id` es el único output público de la prueba.

Ventajas

- La separación entre `anon_seed` y `anon_id` es criptográficamente forzada por el nonce desconocido fuera del momento de emisión. Un atacante con acceso al emisor obtiene `anon_seeds` pero no puede derivar los `anon_ids` correspondientes porque los nonces no se almacenan. Un atacante con acceso a la plataforma obtiene `anon_ids` pero no puede derivar `anon_seeds`.
- La prueba ZK permite a la plataforma verificar de forma autónoma que cualquier `anon_id` tiene un ciudadano real detrás, sin cruzar datos con el emisor.
- El emisor no participa del manejo de la frase secreta del ciudadano. La frase secreta es asunto exclusivo de la plataforma, lo que preserva la separación de funciones.

Desventajas

- El circuito ZK requiere constraints adicionales respecto al diseño de P-0014, al incluir la derivación `anon_seed + nonce → anon_id`.

### Decisión 4 — Alcance de ZK respecto a P-0014

P-0014 adoptó ZK para auditar la legitimidad del `anon_seed`. Con la derivación definida en la Decisión 3 de este ADR, el circuito ZK debe cubrir también el `anon_id`. La pregunta es si ZK debe cubrir ambos o solo el `anon_id`.

#### Opción A — ZK cubre anon_seed y anon_id

El circuito certifica tanto que el `anon_seed` tiene un ciudadano real detrás como que el `anon_id` fue derivado correctamente.

Ventajas

- Auditoría criptográfica completa de ambas capas.
- Previene que un admin malicioso fabrique `anon_seeds` para bloquear ciudadanos durante el cool-down.

Desventajas

- El ataque que ZK sobre el `anon_seed` previene requiere acceso de escritura a la base de datos del emisor. Quien tiene ese acceso dispone de vectores más simples: modificar fechas, borrar filas, o interrumpir el servicio directamente. ZK no protege contra ninguno de esos ataques.
- El `anon_seed` no sale del emisor y no llega a ningún auditor externo. La auditoría criptográfica con valor externo es la del `anon_id`.

#### Opción B — ZK cubre solo el anon_id

El circuito certifica únicamente que el `anon_id` fue generado a partir de un ciudadano real de AUTENTICAR y un nonce generado por el emisor. El `anon_seed` queda protegido por los controles de acceso al emisor.

Ventajas

- Circuito más simple, menor costo de auditoría especializada del circuito.
- La auditoría criptográfica se concentra donde tiene valor externo: el `anon_id` que la plataforma recibe y verifica.

Desventajas

- La legitimidad de los `anon_seeds` no es auditable criptográficamente. Queda cubierta por los controles de acceso al emisor y por la auditoría procedimental del código.

## Decisiones

**Decisión 1:** Se adopta la **Opción B**. El período mínimo entre emisiones es de 6 meses contados desde la `fecha_emision` de la identidad activa. El valor es configurable por el operador; 6 meses es el valor por defecto. No se establece límite a la cantidad de renovaciones de por vida.

**Decisión 2:** Se adopta la **Opción B**. El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena el `anon_id` ni ninguna asociación entre ambos identificadores.

**Decisión 3:** Se adopta la **Opción D**. El `anon_id` se deriva como `HASH(anon_seed + nonce)`, donde `nonce` es un valor aleatorio generado por el emisor para cada emisión. Durante la emisión, el emisor verifica unicidad y cool-down sobre el `anon_seed`; si procede, genera el `nonce`, calcula el `anon_id`, genera la prueba ZK, y descarta el `nonce` inmediatamente. La plataforma recibe del emisor el pseudónimo amigable, el `anon_id` y la prueba ZK. La frase secreta del ciudadano no interviene en este flujo y es asunto exclusivo de la plataforma.

**Decisión 4:** Se adopta la **Opción B**. ZK cubre únicamente el `anon_id`. Este ADR superseda parcialmente P-0014 en lo relativo al alcance del circuito ZK: la certificación del `anon_seed` mediante ZK queda descartada. La infraestructura de JWKS histórico y el mecanismo de revocación por `kid` comprometido definidos en P-0014 se mantienen vigentes, ya que son necesarios para la verificabilidad de las pruebas sobre el `anon_id`.

## Justificación

El cool-down de 6 meses es el mecanismo central de control de abuso. Sin él, un ciudadano podría acumular identidades activas simultáneas aprovechando que las identidades anteriores siguen siendo funcionales en la plataforma una vez emitidas. El valor por defecto de 6 meses es suficientemente largo para desincentivar el abuso y suficientemente corto para no penalizar permanentemente al ciudadano que perdió sus credenciales.

La tupla mínima `{anon_seed, fecha_emision}` en el emisor es consecuencia directa del principio de minimización de datos de P-0006: el emisor almacena estrictamente lo necesario para sus dos funciones operativas. No almacenar la asociación `anon_seed → anon_id` reduce la información disponible para un atacante con acceso al emisor y elimina la posibilidad de recuperación de identidad, lo cual es un beneficio de diseño: la irrecuperabilidad refuerza la percepción del ciudadano de que el sistema no puede identificarlo.

La derivación `anon_id = HASH(anon_seed + nonce)` resuelve la tensión central de este ADR. El nonce es generado por el emisor, consumido para calcular el `anon_id` y la prueba ZK, y descartado inmediatamente sin almacenarse. Con el nonce descartado, la separación entre `anon_seed` y `anon_id` deja de ser organizacional y pasa a ser criptográficamente forzada: aunque un atacante obtenga el `anon_seed` desde el emisor, no puede derivar el `anon_id` correspondiente sin el nonce, que ya no existe en ningún lado.

Se consideró usar `HASH(frase_secreta)` del ciudadano como segundo componente en lugar de un nonce aleatorio. Esa alternativa quedó descartada porque involucraría al emisor en el manejo de la frase secreta, que es credencial de acceso a la plataforma y no pertenece al alcance de responsabilidades del emisor. La frase secreta es asunto exclusivo de la plataforma; el emisor no necesita conocerla, ni siquiera como hash, para cumplir su función. Un nonce aleatorio generado por el emisor cumple el mismo rol criptográfico en la derivación del `anon_id` sin violar la separación de funciones.

La firma independiente del emisor sobre el `anon_id` fue evaluada como mecanismo de auditoría más simple que ZK. Se descartó porque en el escenario de despliegue habitual del sistema, emisor y plataforma comparten infraestructura y administrador. En ese contexto, la clave privada del emisor es accesible para el mismo actor que podría fabricar identidades falsas, lo que hace que la firma no agregue protección real frente a la amenaza que intenta mitigar.

ZK se concentra en el `anon_id` porque es el único identificador que sale del emisor y llega a la plataforma. La legitimidad del `anon_seed` no requiere prueba criptográfica porque el `anon_seed` nunca sale del emisor; queda cubierta por los controles de acceso y la auditoría procedimental del código del emisor.

## Consecuencias

- El emisor almacena `{anon_seed, fecha_emision}` por cada ciudadano registrado. No almacena `anon_id`, pseudónimo amigable, nonce, frase secreta ni ningún derivado de ella.
- El flujo de emisión ocurre en una única interacción. El emisor verifica el JWT de AUTENTICAR, calcula el `anon_seed`, verifica unicidad y cool-down, y si procede genera un `nonce` aleatorio, calcula el `anon_id` y genera la prueba ZK. El `nonce` se descarta inmediatamente; no debe almacenarse ni loguearse en ningún punto.
- La plataforma recibe del emisor en el momento de la emisión: el pseudónimo amigable, el `anon_id` y la prueba ZK. A partir de ese momento opera de forma completamente independiente del emisor.
- La frase secreta del ciudadano no interviene en el flujo de emisión. Su definición, almacenamiento y verificación son responsabilidad exclusiva de la plataforma, según P-0008 y P-0009.
- El login del ciudadano en la plataforma se realiza con pseudónimo amigable y frase secreta según P-0004, contra el material que la plataforma haya almacenado al momento del registro del ciudadano en la plataforma.
- La pérdida del pseudónimo amigable, de la frase secreta, o de ambos, hace la identidad irrecuperable. El ciudadano debe esperar el cool-down para solicitar una nueva identidad completa.
- El historial de una identidad inactiva —propuestas publicadas, apoyos dados— permanece visible en la plataforma y sigue contando. El sistema no puede distinguir una identidad abandonada de una simplemente inactiva. La decisión sobre si la plataforma puede marcar `anon_ids` como inactivos queda fuera del alcance de este ADR y corresponde al modelo de datos de la plataforma participativa.
- El robo de credenciales no tiene mitigación técnica en este nivel. Su impacto es acotado: el ladrón opera dentro de los mismos límites que el ciudadano legítimo.
- El circuito ZK de P-0014 debe adaptarse para certificar la relación `anon_seed + nonce → anon_id`, con el `anon_id` como output público y el `anon_seed` y el `nonce` como witnesses privados. El constraint de certificación del `anon_seed` definido en P-0014 queda descartado.
- La infraestructura de JWKS histórico y el mecanismo de revocación por `kid` comprometido definidos en P-0014 se mantienen vigentes.

## Limitaciones conocidas del principio de identidad única

El principio declarado del sistema es "un ciudadano real, una identidad anónima dentro de la plataforma". Las decisiones adoptadas en este ADR y en P-0016 respetan ese principio en el caso esperado: el ciudadano mantiene una única identidad anónima activa y, si la pierde, debe esperar el cool-down antes de obtener una nueva.

Sin embargo, el sistema no puede distinguir entre un ciudadano que realmente perdió su identidad y uno que simula haberla perdido para obtener otra adicional. Esta limitación es una consecuencia directa de dos decisiones del diseño:

- El emisor no almacena ningún vínculo entre el ciudadano y identidad anónima.
- La plataforma no implementa mecanismos de invalidación de identidades anónimas (ver P-0016).

En consecuencia, un ciudadano que conserva acceso a su identidad anónima original y, cumplido el cool-down, solicita una nueva alegando pérdida, puede operar con dos identidades activas de forma simultánea. Esto le permitiría:

- Duplicar su cupo anual de propuestas (ver P-0017).
- Apoyar la misma propuesta desde ambas identidades, inflando el conteo de apoyos.
- Participar con dos identidades en propuestas vinculadas de forma que parezca apoyo independiente.

Esta limitación es aceptada dentro del modelo de amenazas intermedio de P-0006. El cool-down (default 6 meses) impone un costo temporal significativo que reduce pero no elimina el incentivo de este comportamiento. El diseño del sistema prioriza la no construcción de mecanismos de invalidación (porque introducirían vectores de ataque de denegación de servicio, ver P-0016) por sobre la eliminación completa de esta posibilidad de abuso.

La limitación debe estar documentada en cualquier material orientado a operadores o asesores técnicos para que el alcance real del principio sea correctamente entendido.

## Referencias

- P-0002 — Representación de identidades anónimas mediante pseudónimos amigables
- P-0003 — Selección del pseudónimo de identidad anónima
- P-0004 — Autenticación de identidad anónima
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0008 — Mecanismo de credencial de acceso: passphrase vs password
- P-0009 — Algoritmo de almacenamiento de la frase secreta
- P-0013 — Integración con AUTENTICAR
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero (parcialmente supersedado por este ADR en lo relativo al alcance del circuito ZK)
- `design/identity_model.md` — Modelo de identidad
- `design/threat_model.md` — Modelo de amenazas
