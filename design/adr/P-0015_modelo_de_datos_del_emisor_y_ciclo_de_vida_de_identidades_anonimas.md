# P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas

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

- Un ciudadano que pierde sus credenciales debe esperar el cool-down aunque no haya tenido intención de acumular identidades.

### Decisión 2 — Qué almacena el emisor

El emisor necesita cumplir exactamente dos funciones después de la emisión: detectar si un ciudadano ya tiene una identidad activa, y verificar si expiró el período de cool-down definido en la Decisión 1. La pregunta es qué datos mínimos son necesarios para eso.

#### Opción A — Almacenar la asociación anon_seed → anon_id

El emisor almacena `{anon_seed, anon_id, fecha_emision}`, manteniendo la trazabilidad completa entre el identificador derivado del CUIT y la identidad anónima emitida.

Ventajas

- Permite recuperar el `anon_id` de un ciudadano que olvidó su pseudónimo, re-autenticándose vía AUTENTICAR.

Desventajas

- La asociación `anon_seed → anon_id` almacenada es un vínculo directo entre el identificador derivado del CUIT y la identidad usada en la plataforma. Un atacante con acceso al emisor tiene acceso a esa tabla completa.
- La recuperación de identidad ante pérdida de credenciales transmite al ciudadano la percepción de que el sistema sabe quién es, erosionando la confianza en el anonimato aunque técnicamente no lo comprometa más que la emisión original.

#### Opción B — Almacenar solo anon_seed y fecha_emision

El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena el `anon_id` ni ninguna asociación entre ambos identificadores.

Ventajas

- Minimización de datos: el emisor almacena estrictamente lo necesario para sus dos funciones operativas.
- Un atacante con acceso al emisor obtiene `anon_seeds` y fechas, pero no puede determinar qué `anon_id` corresponde a cada uno sin la frase secreta del ciudadano.
- La irrecuperabilidad de credenciales es coherente con la percepción del ciudadano de que el sistema no sabe quién es.

Desventajas

- Si el ciudadano pierde su pseudónimo o su frase secreta, la identidad es irrecuperable. Debe esperar el cool-down para solicitar una nueva.

### Decisión 3 — Derivación del anon_id y auditoría de legitimidad

El `anon_id` es el identificador que el ciudadano usa dentro de la plataforma. La pregunta es cómo derivarlo de forma que sea auditable —demostrable que corresponde a un ciudadano real— sin que esa demostración permita vincular el `anon_id` con el `anon_seed` ni con el CUIT.

Durante el diseño se evaluaron tres familias de solución.

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

El `anon_id` interno se calcula como `HASH(salt + CUIT)`, colapsando `anon_seed` y `anon_id` en un único identificador. La prueba ZK certifica directamente ese identificador.

Ventajas

- La prueba ZK cubre directamente el identificador usado en la plataforma.
- Sin asociación separada que almacenar.

Desventajas

- El identificador que vive en la plataforma es función directa del CUIT. Un atacante con acceso a la base de datos de la plataforma, al salt del sistema y a una lista de CUITs puede ejecutar un ataque de diccionario para identificar ciudadanos, dado que el espacio de CUITs es finito y semi-público.
- Incompatible con el principio de minimización de correlaciones de P-0006.

#### Opción D — anon_id derivado del anon_seed y la frase secreta del ciudadano

El `anon_id` se calcula como `HASH(anon_seed + HASH(frase_secreta))`, donde `anon_seed = HASH(salt + CUIT)` según P-0013. La frase secreta actúa como ingrediente que el emisor nunca almacena.

Durante la emisión el flujo ocurre en dos fases secuenciales. En la primera, el emisor calcula el `anon_seed` a partir del token de AUTENTICAR, verifica unicidad y cool-down, y decide si procede. Si el `anon_seed` ya existe y no expiró el cool-down, rechaza la solicitud sin procesar ningún dato adicional. En la segunda fase, el ciudadano envía `HASH(frase_secreta)` al emisor. El emisor lo usa para calcular el `anon_id = HASH(anon_seed + HASH(frase_secreta))` y para derivar el verificador de frase secreta, que es un valor separado calculado a partir del `HASH(frase_secreta)` y que la plataforma usará en el login para verificar que el ciudadano conoce la frase sin que la plataforma necesite el `anon_seed`. El `HASH(frase_secreta)` se descarta inmediatamente una vez calculados el `anon_id` y el verificador.

