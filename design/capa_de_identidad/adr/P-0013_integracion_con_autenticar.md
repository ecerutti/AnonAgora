# P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad

**Estado:** Parcialmente supersedido por P-0014

## Contexto

El sistema requiere verificar que cada ciudadano que solicita una identidad anónima es una persona real y única. Esta verificación es la base de la propiedad fundamental del sistema:

    una persona real → una identidad anónima activa

Sin un mecanismo de verificación confiable, un mismo actor podría obtener múltiples identidades anónimas y manipular los resultados de la plataforma (ataque Sybil). Además, el diseño debe evitar que un administrador con acceso al sistema pueda generar identidades anónimas ficticias sin respaldo en una persona real.

P-0007 define que la capa de identidad opera con proveedores externos de verificación de identidad y que el nivel de garantía de unicidad depende de las propiedades del proveedor. Este ADR concreta esa decisión para el contexto argentino: qué proveedor se usa, qué proveedores de identidad se aceptan dentro de ese proveedor, y cómo se deriva el identificador interno del sistema a partir de la verificación.

Las preguntas de diseño que motivaron este ADR son:

- ¿Qué proveedor externo de verificación de identidad se utiliza?
- ¿Qué proveedores de identidad se aceptan?
- ¿Cómo se deriva el `anon_seed` a partir del token de verificación?
- ¿Qué mecanismo de auditoría de legitimidad de identidades emitidas se adopta?

## Opciones consideradas

### Decisión 1 — Elección de AUTENTICAR como proveedor de verificación

#### Opción A — Integración directa con cada organismo estatal

La capa de identidad se integraría directamente con ARCA (ex-AFIP), ANSES, ReNaPer u otros organismos de forma individual, sin intermediario.

**Ventajas**

- Control directo sobre cada integración.

**Desventajas**

- Requiere integrarse, mantener y actualizar múltiples integraciones independientes.
- Mayor superficie operativa y de mantenimiento.
- No hay un protocolo uniforme entre organismos.

#### Opción B — AUTENTICAR como intermediario

La capa de identidad se integra con AUTENTICAR, que actúa como nexo entre la capa y los proveedores de identidad estatales. AUTENTICAR implementa OIDC sobre Keycloak y expone una interfaz uniforme independientemente del proveedor subyacente.

**Ventajas**

- Una sola integración cubre múltiples proveedores de identidad estatales.
- Protocolo estándar (OIDC/OAuth 2.0).
- Infraestructura existente del Estado argentino, sin costo adicional.
- Los tokens están firmados criptográficamente y son verificables offline con el JWKS público.
- Reduce la posibilidad de que un administrador genere identidades ficticias: el sistema no puede emitir una identidad anónima sin un token válido firmado por AUTENTICAR.

**Desventajas**

- Dependencia de disponibilidad de AUTENTICAR (sin SLA público documentado).
- El diseño del `anon_seed` debe considerar las particularidades técnicas de AUTENTICAR (variación del `sub` entre reinos, identificadores distintos por proveedor).

### Decisión 2 — Proveedores de identidad aceptados dentro de AUTENTICAR

AUTENTICAR integra múltiples proveedores: ARCA, ANSES, Mi Argentina, ReNaPer y NIC.ar. Cada uno entrega un identificador distinto en el token (CUIT, CUIL, DNI, pasaporte).

El problema de aceptar todos los proveedores es el siguiente: el `anon_seed` debe ser estable e independiente del proveedor usado en cada autenticación. Si la misma persona puede autenticarse con ARCA (entregando CUIT) o con ReNaPer (entregando DNI), el sistema calcularía dos `anon_seed` distintos y la trataría como dos personas diferentes. Esto permitiría eludir el límite de una identidad anónima activa por persona y el cool-down entre identidades, rompiendo la propiedad fundamental del sistema.

Para que el `anon_seed` sea estable, el identificador de base debe ser el mismo independientemente del proveedor usado en cada autenticación.

#### Opción A — Aceptar todos los proveedores disponibles en AUTENTICAR

**Ventajas**

- Mayor accesibilidad: cualquier ciudadano puede verificarse con las credenciales que tenga disponibles.

**Desventajas**

