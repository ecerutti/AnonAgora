# P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero

## Contexto

El emisor es el componente responsable de verificar que un ciudadano corresponde a una persona real —mediante un token JWT firmado por AUTENTICAR— y de emitir la identidad anónima (`anon_id`) correspondiente. Una vez completado ese proceso, el token se descarta sin retener ningún metadato, por decisión de P-0013.

Esta ausencia de retención introduce una limitación conocida: no es posible demostrar retrospectivamente que un `anon_seed` específico fue generado a partir de un token real de AUTENTICAR. Un administrador malicioso con acceso al emisor podría fabricar identidades anónimas sin que ningún ciudadano real las respalde, y esa manipulación sería indetectable mediante auditoría forense sobre los datos almacenados. La única defensa disponible bajo P-0013 Decisión 4 es procedimental: revisión del código fuente y del diseño del sistema.

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

Antes de descartar el token, el emisor genera una prueba ZK que demuestra: *"existe un token JWT válido firmado por AUTENTICAR cuyo CUIT produce este `anon_seed`"*, sin revelar el CUIT ni el token. La prueba se almacena asociada al `anon_seed` y es verificable públicamente usando solo el JWKS de AUTENTICAR y la clave de verificación derivada del trusted setup.

Ventajas

- Auditoría forense retrospectiva fuerte: cualquier auditor puede verificar que cada `anon_seed` tiene un token real de AUTENTICAR detrás, usando solo información pública, sin cruzar datos con AUTENTICAR ni con la plataforma participativa.
- Sin un token real de AUTENTICAR, es criptográficamente imposible generar una prueba válida. Elimina la capacidad del admin malicioso de fabricar identidades.
- La prueba no contiene datos correlacionables con la identidad real del ciudadano.
- Compatible con el requisito de multi-dispositivo: la prueba se asocia al `anon_seed`, no al dispositivo del ciudadano.

Desventajas

- El emisor ve el JWT durante la generación de la prueba. Esta ventana de exposición del CUIT al emisor ya existe en el diseño actual; ZK no la elimina pero agrega el recibo criptográfico de que ese JWT existió y era válido.
- Complejidad de implementación significativamente mayor que la auditoría procedimental.
- Requiere una ceremonia de trusted setup Phase 2 específica para el circuito adaptado, que debe realizarse antes de cualquier despliegue en producción.
- Requiere mantener un registro histórico de JWKs de AUTENTICAR (JWKS histórico) para que las pruebas sean verificables tras rotaciones de clave.
- El circuito adaptado al caso concreto del sistema no está auditado y requiere auditoría de seguridad especializada antes de cualquier despliegue en producción.

#### Opción C — Adoptar ZK con generación de prueba en el cliente

El ciudadano genera la prueba localmente en su dispositivo antes de enviar ningún dato sensible al emisor. El emisor nunca ve el token.

Ventajas

- El emisor nunca tiene acceso al JWT ni al CUIT. Elimina completamente la ventana de exposición.

Desventajas

- Tiempos de generación estimados de 60-180 segundos en smartphones de gama media-baja, inaceptables para la experiencia del usuario.
- El archivo `.zkey` (proving key) pesa cientos de MB y debe descargarse al dispositivo antes de poder generar la prueba.
- Probable fallo por memoria insuficiente en dispositivos con menos de 3-4 GB de RAM libre, segmento de hardware habitual entre usuarios de la plataforma.

### Decisión 2 — Stack tecnológico y circuito base

Esta decisión solo aplica si la Decisión 1 adopta ZK.

#### Opción A — circom + snarkjs con circuito base de zk-email-verify

Usar el circuito RSA de `zkemail/zk-email-verify` (MIT, auditado por zkSecurity y yAcademy en mayo 2024) como núcleo de verificación de firma RS256, adaptado para extraer el claim `cuit` y calcular el `anon_seed`. El proving y la verificación se realizan con snarkjs (GPL-3, v0.7.6) en Node.js en el servidor.

El esquema de prueba es Groth16 sobre la curva BN254. El circuito RSA-2048 tiene aproximadamente 1,5 millones de constraints, con tiempos de proving de 15-45 segundos en servidor estándar de 8 cores. Las pruebas Groth16 tienen un tamaño de ~256 bytes y se verifican en 1-5 ms en servidor.

Ventajas

- El núcleo RSA está auditado. La parte no auditada se limita a la adaptación de extracción de claim y cálculo de `anon_seed`.
- La misma pila es la que usa zkLogin de Sui en producción desde 2023.
- Compatible con RSA-2048, que es el tamaño de clave confirmado en el JWKS de los reinos `afip` y `anses` de AUTENTICAR.
- Pruebas compactas (~256 bytes) y verificación rápida (1-5 ms en servidor).

Desventajas

- La adaptación del circuito (extracción de `cuit`, cálculo de `anon_seed`, separación de dominios en el hash Poseidon) no está auditada y debe serlo antes de cualquier despliegue en producción. El costo estimado de mercado para una auditoría especializada en ZK es de USD 30.000–150.000 según el tamaño del circuito y la firma auditora.
- `zkemail/zk-jwt`, que preempaqueta una adaptación similar sobre el mismo circuito base, se autodeclara no apto para producción y no está auditado; no puede usarse directamente.
- Requiere expertise en circom para realizar y validar la adaptación.
- Requiere una ceremonia de trusted setup Phase 2 específica para el circuito adaptado.

