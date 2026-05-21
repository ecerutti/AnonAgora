# P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero

**Estado:** Parcialmente supersedido por P-0015

## Contexto

El emisor es el componente de la capa de identidad responsable de verificar que un ciudadano corresponde a una persona real —mediante un token JWT firmado por AUTENTICAR— y de emitir la identidad anónima (`anon_id`) correspondiente. Una vez completado ese proceso, el token se descarta sin retener ningún metadato, por decisión de P-0013.

Esta ausencia de retención introduce una limitación conocida: no es posible demostrar retrospectivamente que un `anon_id` específico fue generado a partir de un token real de AUTENTICAR. Un administrador malicioso con acceso al emisor podría fabricar identidades anónimas sin que ningún ciudadano real las respalde, y esa manipulación sería indetectable mediante auditoría forense sobre los datos almacenados. La única defensa disponible bajo P-0013 Decisión 4 es procedimental: revisión del código fuente y del diseño del sistema.

La Decisión 4 de P-0013 adoptó esa postura de forma explícitamente temporal, dejando pendiente la evaluación de pruebas de conocimiento cero (ZK) como mecanismo de auditoría criptográfica fuerte. Este ADR cierra esa evaluación y formaliza las decisiones resultantes.

Las preguntas de diseño que motivaron este ADR son:

- ¿Se adopta ZK como mecanismo de auditoría de legitimidad del emisor, reemplazando la auditoría procedimental adoptada temporalmente en P-0013?
- Si se adopta, ¿qué circuito, stack tecnológico y arquitectura de generación de pruebas se utiliza?

La investigación técnica de referencia que sustenta este ADR está documentada en `docs/zk_jwt_investigacion.md`.

## Opciones consideradas

### Decisión 1 — Adopción o descarte de ZK como mecanismo de auditoría

#### Opción A — Mantener auditoría procedimental (descartar ZK)

Confirmar como permanente la Decisión 4 de P-0013: el token se descarta sin generar ninguna prueba, y la auditoría del emisor queda limitada a revisión de código y diseño.

Ventajas

- Sin complejidad adicional en el emisor.
- Sin requisitos de trusted setup ni infraestructura de JWKS histórico.
- Implementación completamente accesible sin expertise en criptografía ZK.

Desventajas

- Un administrador malicioso con acceso al emisor puede fabricar identidades anónimas sin respaldo en ciudadanos reales, sin que sea detectable retrospectivamente.
- La garantía de legitimidad depende exclusivamente de la confianza en los operadores y en la auditabilidad del código fuente.
- No es coherente con el principio de integridad verificable del sistema, que busca que la corrección no dependa de confianza ciega en los operadores.

#### Opción B — Adoptar ZK con generación de prueba en el servidor (emisor)

Antes de descartar el token, el emisor genera una prueba ZK que demuestra: *"existe un token JWT válido firmado por AUTENTICAR cuyo CUIT produce este `anon_seed`, y existe un nonce que combinado con ese `anon_seed` produce este `anon_id`"*, sin revelar el CUIT, el token ni el nonce. El `anon_id` es el único output público relacionado con la identidad del ciudadano.

Ventajas

- Auditoría criptográfica verificable matemáticamente por cualquier tercero con información pública.
- Un admin malicioso no puede fabricar identidades sin un token real de AUTENTICAR, porque no puede producir pruebas válidas sin él.
- Coherente con el principio de integridad verificable.
- La prueba cubre la cadena completa hasta el `anon_id`, que es el identificador que la aplicación destino recibe y verifica.

Desventajas

- Introduce dependencia en infraestructura de trusted setup y JWKS histórico.
- Requiere auditoría especializada del circuito adaptado antes del despliegue en producción.
- Agrega complejidad al emisor.

#### Opción C — Adoptar ZK con generación de prueba en el cliente

El ciudadano genera la prueba ZK localmente en su dispositivo antes de enviarla al emisor. El CUIT nunca sale del dispositivo del ciudadano.

Ventajas

- El CUIT nunca es visto por el emisor, ni siquiera durante la generación de la prueba. Elimina completamente la ventana de exposición.

Desventajas

- Tiempos de generación estimados de 60-180 segundos en smartphones de gama media-baja, inaceptables para la experiencia del usuario.
- El archivo `.zkey` (proving key) pesa cientos de MB y debe descargarse al dispositivo antes de poder generar la prueba.
- Probable fallo por memoria insuficiente en dispositivos con menos de 3-4 GB de RAM libre, segmento de hardware habitual entre usuarios de la aplicación destino.

### Decisión 2 — Stack tecnológico y circuito base

Esta decisión solo aplica si la Decisión 1 adopta ZK.

#### Opción A — circom + snarkjs con circuito base de zk-email-verify