- Los distintos proveedores entregan identificadores en espacios distintos (CUIT, CUIL, DNI, pasaporte). No hay un identificador común universal.
- La misma persona autenticándose con distintos proveedores generaría `anon_seed` distintos, permitiendo múltiples identidades anónimas.
- Rompe la propiedad fundamental del sistema.

#### Opción B — Restringir a los proveedores que comparten espacio de identificadores

ARCA (ex-AFIP) entrega CUIT y ANSES entrega CUIL. En Argentina, CUIT y CUIL son el mismo número para personas físicas: una misma persona tiene idéntico valor en ambos proveedores. Esto permite construir un `anon_seed` estable e independiente de cuál de los dos proveedores usó el ciudadano en cada autenticación.

Ningún otro proveedor disponible en AUTENTICAR comparte este espacio de identificadores.

**Ventajas**

- El `anon_seed` es estable e independiente del proveedor usado.
- Garantía fuerte de unicidad: una persona real no puede obtener dos `anon_seed` distintos alternando entre ARCA y ANSES.
- Ambos proveedores ofrecen nivel mínimo 2, suficiente para verificar que el ciudadano es una persona real con credenciales estatales activas.

**Desventajas**

- Excluye ciudadanos que no tengan credenciales de ARCA ni de ANSES.
- Excluye extranjeros (cubiertos por NIC.ar o Mi Argentina con pasaporte).

### Decisión 3 — Fórmula del `anon_seed`

El `anon_seed` es el identificador derivado que el emisor usa para detectar si un ciudadano ya tiene una identidad anónima registrada. Debe ser determinista (el mismo ciudadano siempre produce el mismo valor), irreversible (no debe permitir reconstruir el identificador original) y estable entre los proveedores aceptados.

#### Opción A — Incluir el proveedor en el hash

    anon_seed = HASH(salt_del_sistema + proveedor + CUIT/CUIL)

**Ventajas**

- Previene colisiones hipotéticas entre proveedores con identificadores numéricos similares.

**Desventajas**

- La misma persona autenticándose por ARCA y por ANSES generaría `anon_seed` distintos, incluso siendo el mismo número. Esto rompe la unicidad entre proveedores.
- Para mantener la unicidad habría que normalizar el valor del proveedor a una constante fija, lo que haría el parámetro inútil.

#### Opción B — Hash solo sobre el identificador

    anon_seed = HASH(salt_del_sistema + CUIT/CUIL)

**Ventajas**

- El `anon_seed` es idéntico independientemente de si el ciudadano usó ARCA o ANSES.
- Simple y correcto dado que CUIT y CUIL son el mismo número para personas físicas.
- La unicidad entre proveedores está garantizada por la restricción de la Decisión 2.

**Desventajas**

- Depende de que el identificador sea efectivamente el mismo entre los proveedores aceptados, lo cual es una propiedad del conjunto restringido elegido.

### Decisión 4 — Mecanismo de auditoría de legitimidad de identidades emitidas

El emisor recibe un token firmado por AUTENTICAR como prueba de que existe un ciudadano real detrás de cada solicitud de identidad anónima. La pregunta de diseño es qué mecanismo permite auditar retrospectivamente que cada identidad anónima emitida corresponde efectivamente a un ciudadano real verificado, sin almacenar datos que permitan revelar la identidad real directamente o por correlación.

#### Por qué retener metadatos del token no resuelve el problema

Retener metadatos del token (como el `jti`, timestamps, proveedor o nivel) no proporciona auditoría fuerte de legitimidad por las siguientes razones:

- **El `jti` no es verificable externamente.** AUTENTICAR no expone un mecanismo público para confirmar si un `jti` existió realmente. Sin ese mecanismo, el `jti` solo habilita conteo interno, que es débil: un admin malicioso puede fabricar UUIDs que cuadren el conteo sin que correspondan a autenticaciones reales.
- **La firma del token no es separable del payload.** La verificación RSA requiere el header y el payload originales. Guardar solo la firma o solo el header no permite verificación posterior. Guardar el token completo equivale a almacenar la identidad real del ciudadano.
- **Los metadatos son correlacionables.** Cualquier metadato del token (proveedor, timestamps, `jti`) puede usarse para reconstruir el evento de autenticación en un escenario de colusión con AUTENTICAR, comprometiendo el anonimato del ciudadano.