La prueba ZK demuestra: "existe un JWT válido de AUTENTICAR cuyo CUIT produce este `anon_seed`, y existe un `HASH(frase_secreta)` que combinado con ese `anon_seed` produce este `anon_id`", sin revelar ninguno de los dos valores privados. El `anon_id` es el único output público de la prueba.

Ventajas

- La separación entre `anon_seed` y `anon_id` es criptográficamente forzada por el secreto del ciudadano, no solo organizacional. Un atacante con acceso al emisor obtiene `anon_seeds` pero no puede derivar `anon_ids` sin las frases secretas. Un atacante con acceso a la plataforma obtiene `anon_ids` pero no puede derivar `anon_seeds`.
- La prueba ZK permite a la plataforma verificar de forma autónoma que cualquier `anon_id` tiene un ciudadano real detrás, sin cruzar datos con el emisor.
- La irrecuperabilidad ante pérdida de credenciales refuerza la percepción del ciudadano de que el sistema no puede identificarlo, coherente con los principios declarados del sistema.

Desventajas

- El circuito ZK requiere constraints adicionales respecto al diseño de P-0014, al incluir la derivación `anon_seed + HASH(frase_secreta) → anon_id`.
- Si el ciudadano pierde su pseudónimo o su frase secreta, la identidad es irrecuperable.

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

El circuito certifica únicamente que el `anon_id` fue generado a partir de un ciudadano real de AUTENTICAR y una frase secreta conocida por ese ciudadano. El `anon_seed` queda protegido por los controles de acceso al emisor.

Ventajas

- Circuito más simple, menor costo de auditoría especializada del circuito.
- La auditoría criptográfica se concentra donde tiene valor externo: el `anon_id` que la plataforma recibe y verifica.

Desventajas

- La legitimidad de los `anon_seeds` no es auditable criptográficamente. Queda cubierta por los controles de acceso al emisor y por la auditoría procedimental del código.

## Decisiones

**Decisión 1:** Se adopta la **Opción B**. El período mínimo entre emisiones es de 6 meses contados desde la `fecha_emision` de la identidad activa. El valor es configurable por el operador; 6 meses es el valor por defecto. No se establece límite a la cantidad de renovaciones de por vida.

**Decisión 2:** Se adopta la **Opción B**. El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena el `anon_id` ni ninguna asociación entre ambos identificadores.

**Decisión 3:** Se adopta la **Opción D**. El `anon_id` se deriva como `HASH(anon_seed + HASH(frase_secreta))`. Durante la emisión el ciudadano envía `HASH(frase_secreta)` al emisor en la segunda fase del flujo, una vez que el emisor verificó unicidad y cool-down. El emisor calcula el `anon_id`, genera la prueba ZK y deriva el verificador de frase secreta, descartando el `HASH(frase_secreta)` inmediatamente. La plataforma recibe el pseudónimo amigable, el `anon_id`, la prueba ZK y el verificador de frase secreta.

**Decisión 4:** Se adopta la **Opción B**. ZK cubre únicamente el `anon_id`. Este ADR superseda parcialmente P-0014 en lo relativo al alcance del circuito ZK: la certificación del `anon_seed` mediante ZK queda descartada. La infraestructura de JWKS histórico y el mecanismo de revocación por `kid` comprometido definidos en P-0014 se mantienen vigentes, ya que son necesarios para la verificabilidad de las pruebas sobre el `anon_id`.

## Justificación

El cool-down de 6 meses es el mecanismo central de control de abuso. Sin él, un ciudadano podría acumular identidades activas simultáneas aprovechando que las identidades anteriores siguen siendo funcionales en la plataforma una vez emitidas. El valor por defecto de 6 meses es suficientemente largo para desincentivar el abuso y suficientemente corto para no penalizar permanentemente al ciudadano que perdió sus credenciales.

La tupla mínima `{anon_seed, fecha_emision}` en el emisor es consecuencia directa del principio de minimización de datos de P-0006: el emisor almacena estrictamente lo necesario para sus dos funciones operativas. No almacenar la asociación `anon_seed → anon_id` reduce la información disponible para un atacante con acceso al emisor y elimina la posibilidad de recuperación de identidad, lo cual es un beneficio de diseño: la irrecuperabilidad refuerza la percepción del ciudadano de que el sistema no puede identificarlo.