Usar el circuito RSA de `zkemail/zk-email-verify` (MIT, auditado por zkSecurity y yAcademy en mayo 2024) como núcleo de verificación de firma RS256, adaptado para extraer el claim `cuit`, calcular el `anon_seed` internamente, y derivar el `anon_id` a partir del `anon_seed` y un nonce aleatorio generado por el emisor. El proving y la verificación se realizan con snarkjs (GPL-3, v0.7.6) en Node.js en el servidor.

El esquema de prueba es Groth16 sobre la curva BN254. El circuito RSA-2048 tiene aproximadamente 1,5 millones de constraints, con tiempos de proving de 15-45 segundos en servidor estándar de 8 cores. Las pruebas Groth16 tienen un tamaño de ~256 bytes y se verifican en 1-5 ms en servidor.

Ventajas

- El núcleo RSA está auditado. La parte no auditada se limita a la adaptación de extracción de claim, cálculo del `anon_seed` y derivación del `anon_id`.
- La misma pila es la que usa zkLogin de Sui en producción desde 2023.
- Compatible con RSA-2048, que es el tamaño de clave confirmado en el JWKS de los reinos `afip` y `anses` de AUTENTICAR.
- Pruebas compactas (~256 bytes) y verificación rápida (1-5 ms en servidor).

Desventajas

- La adaptación del circuito no está auditada y debe serlo antes de cualquier despliegue en producción. El costo estimado de mercado para una auditoría especializada en ZK es de USD 30.000–150.000 según el tamaño del circuito y la firma auditora.
- `zkemail/zk-jwt`, que preempaqueta una adaptación similar sobre el mismo circuito base, se autodeclara no apto para producción y no está auditado; no puede usarse directamente.
- Requiere expertise en circom para realizar y validar la adaptación.
- Requiere una ceremonia de trusted setup Phase 2 específica para el circuito adaptado.

#### Opción B — zkVM (RISC Zero o SP1)

Implementar la verificación del JWT RS256 como código Rust ejecutado en una zkVM. RISC Zero (v3.0.5, Apache/MIT) y SP1 (v6.1.0, MIT) están en producción con auditorías publicadas. El "circuito" es código Rust estándar que verifica el JWT, extrae el CUIT, calcula el `anon_seed` y deriva el `anon_id`.

Ventajas

- No requiere implementar circuitos en lenguaje de constraints: el código es Rust estándar.
- Menor expertise en criptografía ZK requerido para la implementación.
- Ambas plataformas gestionan el trusted setup de su Groth16 recursivo interno.

Desventajas

- Tiempos de proving mayores: 10-60 segundos en modo cloud (Bonsai de RISC Zero), con latencia variable e introducción de dependencia en infraestructura externa.
- Menor control sobre el circuito resultante y su auditabilidad independiente.
- Stack menos establecido para este caso de uso específico que circom/snarkjs.

## Decisiones

**Decisión 1:** Se adopta ZK como mecanismo de auditoría criptográfica de legitimidad del emisor (Opción B). La auditoría procedimental adoptada temporalmente en P-0013 Decisión 4 queda reemplazada por este ADR. P-0013 Decisión 4 queda supersedada.

La generación de la prueba se realiza en el servidor (emisor). La Opción C (generación en cliente) queda descartada por incompatibilidad con el perfil de dispositivos de los usuarios.

**Decisión 2:** Se adopta la pila circom + snarkjs con el circuito RSA de `zkemail/zk-email-verify` como base (Opción A).

La adaptación específica al caso del sistema consiste en:

- Configurar la extracción del claim `cuit` como input privado del circuito.
- Calcular `anon_seed = HASH(salt_del_sistema + cuit)` como valor intermedio privado, usando separación de dominios en el hash Poseidon para evitar colisiones cruzadas con otros usos del mismo primitivo.
- Calcular `anon_id = HASH(anon_seed + nonce)` como output público, donde `nonce` es un input privado aleatorio generado por el emisor para cada emisión, según P-0015.
- Exponer el `publicKeyHash` (hash Poseidon de la clave pública RSA) y el `kid` como outputs públicos para vincular la prueba a la JWK correspondiente del JWKS de AUTENTICAR.

Los witnesses privados del circuito son el CUIT, el `anon_seed` (como valor intermedio) y el `nonce`. El output público relacionado con la identidad es únicamente el `anon_id`. El circuito núcleo RSA de `zk-email-verify` no debe modificarse. Las únicas modificaciones permitidas son las que configuran los parámetros de extracción de claim y la derivación del `anon_id`, sin alterar la lógica de verificación de firma.

## Justificación

La adopción de ZK es coherente con el principio de integridad verificable del sistema definido en `design/capa_de_identidad/identity_model.md`: el sistema debe poder demostrar su correcto funcionamiento sin depender de confianza ciega en los operadores. La auditoría procedimental cumple ese objetivo de forma débil —depende de que alguien revise el código y confíe en que lo que ejecuta en producción coincide con lo auditado. ZK lo cumple de forma fuerte: la prueba es verificable matemáticamente por cualquier tercero con información pública, sin cruzar datos con ningún componente del sistema.