Por lo tanto, retener metadatos del token tiene el costo de correlación sin el beneficio de auditoría fuerte que lo justificaría. La decisión de no retener metadatos no es una concesión: es la consecuencia natural de que ningún metadato disponible resuelve el problema de auditoría de forma segura con la integración OIDC estándar.

#### Opción A — Auditoría procedimental sin retención de metadatos

El token se descarta inmediatamente después de verificar la firma y calcular el `anon_seed`. No se retiene ningún dato del token.

La garantía contra identidades ficticias descansa en el diseño auditable del código del emisor: cualquier auditor puede verificar que el sistema está construido para requerir un token válido firmado por AUTENTICAR antes de emitir cualquier identidad anónima.

**Ventajas**

- Minimización máxima de datos en el emisor.
- Sin superficie de correlación.
- Coherente con el principio de minimización de P-0006.

**Desventajas y limitaciones explícitas**

- No permite auditoría forense retrospectiva de legitimidad de identidades individuales.
- No es posible demostrar criptográficamente que una `anon_id` específica tiene un ciudadano real detrás sin cruzar datos con AUTENTICAR.
- Un admin malicioso con acceso al emisor podría fabricar identidades anónimas sin que sea detectable retrospectivamente mediante datos almacenados.
- Las auditorías posibles son exclusivamente de índole procedimental: revisión del código fuente, verificación del diseño del sistema, y consistencia observable en el comportamiento del emisor.

#### Opción B — Auditoría criptográfica fuerte con pruebas de conocimiento cero (ZK)

En esta opción, antes de descartar el token, el ciudadano o el emisor genera una prueba ZK que demuestra: *"existe un token válido firmado por AUTENTICAR cuyo CUIL produce este `anon_seed`"*, sin revelar el CUIL ni el token. La prueba es verificable con el JWKS público de AUTENTICAR y se almacena asociada al `anon_seed`. No contiene datos correlacionables con la identidad real.

Proyectos como zkLogin y zk-JWT demuestran que este enfoque es técnicamente viable con tokens JWT estándar.

**Ventajas**

- Permite auditoría forense retrospectiva fuerte: cualquier auditor puede verificar que cada `anon_seed` tiene detrás un token real de AUTENTICAR usando solo el JWKS público, sin cruzar datos con AUTENTICAR ni con la plataforma participativa.
- Permite detectar identidades ficticias fabricadas por un admin malicioso: una identidad sin prueba ZK válida no puede superar la verificación.
- No retiene datos correlacionables.
- Resuelve el problema de auditoría de legitimidad de forma criptográficamente sólida y sin dependencia de terceros.

**Desventajas**

- Complejidad de implementación significativamente mayor.
- Requiere expertise en criptografía ZK, menos disponible que criptografía estándar.
- Las librerías disponibles son menos maduras que las de criptografía estándar.
- La auditoría del código del emisor se vuelve más compleja.
- Puede introducir latencia en el proceso de emisión por el tiempo de generación de pruebas.
- Requiere evaluación y decisión explícita antes de adoptar.

## Decisiones

**Decisión 1:** Se utiliza **AUTENTICAR** como proveedor de verificación de identidad.

**Decisión 2:** Se aceptan únicamente los proveedores **ARCA** y **ANSES**. Todo otro proveedor disponible en AUTENTICAR queda excluido para la emisión de identidades anónimas.

El criterio general para aceptar un proveedor es que entregue un identificador en el mismo espacio que los demás proveedores aceptados, y que su nivel mínimo sea suficiente para verificar que el ciudadano es una persona real con credenciales estatales activas. Si en el futuro se evalúa incorporar un nuevo proveedor, debe verificarse explícitamente que su identificador sea compatible con el espacio CUIT/CUIL y que el nivel mínimo disponible ofrezca garantías equivalentes.

El nivel mínimo requerido es **nivel 2**. Su justificación no es el número en sí sino el principio: se requiere el nivel mínimo que garantice verificación de una persona real con credenciales estatales activas en cada proveedor aceptado.

**Decisión 3:** El `anon_seed` se calcula como:

    anon_seed = HASH(salt_del_sistema + CUIT/CUIL)

