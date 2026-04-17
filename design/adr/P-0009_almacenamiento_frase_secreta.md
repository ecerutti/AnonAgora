# P-0009 — Recepción y almacenamiento de la frase secreta en la plataforma

## Contexto

La frase secreta definida por el ciudadano durante el registro en la plataforma es el único mecanismo de acceso y recuperación de su identidad anónima dentro de ella. Por esta razón, tanto el canal por el que la frase llega a la plataforma como el formato en que se almacena requieren protección especial.

La frase es asunto exclusivo de la plataforma participativa: el emisor no participa de su manejo, según P-0015. El ciudadano la define en su primer contacto con la plataforma y la usa en cada login posterior. La plataforma la recibe, la procesa, y almacena un valor derivado que le permita verificar futuros logins sin poder reconstruir la frase original.

Hay dos preguntas de diseño a resolver.

La primera es qué envía el cliente a la plataforma. Enviar la frase en texto plano es el comportamiento tradicional en sistemas de autenticación y simplifica el cliente, pero expone la frase a cualquier actor que pueda observar tráfico hacia la plataforma: administradores con acceso a terminación TLS, logs mal higienizados, o conexiones en redes comprometidas. Enviar un hash criptográfico de la frase desde el cliente reduce esa exposición sin agregar complejidad significativa.

La segunda es qué algoritmo aplica la plataforma sobre el valor recibido antes de almacenarlo. Almacenar el valor recibido tal cual es inaceptable, porque un acceso no autorizado a la base de datos expondría todas las frases del sistema. Es necesario elegir un algoritmo de derivación de claves que haga computacionalmente inviable recuperar la frase original a partir del valor almacenado.

## Opciones consideradas

### Decisión 1 — Qué envía el cliente a la plataforma

#### Opción A — Frase en texto plano

El cliente envía la frase tal como el ciudadano la escribió, sobre canal seguro (HTTPS/TLS). La plataforma recibe la frase, le aplica el algoritmo de derivación elegido, y almacena el resultado.

Ventajas

- Comportamiento tradicional, universalmente entendido.
- Implementación del cliente trivial: solo envía un string.

Desventajas

- La frase queda expuesta al punto de terminación TLS. Un administrador con acceso a ese punto puede ver la frase en claro.
- Un bug de logging, una configuración incorrecta de observabilidad o un middleware mal configurado pueden persistir la frase en logs.
- En entornos donde emisor y plataforma comparten infraestructura, el actor con acceso privilegiado ve la frase directamente, lo que es particularmente sensible en un sistema cuyo modelo de anonimato depende de minimizar lo que cada componente sabe del ciudadano.

#### Opción B — HASH(frase_secreta) calculado en el cliente

El cliente aplica una función hash criptográfica estándar (por ejemplo SHA-256) sobre la frase antes de enviarla a la plataforma, y envía únicamente el hash. La plataforma recibe el hash, le aplica el algoritmo de derivación elegido, y almacena el resultado. El mismo esquema se usa en registro y en login: el cliente siempre envía el hash, nunca la frase.

Ventajas

- La frase en claro nunca sale del dispositivo del ciudadano. Ningún componente servidor la ve.
- Un administrador con acceso a terminación TLS o a logs de la plataforma no obtiene la frase original: obtiene el hash, que sigue siendo resistente a preimagen.
- Alineado con la decisión análoga de P-0015 respecto a la interacción ciudadano-emisor: aplicar el mismo criterio a ambos tramos unifica el principio de no exponer la frase ante ningún componente servidor.

Desventajas

- Requiere que el cliente implemente la función hash. En la web moderna es trivial (WebCrypto API), pero introduce una dependencia del lado del cliente que antes no existía.
- El valor hasheado no protege contra ataques de fuerza bruta por sí solo, porque el espacio de frases posibles es limitado. La protección contra fuerza bruta sigue dependiendo del algoritmo de derivación que la plataforma aplica sobre el hash.

### Decisión 2 — Algoritmo de derivación aplicado por la plataforma

#### Opción 1 — bcrypt

Algoritmo ampliamente adoptado, diseñado específicamente para el hash de contraseñas. Incorpora salt automático y un factor de costo ajustable.

Ventajas

- Muy maduro y ampliamente soportado en todas las plataformas y lenguajes.
- Probado en producción durante décadas.
- Salt automático integrado.

Desventajas

- Limita la entrada a 72 bytes, lo que puede truncar passphrases largas sin advertir al usuario. Nota: esta desventaja se aplica cuando el input es la frase; con un hash de tamaño fijo como input (Opción B de Decisión 1) el truncado ya no aplica.
- Diseñado para hardware de hace décadas; los ataques con GPU modernos son más eficientes contra bcrypt que contra algoritmos más recientes.
- No permite configurar uso de memoria, lo que lo hace más vulnerable a ataques con hardware especializado (ASICs).

#### Opción 2 — PBKDF2

Estándar recomendado por NIST, basado en aplicar repetidamente una función hash (generalmente SHA-256 o SHA-512).

Ventajas

- Estándar reconocido por NIST y requerido en algunos contextos regulatorios.
- Amplio soporte en bibliotecas estándar.
- Sin límite práctico de longitud de entrada.

Desventajas

- Más eficiente en GPU que bcrypt o Argon2, lo que facilita ataques de fuerza bruta con hardware moderno.
- No tiene componente de uso de memoria, haciéndolo más vulnerable a hardware especializado.