#### Opción B — zkVM (RISC Zero o SP1)

Implementar la verificación del JWT RS256 como código Rust ejecutado en una zkVM. RISC Zero (v3.0.5, Apache/MIT) y SP1 (v6.1.0, MIT) están en producción con auditorías publicadas. El "circuito" es código Rust estándar que verifica el JWT, extrae el CUIT y calcula el `anon_seed`.

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
- Definir `anon_seed = HASH(salt_del_sistema + cuit)` como output público, usando separación de dominios en el hash Poseidon para evitar colisiones cruzadas con otros usos del mismo primitivo.
- Exponer el `publicKeyHash` (hash Poseidon de la clave pública RSA) y el `kid` como outputs públicos para vincular la prueba a la JWK correspondiente del JWKS de AUTENTICAR.

El circuito núcleo RSA de `zk-email-verify` no debe modificarse. Las únicas modificaciones permitidas son las que configuran los parámetros de extracción de claim sin alterar la lógica de verificación de firma.

## Justificación

La adopción de ZK es coherente con el principio de integridad verificable del sistema definido en `design/identity_model.md`: el sistema debe poder demostrar su correcto funcionamiento sin depender de confianza ciega en los operadores. La auditoría procedimental cumple ese objetivo de forma débil —depende de que alguien revise el código y confíe en que lo que ejecuta en producción coincide con lo auditado. ZK lo cumple de forma fuerte: la prueba es verificable matemáticamente por cualquier tercero con información pública, sin cruzar datos con ningún componente del sistema.

El escenario de administrador malicioso con acceso al emisor es plausible en el contexto operativo habitual del sistema, donde emisor y plataforma participativa pueden compartir infraestructura física. ZK elimina la capacidad de ese actor de fabricar `anon_seeds` sin ciudadanos reales detrás, que es la amenaza específica que P-0013 Decisión 4 dejaba sin cobertura.

La generación en servidor se elige sobre la generación en cliente porque los tiempos de proving en cliente (60-180 segundos estimados en dispositivos de gama media-baja) son inaceptables para la experiencia del usuario, y los requisitos de RAM del archivo `.zkey` son incompatibles con smartphones del segmento objetivo.

La pila circom + snarkjs sobre `zk-email-verify` se elige sobre zkVMs porque el núcleo RSA ya está auditado, el stack es el más establecido para este caso de uso (zkLogin de Sui lo usa en producción), las pruebas Groth16 son las más compactas (~256 bytes) y la verificación es la más rápida (1-5 ms). Las zkVMs ofrecen menor barrera de entrada en implementación pero mayor dependencia en infraestructura externa y menor control sobre la auditabilidad del circuito resultante.

## Consecuencias

- El emisor debe implementar un componente de proving ZK que, antes de descartar el token de AUTENTICAR, genere una prueba Groth16 usando el circuito adaptado de `zk-email-verify`. La prueba se almacena asociada al `anon_seed` en la base de datos del emisor.
- La prueba tiene un tamaño de ~256 bytes y no contiene datos correlacionables con la identidad real del ciudadano.
- La prueba ZK certifica la legitimidad del `anon_seed`. La auditoría de legitimidad en la plataforma participativa —que trabaja con `anon_id` y no con `anon_seed`— es un problema separado que se resuelve en P-0015 mediante firma del emisor sobre cada `anon_id`. El vínculo entre ambas capas de auditoría queda definido en ese ADR.
- El emisor debe implementar un servicio de JWKS histórico que registre todas las JWKs que AUTENTICAR haya publicado, indexadas por `kid`. Este componente es necesario para que las pruebas generadas bajo claves rotadas sigan siendo verificables. Referencia de implementación: `historical-jwks-zklogin` de MystenLabs.
- El emisor debe implementar un mecanismo de revocación por `kid` comprometido: si AUTENTICAR reporta una clave privada comprometida, el sistema debe poder marcar ese `kid` como inválido y gestionar las identidades emitidas bajo él. El `kid` es output público de cada prueba, lo que permite esta operación sin revelar datos privados.
- La adaptación del circuito requiere auditoría de seguridad especializada en ZK antes de cualquier despliegue en producción. El costo estimado es USD 30.000–150.000 según tamaño del circuito y firma auditora.
- Antes del despliegue en producción debe realizarse una ceremonia de trusted setup Phase 2 específica para el circuito adaptado. La Phase 1 (Powers of Tau) puede reutilizarse de la ceremonia pública perpetua de Hermez/Polygon. La ceremonia Phase 2 puede realizarse vía navegador con snarkjs y debe contar con participantes independientes del equipo operador.
- `zkemail/zk-jwt` no debe usarse como dependencia directa. Puede consultarse como referencia de implementación para la adaptación del circuito base.
- La implementación del emisor con ZK está diseñada para ser desarrollada con asistencia de herramientas de IA. La corrección del código de integración puede validarse con tests funcionales estándar. La corrección de los constraints del circuito adaptado no puede validarse sin la auditoría especializada mencionada; esta limitación debe ser conocida por quien implemente el sistema.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0013 — Integración con AUTENTICAR (Decisión 4 supersedada por este ADR)
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- `docs/zk_jwt_investigacion.md` — Investigación técnica de referencia
- `docs/autenticar.md` — Referencia técnica de AUTENTICAR (incluye JWKs de producción confirmadas como RS256-2048)
- `design/identity_model.md` — Modelo de identidad y principio de integridad verificable