El identificador extraído del token es el claim `cuit`, que es el claim específico del proveedor y el más confiable para este propósito. No se usa `preferred_username` (inestable entre proveedores) ni `sub` (varía entre reinos de Keycloak y no es un identificador universal).

**Decisión 4:** Se adopta temporalmente la **Opción A**. El token de AUTENTICAR se descarta inmediatamente tras verificar la firma y calcular el `anon_seed`. No se retiene ningún metadato del token.

Esta decisión está vigente hasta que la evaluación pendiente de ZK concluya. Si esa evaluación concluye en adoptar la Opción B, debe crearse un nuevo ADR que superseda la Decisión 4 de este documento.

Las implicaciones de esta decisión temporal son explícitas y conocidas:

- La garantía contra identidades ficticias en el emisor es procedimental, no criptográfica. Descansa en la auditabilidad del código fuente y el diseño del sistema, no en datos verificables retrospectivamente.
- No es posible detectar mediante auditoría forense si un admin malicioso con acceso al emisor fabricó identidades anónimas sin respaldo en ciudadanos reales.
- Las auditorías del emisor están limitadas a revisión de código, verificación del diseño, y consistencia observable del comportamiento del sistema.
- La auditoría de legitimidad en la capa de identidad —demostrar que cada `anon_id` emitido proviene de un ciudadano real verificado por AUTENTICAR— es un problema separado e independiente que se resuelve en P-0014.

## Justificación

AUTENTICAR permite verificar ciudadanos reales usando infraestructura estatal existente sin que el sistema gestione credenciales propias. Los tokens firmados criptográficamente limitan la posibilidad de que un administrador genere identidades ficticias, ya que no puede producir tokens válidos sin intervención de AUTENTICAR.

La restricción a ARCA y ANSES resuelve el problema central de unicidad: ambos proveedores entregan CUIT/CUIL, que es el mismo número para una misma persona física. Esto permite que el `anon_seed` sea estable independientemente del proveedor usado en cada autenticación, garantizando que una persona no pueda obtener múltiples identidades anónimas alternando entre proveedores.

La fórmula del `anon_seed` sin el proveedor en el hash es correcta precisamente porque el identificador es el mismo en ambos proveedores aceptados. Incluir el proveedor en el hash requeriría normalizarlo a una constante, lo que lo haría redundante y añadiría complejidad sin beneficio.

La decisión de no retener metadatos del token es la consecuencia directa de que ningún metadato disponible con la integración OIDC estándar resuelve el problema de auditoría de forma segura: retener metadatos tiene costo de correlación sin beneficio de auditoría fuerte verificable externamente. La auditoría criptográfica fuerte requiere ZK, cuya evaluación queda pendiente. Esta limitación es conocida, aceptada dentro del modelo de amenazas intermedio de P-0006, y debe revisarse si se adopta ZK.

## Consecuencias

- El emisor debe implementar el flujo OAuth 2.0 / OIDC con AUTENTICAR (Authorization Code Flow).
- El emisor debe verificar la firma del token offline contra el JWKS del reino correspondiente antes de procesar cualquier solicitud de emisión.
- El emisor debe extraer el claim `cuit` del token y calcular el `anon_seed` descartando el identificador original inmediatamente después.
- El emisor no debe almacenar: CUIT, CUIL, DNI, `sub`, `jti`, ni el token completo de AUTENTICAR. Ver P-0006 y `design/identity_model.md`.
- La auditoría de legitimidad del emisor es procedimental hasta que se evalúe y decida sobre ZK.
- La auditoría de legitimidad de los `anon_id` emitidos se resuelve de forma independiente en P-0014.
- Si en el futuro se evalúa incorporar un nuevo proveedor, debe verificarse compatibilidad de espacio de identificadores y nivel mínimo según el criterio definido en la Decisión 2.
- Si la evaluación de ZK concluye en su adopción, debe crearse un nuevo ADR que superseda la Decisión 4 de este documento.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0007 — Proveedores de verificación y niveles de garantía de unicidad
- `design/identity_model.md` — Modelo de identidad
- `design/threat_model.md` — Modelo de amenazas
- `docs/autenticar.md` — Referencia técnica de AUTENTICAR