#### Opción 3 — Argon2id

Ganador del Password Hashing Competition (2015). Combina resistencia a ataques de canal lateral (variante Argon2i) con resistencia a ataques de fuerza bruta con GPU y hardware especializado (variante Argon2d).

Ventajas

- Recomendación actual de OWASP para almacenamiento de contraseñas.
- Costoso en CPU y en memoria de forma configurable, dificultando ataques con GPU y ASICs.
- Sin límite de longitud de entrada.
- Salt único por registro generado automáticamente.
- Parámetros de costo (memoria, iteraciones, paralelismo) configurables y ajustables a futuro sin romper hashes existentes.

Desventajas

- Más reciente que bcrypt o PBKDF2, aunque ya cuenta con soporte maduro en las principales bibliotecas de los lenguajes más usados.

## Decisiones

**Decisión 1:** Se adopta la **Opción B**. El cliente calcula `HASH(frase_secreta)` localmente antes de enviar cualquier valor a la plataforma, y envía únicamente el hash. Esto aplica tanto en el registro inicial del ciudadano en la plataforma como en cada login posterior. La plataforma nunca recibe la frase secreta en texto plano.

La función hash del cliente debe ser una función criptográfica estándar (por ejemplo SHA-256). La función elegida debe ser fija para todo el sistema y conocida por la plataforma, ya que la plataforma necesita que el cliente calcule el mismo hash en registro y en login para que la comparación sea correcta.

**Decisión 2:** Se adopta la **Opción 3 — Argon2id**. La plataforma aplica Argon2id al `HASH(frase_secreta)` recibido del cliente, con salt único generado aleatoriamente por registro. El valor almacenado es el hash Argon2id resultante junto con su salt.

Los parámetros de costo (memoria, iteraciones, paralelismo) son configurables por el operador y deben revisarse periódicamente a medida que el hardware disponible evolucione. El `HASH(frase_secreta)` recibido se descarta inmediatamente después de calcular el Argon2id y nunca se persiste.

## Justificación

El envío del hash desde el cliente en lugar de la frase en claro responde al mismo principio que P-0015 aplicó al tramo entre el ciudadano y el emisor: minimizar los componentes servidor que ven la frase en su forma original. Aunque TLS protege el canal, no protege contra el punto de terminación del canal ni contra los componentes que procesan la frase después de su recepción. Aplicar el hash en el cliente garantiza que la frase en claro exista únicamente en el dispositivo del ciudadano, y que ningún servidor (emisor o plataforma) la vea jamás.

Esta decisión es independiente de la decisión análoga en P-0015: cada componente servidor decide por su cuenta qué recibe del cliente. Que ambos coincidan en recibir el hash es consecuencia de que la amenaza (exposición ante administradores locales, logs mal higienizados, infraestructura compartida) es la misma en ambos tramos.

El hash enviado por el cliente no protege por sí solo contra ataques de fuerza bruta offline: un atacante con acceso a la base de datos que quiera recuperar la frase original puede probar frases candidatas, aplicarles el mismo hash del cliente, y comparar. La protección contra ese tipo de ataque vive en el algoritmo de derivación que la plataforma aplica sobre el hash recibido. El hash del cliente y Argon2id en la plataforma cumplen roles distintos y complementarios.

Argon2id es la recomendación actual de OWASP para almacenamiento de contraseñas y el resultado del proceso de selección más riguroso realizado por la comunidad de seguridad (PHC). Su diseño híbrido lo hace resistente tanto a ataques de canal lateral como a ataques de fuerza bruta con hardware especializado, que son los vectores más relevantes en un sistema donde el anonimato del ciudadano depende de que su frase secreta no pueda recuperarse aunque un atacante obtenga acceso a la base de datos. Aplicar Argon2id sobre `HASH(frase_secreta)` en lugar de sobre la frase directa no altera las propiedades de seguridad del algoritmo: Argon2id opera sobre un input de longitud arbitraria, y el hash del cliente es simplemente el input que recibe en este diseño.

## Consecuencias

- El cliente debe implementar el cálculo del hash criptográfico estándar sobre la frase antes de enviarla a la plataforma, tanto en registro como en login.
- La función hash del cliente debe documentarse como parte de la especificación del protocolo entre cliente y plataforma, para que implementaciones alternativas del cliente sean interoperables con la plataforma.
- La plataforma recibe `HASH(frase_secreta)` del cliente, le aplica Argon2id con salt único generado aleatoriamente, y almacena el resultado junto con el salt y los parámetros de costo utilizados.
- La base de datos almacena únicamente el hash Argon2id, el salt, y los parámetros de costo; nunca la frase en texto plano ni el `HASH(frase_secreta)` intermedio.
- Los parámetros de costo son configurables y deben documentarse en la guía de instalación y operación de la plataforma.
- La verificación de la frase en el login consiste en recibir el `HASH(frase_secreta)` calculado por el cliente, recomputar Argon2id con el salt y parámetros almacenados, y comparar con el valor guardado, sin necesidad de conocer la frase original.
- Si en el futuro se decidiera cambiar la función hash del cliente, todos los registros existentes quedarían inutilizables y los ciudadanos deberían esperar el cool-down para registrar nuevas identidades. Por eso la función hash debe elegirse con visión de largo plazo, usando una función criptográfica estándar y estable.

## Referencias

- P-0004 — Autenticación de identidad anónima
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0008 — Mecanismo de credencial de acceso: passphrase vs password
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