El escenario de administrador malicioso con acceso al emisor es plausible en el contexto operativo habitual del sistema, donde emisor y aplicación destino pueden compartir infraestructura física. ZK elimina la capacidad de ese actor de fabricar identidades anónimas sin ciudadanos reales detrás, que es la amenaza específica que P-0013 Decisión 4 dejaba sin cobertura.

La prueba cubre la cadena completa hasta el `anon_id` porque ese es el identificador que llega a la aplicación destino y que un auditor externo puede verificar. Una prueba que certificara solo el `anon_seed` no sería útil para la aplicación destino: el `anon_seed` no sale del emisor, y la aplicación destino no puede verificar algo que no ve. Al extender el circuito hasta la derivación del `anon_id` según P-0015, la aplicación destino y cualquier auditor externo pueden verificar de forma autónoma que cada `anon_id` que aparece en el sistema fue generado a partir de un token real de AUTENTICAR.

La generación en servidor se elige sobre la generación en cliente porque los tiempos de proving en cliente (60-180 segundos estimados en dispositivos de gama media-baja) son inaceptables para la experiencia del usuario, y los requisitos de RAM del archivo `.zkey` son incompatibles con smartphones del segmento objetivo.

La pila circom + snarkjs sobre `zk-email-verify` se elige sobre zkVMs porque el núcleo RSA ya está auditado, el stack es el más establecido para este caso de uso (zkLogin de Sui lo usa en producción), las pruebas Groth16 son las más compactas (~256 bytes) y la verificación es la más rápida (1-5 ms). Las zkVMs ofrecen menor barrera de entrada en implementación pero mayor dependencia en infraestructura externa y menor control sobre la auditabilidad del circuito resultante.

## Consecuencias

- El emisor debe implementar un componente de proving ZK que, antes de descartar el token de AUTENTICAR y el `nonce`, genere una prueba Groth16 usando el circuito adaptado de `zk-email-verify`. La prueba se entrega a la aplicación destino junto con el `anon_id` y el pseudónimo amigable, según P-0015.
- La prueba tiene un tamaño de ~256 bytes y no contiene datos correlacionables con la identidad real del ciudadano.
- La prueba ZK certifica la legitimidad del `anon_id` completo: existe un token JWT válido de AUTENTICAR cuyo CUIT deriva en un `anon_seed` que, combinado con el nonce generado por el emisor, produce ese `anon_id`. La aplicación destino y cualquier auditor externo pueden verificar autónomamente esta cadena sin cruzar datos con el emisor.
- El emisor debe implementar un servicio de JWKS histórico que registre todas las JWKs que AUTENTICAR haya publicado, indexadas por `kid`. Este componente es necesario para que las pruebas generadas bajo claves rotadas sigan siendo verificables. Referencia de implementación: `historical-jwks-zklogin` de MystenLabs.
- El emisor debe implementar un mecanismo de revocación por `kid` comprometido: si AUTENTICAR reporta una clave privada comprometida, el sistema debe poder marcar ese `kid` como inválido y gestionar las identidades emitidas bajo él. El `kid` es output público de cada prueba, lo que permite esta operación sin revelar datos privados.
- La adaptación del circuito requiere auditoría de seguridad especializada en ZK antes de cualquier despliegue en producción. El costo estimado es USD 30.000–150.000 según tamaño del circuito y firma auditora.
- Antes del despliegue en producción debe realizarse una ceremonia de trusted setup Phase 2 específica para el circuito adaptado. La Phase 1 (Powers of Tau) puede reutilizarse de la ceremonia pública perpetua de Hermez/Polygon. La ceremonia Phase 2 puede realizarse vía navegador con snarkjs y debe contar con participantes independientes del equipo operador.
- `zkemail/zk-jwt` no debe usarse como dependencia directa. Puede consultarse como referencia de implementación para la adaptación del circuito base.
- La implementación del emisor con ZK está diseñada para ser desarrollada con asistencia de herramientas de IA. La corrección del código de integración puede validarse con tests funcionales estándar. La corrección de los constraints del circuito adaptado no puede validarse sin la auditoría especializada mencionada; esta limitación debe ser conocida por quien implemente el sistema.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza. La adopción de ZK en este ADR no contradice a P-0006: la mención de ZK en P-0006 es ilustrativa de un modelo de adversario total descartado, mientras que aquí ZK resuelve la auditabilidad de legitimidad del emisor dentro del modelo de amenazas intermedio.
- P-0013 — Integración con AUTENTICAR (Decisión 4 supersedada por este ADR)
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- `docs/zk_jwt_investigacion.md` — Investigación técnica de referencia
- `docs/autenticar.md` — Referencia técnica de AUTENTICAR (incluye JWKs de producción confirmadas como RS256-2048)
- `design/capa_de_identidad/identity_model.md` — Modelo de identidad y principio de integridad verificable