La derivación `anon_id = HASH(anon_seed + HASH(frase_secreta))` resuelve la tensión central de este ADR. Al incorporar la frase secreta como ingrediente que el emisor nunca almacena, la separación entre `anon_seed` y `anon_id` pasa de ser organizacional a ser criptográficamente forzada. La estructura en dos fases del flujo de emisión garantiza que el `HASH(frase_secreta)` solo se procesa si la emisión va a proceder, minimizando la exposición.

La firma independiente del emisor sobre el `anon_id` fue evaluada como mecanismo de auditoría más simple que ZK. Se descartó porque en el escenario de despliegue habitual del sistema, emisor y plataforma comparten infraestructura y administrador. En ese contexto, la clave privada del emisor es accesible para el mismo actor que podría fabricar identidades falsas, lo que hace que la firma no agregue protección real frente a la amenaza que intenta mitigar.

ZK se concentra en el `anon_id` porque es el único identificador que sale del emisor y llega a la plataforma. La legitimidad del `anon_seed` no requiere prueba criptográfica porque el `anon_seed` nunca sale del emisor; queda cubierta por los controles de acceso y la auditoría procedimental del código del emisor.

## Consecuencias

- El emisor almacena `{anon_seed, fecha_emision}` por cada ciudadano registrado. No almacena `anon_id`, pseudónimo amigable, frase secreta ni ningún derivado de ella.
- El flujo de emisión ocurre en dos fases. En la primera el emisor calcula el `anon_seed`, verifica unicidad y cool-down, y decide si procede. En la segunda el ciudadano envía `HASH(frase_secreta)`, el emisor calcula el `anon_id`, genera la prueba ZK y deriva el verificador de frase secreta. El `HASH(frase_secreta)` no debe almacenarse ni loguearse en ningún punto.
- La plataforma recibe del emisor en el momento de la emisión: el pseudónimo amigable, el `anon_id`, la prueba ZK y el verificador de frase secreta. A partir de ese momento opera de forma completamente independiente del emisor.
- El login del ciudadano en la plataforma se realiza con pseudónimo amigable y frase secreta según P-0004. La plataforma verifica la frase secreta contra el verificador almacenado sin necesitar el `anon_seed` ni contactar al emisor.
- La pérdida del pseudónimo amigable, de la frase secreta, o de ambos, hace la identidad irrecuperable. El ciudadano debe esperar el cool-down para solicitar una nueva identidad completa.
- El historial de una identidad inactiva —propuestas publicadas, apoyos dados— permanece visible en la plataforma y sigue contando. El sistema no puede distinguir una identidad abandonada de una simplemente inactiva. La decisión sobre si la plataforma puede marcar `anon_ids` como inactivos queda fuera del alcance de este ADR y corresponde al modelo de datos de la plataforma participativa.
- El robo de credenciales no tiene mitigación técnica en este nivel. Su impacto es acotado: el ladrón opera dentro de los mismos límites que el ciudadano legítimo.
- El circuito ZK de P-0014 debe adaptarse para certificar la relación `anon_seed + HASH(frase_secreta) → anon_id`, con el `anon_id` como output público y el `anon_seed` y el `HASH(frase_secreta)` como witnesses privados. El constraint de certificación del `anon_seed` definido en P-0014 queda descartado.
- La infraestructura de JWKS histórico y el mecanismo de revocación por `kid` comprometido definidos en P-0014 se mantienen vigentes.
- P-0009 asumió un flujo donde la plataforma recibe la frase secreta en texto plano para almacenarla. Con la Decisión 3 de este ADR ese flujo cambia: la plataforma recibe el verificador ya calculado por el emisor y nunca ve la frase en texto plano. La corrección formal de P-0009 queda pendiente en un ADR posterior.

## Referencias

- P-0002 — Representación de identidades anónimas mediante pseudónimos amigables
- P-0003 — Selección del pseudónimo de identidad anónima
- P-0004 — Autenticación de identidad anónima
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0009 — Algoritmo de almacenamiento de la frase secreta (pendiente de corrección formal en ADR posterior)
- P-0013 — Integración con AUTENTICAR
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero (parcialmente supersedado por este ADR en lo relativo al alcance del circuito ZK)
- `design/identity_model.md` — Modelo de identidad
- `design/threat_model.md` — Modelo de amenazas
