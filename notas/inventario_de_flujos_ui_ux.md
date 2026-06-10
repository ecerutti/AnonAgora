# Inventario de flujos de UI/UX del sistema

## Propósito

Este documento inventaría los flujos de UI/UX del sistema: el conjunto de recorridos de pantallas e interacciones que un ciudadano atraviesa para hacer cualquier cosa que el sistema le permite hacer. Es un material preparatorio de la fase de diseño de interfaces y desarrollo, y vive en `notas/` por esa razón.

El inventario describe la superficie de UI que se desprende de decisiones de diseño ya tomadas, registradas en los ADRs del sistema. No toma decisiones de producto nuevas. Cuando un recorrido revela una pregunta de UX que ningún ADR resolvió, esa pregunta se eleva como hueco al final del documento; no se inventa una respuesta.

## Alcance

El inventario cubre todos los flujos del sistema con interfaz visible al ciudadano: tanto los de la capa de identidad (la verificación con el verificador externo y la emisión de identidad anónima) como los de la aplicación destino (el uso cotidiano de la plataforma de participación ciudadana).

El operador queda fuera. P-0024 establece que el sistema no tiene perfil administrativo y que toda configuración y operación excepcional se ejerce mediante archivos de configuración y herramientas de línea de comando sobre la infraestructura. Las operaciones del operador con efectos visibles para el ciudadano —notablemente el retiro de una propuesta por causal legal (P-0023)— aparecen como consecuencias dentro de los flujos correspondientes (en este caso, la variante tombstone de la visualización de una propuesta), pero no como flujos independientes.

## Qué este documento NO es

- **No es un documento de decisión**. No reemplaza ni complementa ningún ADR.
- **No es una especificación visual**. No prescribe layouts, copies, paletas ni componentes concretos.
- **No es exhaustivo respecto a estados de error de implementación**. Captura las bifurcaciones que tienen consecuencia visible sobre la experiencia del ciudadano, no cada modo de falla técnico.

## Convenciones

- **Identificadores**: `F-CI-XX` para flujos de la capa de identidad, `F-AP-XX` para flujos de la aplicación destino (participación ciudadana). `M-XX` para modificadores transversales que aplican a múltiples flujos.
- **Vocabulario**: consistente con `design/glosario.md`. "El sistema" refiere al conjunto capa de identidad + aplicación destino; "la plataforma" refiere específicamente a la aplicación de participación ciudadana.
- **Referencias a ADRs**: cada flujo lista los ADRs que lo gobiernan. El inventario remite a ellos en lugar de reproducir sus decisiones.

## Resumen de flujos

| ID | Nombre | Componente |
|---|---|---|
| F-CI-01 | Emisión de identidad anónima | Capa de identidad |
| F-AP-00 | Acceso público al sitio (landing) | Aplicación destino |
| F-AP-01 | Onboarding inicial post-emisión | Aplicación destino |
| F-AP-02 | Login | Aplicación destino |
| F-AP-03 | Cierre y expiración de sesión | Aplicación destino |
| F-AP-04 | Exploración del ranking de propuestas | Aplicación destino |
| F-AP-05 | Búsqueda y filtrado de propuestas | Aplicación destino |
| F-AP-06 | Visualización de una propuesta y sus vínculos | Aplicación destino |
| F-AP-07 | Dar / retirar apoyo a propuesta | Aplicación destino |
| F-AP-08 | Creación de propuesta (original o derivada) | Aplicación destino |
| Expiración de sesión | (comportamiento transversal) | Aplicación destino |
| M-DEBUG | Modo debug activo (modificador transversal) | Capa + aplicación |

## Bloque 1 — Capa de identidad

### F-CI-01 — Emisión de identidad anónima

**Componente**: capa de identidad (emisor + integración con verificador externo + componente de proving ZK).

**ADRs y documentos que lo gobiernan**: P-0002, P-0003, P-0007, P-0013, P-0014, P-0015, P-0020, P-0022. También `design/capa_de_identidad/README.md`, `design/capa_de_identidad/identity_model.md`, `design/capa_de_identidad/identity_wordlists.md`, `docs/autenticar.md`.

**Condición de entrada**

El ciudadano llega al emisor desde F-AP-00 (landing) eligiendo "registrarse", o directamente al punto de entrada del emisor si conoce su URL. No se requiere estado previo del lado del ciudadano. Si tuvo una identidad anterior, la fórmula determinista `anon_seed = HASH(salt_del_sistema + CUIT/CUIL)` lo reidentifica internamente al verificarse con el verificador externo, lo que permite que el cool-down opere sin necesidad de credenciales previas. Para el emisor no hay diferencia entre un primer registro y una renovación tras cool-down; la única consecuencia observable de esa distinción aparece en el paso de verificación de cool-down.

**Condición de salida**

- *Éxito*: el emisor entregó la tupla `{pseudónimo, anon_id, prueba ZK}` a la aplicación destino y queda esperando la confirmación de recepción. La tupla `{anon_seed, fecha_emision}` aún no se persiste; la persistencia ocurre solo tras recibir la confirmación desde F-AP-01 (P-0022 Dec 3). El ciudadano continúa la experiencia de forma transparente en F-AP-01.
- *Rechazo por cool-down vigente*: el ciudadano ve un mensaje informativo con la fecha a partir de la cual puede solicitar una nueva identidad (resolución de H-5). Sin emisión. No es un error: es el funcionamiento esperado del sistema.
- *Rechazo por el verificador externo*: AUTENTICAR rechaza la solicitud o no responde tras agotar reintentos. El ciudadano ve uno de los tres mensajes categorizados de P-0022 Dec 1. Sin emisión.
- *Rechazo por el componente de proving ZK*: el proving falla. El ciudadano ve mensaje genérico de P-0022 Dec 2. Sin emisión, sin consumo de cool-down.
- *Abandono*: el ciudadano cierra el navegador o cancela durante el flujo de AUTENTICAR. Sin emisión, sin estado persistido.

**Pasos / pantallas**

1. *Inicio del registro en el emisor*. Pantalla inicial: explicación breve de qué va a pasar (verificación de identidad estatal y emisión de identidad anónima) y botón para iniciar. UX concreta pendiente.
2. *Redirección al verificador externo*. El emisor construye la URL de autorización según P-0013 y `docs/autenticar.md`, y redirige el navegador del ciudadano al endpoint correspondiente. El ciudadano sale del sistema y entra al flujo de AUTENTICAR.
3. *Autenticación en AUTENTICAR (fuera del sistema)*. El ciudadano completa la autenticación con ARCA o ANSES. Esta interacción no pertenece al sistema; el inventario solo registra que ocurre.
4. *Callback al emisor*. AUTENTICAR redirige al ciudadano de vuelta al emisor con un `code`. El emisor intercambia el `code` por tokens server-to-server.
5. *Verificación del JWT y derivación del anon_seed*. El emisor verifica la firma del JWT contra el JWKS de AUTENTICAR, extrae el claim `cuit` y calcula `anon_seed`. El token se descarta tras este paso (P-0013 Decisión 4 con la corrección posterior de P-0014).
6. *Verificación de unicidad y cool-down*. El emisor consulta su modelo de datos y busca un registro previo con el mismo `anon_seed`. Si existe y el cool-down aún no venció, bifurca al camino de rechazo por cool-down vigente.
7. *Generación de nonce, anon_id y prueba ZK*. El emisor genera el `nonce` aleatorio, calcula `anon_id = HASH(anon_seed + nonce)`, genera la prueba ZK con el circuito adaptado (P-0014, P-0015 Dec 3 y 4) y descarta el `nonce` inmediatamente.
8. *Generación y presentación del pseudónimo*. El emisor genera un pseudónimo amigable del formato `animal + color/adjetivo + número [+ letra]` según `design/capa_de_identidad/identity_wordlists.md` y P-0025. Antes de presentarlo al ciudadano, consulta a la aplicación destino la disponibilidad y obtiene reserva con TTL (P-0025). Si el candidato está ocupado, regenera y consulta nuevamente. Tras un umbral configurable de intentos consecutivos con candidatos sin sufijo de letra ocupados, el emisor comienza a generar candidatos con sufijo de letra. Una vez obtenido un candidato con reserva, lo presenta al ciudadano junto con un botón para generar otro (resolución de H-9). Si el ciudadano regenera, el emisor libera explícitamente la reserva anterior y vuelve al inicio de este paso. El ciudadano itera tantas veces como quiera y finalmente acepta. Una vez aceptado, el pseudónimo queda fijado a la identidad emitida (P-0003).
9. *Entrega a la aplicación destino*. El emisor entrega la tupla `{pseudónimo, anon_id, prueba ZK}` a la aplicación destino y espera la confirmación. El cool-down todavía no se consume. El ciudadano transiciona a F-AP-01 sin interrupción visible. La modalidad concreta de la entrega (redirección con datos, llamada server-to-server, vehiculizada por el navegador o no) es decisión de implementación y queda fuera del alcance de este inventario. En el commit de la entrega, la aplicación destino vuelve a verificar atómicamente la disponibilidad del pseudónimo (P-0025). En el caso patológico de colisión post-TTL (reserva expirada y pseudónimo tomado por otra emisión), la entrega falla con código específico, el flujo retoma en el paso 8 con un pseudónimo nuevo y un mensaje informativo al ciudadano, sin consumir cool-down (P-0022 Decisión 3).

**Bifurcaciones, errores y degradación**

- *Cool-down vigente* (paso 6): mensaje informativo con la fecha del próximo slot disponible, derivada de `fecha_emision + cool-down`. Sin emisión, sin error. (Resolución de H-5.)
- *AUTENTICAR no responde o error transitorio* (pasos 2-4): mensaje categoría 1 de P-0022 Dec 1, tras reintentos server-to-server agotados con los defaults configurados.
- *Token vencido en tránsito o `invalid_grant`* (paso 4): mensaje categoría 2 de P-0022 Dec 1. El ciudadano debe iniciar el flujo nuevamente.
- *AUTENTICAR responde `access_denied`* (paso 3 o 4): mensaje categoría 3 de P-0022 Dec 1. Sin reintento automático.
- *JWKS de AUTENTICAR no disponible* (paso 5): mensaje categoría 1 de P-0022 Dec 1.
- *Fallo del componente de proving ZK* (paso 7): mensaje de P-0022 Dec 2. Errores transitorios reintentan automáticamente; errores deterministas fallan de inmediato. Sin consumo de cool-down porque el flujo se interrumpe antes de la entrega.
- *Fallo en la entrega o no llega confirmación de la aplicación destino* (paso 9): el cool-down no se consume (P-0022 Dec 3). El ciudadano puede reintentar el flujo completo. La política de reintentos de la entrega misma queda diferida al ADR posterior que defina el contrato concreto emisor↔aplicación destino (referenciada en P-0022).
- *Colisión post-TTL en el commit de entrega* (paso 9): la aplicación destino rechaza la persistencia. El flujo retoma en el paso 8 con un pseudónimo nuevo y mensaje informativo. Sin consumo de cool-down (P-0025, P-0022 Dec 3).
- *Fallo de la consulta-con-reserva o de la liberación a la aplicación destino* (paso 8): reintentos server-to-server con backoff (P-0025, P-0022 Dec 3). Sin consumo de cool-down.
- *M-DEBUG*: si el modo debug está activo, la pantalla del paso 1 muestra advertencia prominente; el paso 8 (aceptación del pseudónimo, momento que materializa la emisión) requiere confirmación explícita. UX pendiente.

**Notas**

- El ciudadano nunca ve los términos `anon_seed`, `anon_id` ni "prueba ZK" en pantalla. El pseudónimo amigable es la única representación visible de la identidad. El resto es maquinaria interna del sistema.
- Para el emisor, una renovación tras cool-down es indistinguible operacionalmente de un primer registro: la única diferencia es la existencia previa de un `anon_seed` con su `fecha_emision`. Para el ciudadano, la diferencia es cognitiva (ya conoce el sistema), no visual.
- La frase secreta, el tutorial y los T&C no intervienen en este flujo. Pertenecen al alcance exclusivo de la aplicación destino (P-0015, contrato capa↔aplicación) y se tratan en F-AP-01.

## Bloque 2 — Entrada al sistema (aplicación destino)

### F-AP-00 — Acceso público al sitio (landing)

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0021, P-0005. Material narrativo de referencia en `docs/propuesta/03_Cómo_se_usaría.md` (descripción de la primera llegada del ciudadano).

**Condición de entrada**

El ciudadano accede a la URL pública del despliegue sin sesión activa. Es el punto de entrada principal al sistema. Aplica tanto al ciudadano que llega por primera vez como al que tuvo sesión y la cerró (o expiró), porque P-0005 establece que la aplicación destino no preserva memoria de la identidad anónima usada previamente entre sesiones.

**Condición de salida**

- *Bifurcación a F-CI-01 (registro)*: el ciudadano elige iniciar el flujo de emisión de identidad anónima.
- *Bifurcación a F-AP-02 (login)*: el ciudadano elige autenticarse con identidad y frase secreta que ya posee.
- *Salida del sitio*: el ciudadano abandona sin tomar acción. Sin estado persistido.

**Pasos / pantallas**

1. *Pantalla de bienvenida pública*. Explica brevemente el propósito del sistema (termómetro social, identidad anónima verificada) y ofrece dos puntos de entrada explícitos: "registrarme" e "ingresar". El contenido informativo concreto es decisión UX; el inventario solo registra la existencia de estos dos puntos de entrada.

**Bifurcaciones, errores y degradación**

- *M-DEBUG*: si el modo debug está activo, la pantalla muestra la advertencia transversal. Como F-AP-00 no genera eventos logueables propios ni modifica estado, no requiere confirmación adicional.

**Notas**

- Si los visitantes no autenticados pueden o no ver el listado de propuestas desde esta pantalla, o desde una sección pública adyacente, es una decisión de diseño no tomada por ningún ADR. Está registrada como hueco H-1 al final del documento.

### F-AP-01 — Onboarding inicial post-emisión

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0001, P-0005, P-0008, P-0009, P-0014, P-0015, P-0020, P-0022. También `design/capa_de_identidad/README.md` (contrato capa↔aplicación).

**Condición de entrada**

Dos caminos de entrada al mismo flujo, con la misma lógica interna pero distinto estado inicial:

1. *Desde F-CI-01*: el emisor acaba de entregar la tupla `{pseudónimo, anon_id, prueba ZK}` a la aplicación destino. Aún no existe registro del ciudadano en la base de la aplicación. Ninguno de los hitos del onboarding (frase secreta, tutorial, T&C) está cumplido. El emisor está esperando confirmación de recepción para consumir el cool-down (P-0022 Dec 3).
2. *Desde F-AP-02*: el ciudadano se autenticó con éxito pero el chequeo de pendientes detectó que tutorial o T&C no fueron completados en sesiones anteriores. La identidad ya existe en la base, la frase secreta ya está persistida; lo que falta es lo que el ciudadano no terminó. El emisor ya recibió su confirmación en su momento.

**Condición de salida**

- *Éxito (todos los hitos completos)*: el ciudadano queda con identidad registrada, frase secreta persistida, tutorial completado, T&C aceptados, sesión activa. Aterriza en la página principal (F-AP-04). En el camino desde F-CI-01, la confirmación al emisor ya se envió al persistirse la frase; en el camino desde F-AP-02, no hay confirmación al emisor pendiente.
- *Rechazo de T&C*: el ciudadano declina aceptar los T&C. Mensaje breve invitando a volver y cierre de sesión (transición a F-AP-03). El estado de "T&C aceptados" queda en `false`; en próximos logins F-AP-02 detectará el pendiente y volverá a redirigir aquí.
- *Abandono antes de persistir frase (solo desde F-CI-01)*: el ciudadano cierra el navegador o cancela durante la definición de la frase secreta. Nada se persiste en la aplicación destino, no se envía confirmación al emisor, el cool-down no se consume. El ciudadano puede reintentar F-CI-01 desde cero.
- *Abandono después de persistir frase pero antes de completar tutorial o aceptar T&C*: la identidad queda registrada con los flags de pendientes correspondientes. En el próximo login, F-AP-02 redirige aquí mostrando solo lo que falta.

**Pasos / pantallas**

Los pasos 1 a 4 aplican únicamente al camino desde F-CI-01. Los pasos 5 a 8 aplican siempre, pero el flujo desde F-AP-02 entra directamente en el primero de los pasos 5-7 que tenga pendiente.

1. *Recepción y verificación de la prueba ZK*. La aplicación destino recibe la tupla del emisor y verifica la prueba ZK contra el JWKS histórico de la capa de identidad, según P-0014. La verificación es interna y no produce pantalla; si falla, el ciudadano ve un mensaje genérico de error sin detalle operativo (P-0020) y la tupla se descarta sin enviar confirmación al emisor. Caso no esperado en operación normal.
2. *Definición de la frase secreta*. Pantalla con explicación clara del concepto de passphrase (P-0008 destaca que muchos ciudadanos solo conocen "contraseña" como término), un ejemplo o sugerencia ilustrativa, y los criterios mínimos configurables (default: 4 palabras, 20 caracteres). Advertencia inequívoca sobre irrecuperabilidad: si pierde la frase, pierde la identidad hasta vencido el cool-down. El ciudadano ingresa su frase.
3. *Persistencia de la frase y registro del ciudadano*. El cliente calcula `HASH(frase_secreta)` con la función hash estándar acordada en P-0009 y la envía a la aplicación destino. El servidor genera salt único, aplica Argon2id con los parámetros configurados, y persiste el registro: `{anon_id, pseudónimo, hash_argon2id, salt, parámetros, fecha_registro (granularidad de día), tutorial_completado=false, tc_aceptados=false}`. Los textos exactos de los campos de control y su estructura concreta son decisión de implementación, no de diseño.
4. *Confirmación al emisor*. Inmediatamente después de persistir el registro, la aplicación destino envía la confirmación de recepción al emisor. A partir de este momento, el emisor persiste `{anon_seed, fecha_emision}` y consume el cool-down (P-0022 Dec 3). El ciudadano no percibe este paso; ocurre transparentemente.
5. *Tutorial introductorio*. El ciudadano recorre el tutorial explicativo del sistema. El contenido cubre, según `docs/propuesta/03_Cómo_se_usaría.md`: propósito de la plataforma, formas de participación disponibles (apoyar propuestas existentes y crear nuevas), límite anual de propuestas, recomendación de buscar antes de crear. El recorrido es bloqueante: el ciudadano no puede saltearlo ni acceder al resto de la aplicación hasta completarlo. Al finalizar, la aplicación setea `tutorial_completado=true`.
6. *Presentación de los T&C*. Una vez completado el tutorial, la aplicación muestra los Términos y Condiciones. El ciudadano puede aceptarlos o declinarlos. Aceptarlos setea `tc_aceptados=true` y avanza al paso 7. Declinarlos transita al rechazo de T&C (ver bifurcaciones).
7. *Aterrizaje en la página principal*. El ciudadano queda con sesión activa y entra en F-AP-04. El pseudónimo es visible a partir de este momento como elemento persistente de la interfaz (forma concreta pendiente, H-6).

**Bifurcaciones, errores y degradación**

- *Fallo de verificación de la prueba ZK* (paso 1): la tupla se descarta sin confirmar al emisor; el cool-down no se consume. Mensaje genérico al ciudadano. El ciudadano puede reintentar F-CI-01. Caso no esperado en operación normal: si ocurre repetidamente, indica problema sistémico (clave de AUTENTICAR desconocida en el JWKS histórico, compromiso, etc.).
- *Frase secreta no cumple criterios mínimos* (paso 2): la aplicación rechaza el envío con indicación específica de qué criterio no se cumplió. El ciudadano puede volver a intentar.
- *Abandono durante paso 2*: nada se persiste, el emisor no recibe confirmación, el cool-down no se consume. El ciudadano puede reintentar el flujo completo desde F-CI-01.
- *Abandono durante pasos 5 o 6*: el registro queda con los flags pendientes. En el próximo login, F-AP-02 detecta y redirige aquí, mostrando solo lo pendiente. El tutorial se reinicia desde el principio cada vez que se vuelve, hasta completarse.
- *Rechazo de T&C en paso 6*: la aplicación muestra un mensaje breve invitando a volver cuando cambie de opinión (texto exacto es decisión UX) y cierra la sesión (transición a F-AP-03). El registro del ciudadano permanece con `tc_aceptados=false`. Próximo login vuelve a presentar T&C; el ciudadano nunca queda con acceso a la plataforma hasta aceptarlos.
- *M-DEBUG*: si el modo debug está activo, la pantalla del paso 2 (que confirma identidad emitida en la base de la aplicación, evento logueable según P-0020) muestra advertencia prominente y requiere confirmación explícita. UX pendiente.

**Notas**

- El paso 4 (confirmación al emisor) es el cierre del compromiso atómico entre capa de identidad y aplicación destino. La condición que lo dispara es la persistencia exitosa del registro completo de ciudadano en la base de la aplicación. Antes de ese punto, la emisión es reversible sin costo para el ciudadano (cool-down no consumido); después de ese punto, la emisión es definitiva y los pendientes posteriores (tutorial, T&C) son responsabilidad exclusiva de la aplicación destino, sin involucrar al emisor.
- La decisión "tutorial y T&C bloqueantes hasta completarlos / aceptarlos, con re-presentación en cada login pendiente" no está cubierta por ningún ADR vigente. Se registra como hueco H-2 al final del documento.

### F-AP-02 — Login

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0004, P-0005, P-0008, P-0009, P-0020.

**Condición de entrada**

El ciudadano llega desde F-AP-00 eligiendo "ingresar". Posee identidad anónima y frase secreta de un registro previo. No hay sesión activa.

**Condición de salida**

- *Éxito sin pendientes*: la frase secreta verifica contra el registro almacenado y el ciudadano no tiene tutorial ni T&C pendientes. Sesión iniciada; aterriza en la página principal (F-AP-04).
- *Éxito con pendientes*: la frase verifica pero el chequeo de pendientes detecta tutorial o T&C incompletos. Sesión iniciada; redirige a F-AP-01 mostrando solo lo pendiente.
- *Fallo de autenticación*: pseudónimo no existe en la base, o existe pero la frase no verifica. Mensaje genérico de credenciales inválidas (P-0020 oculta el motivo específico en modo operativo). El ciudadano permanece en la pantalla de login y puede reintentar.
- *Bifurcación a pantalla informativa sobre credenciales perdidas*: el ciudadano sigue el link de "olvidé mis credenciales" antes de intentar autenticarse.

**Pasos / pantallas**

1. *Pantalla de login*. Campo para identidad anónima, campo para frase secreta, botón de envío. Link visible "¿Olvidaste tus credenciales?". La pantalla no precarga ninguna identidad usada previamente (P-0005).
2. *Normalización del pseudónimo*. Al enviar, la aplicación normaliza la identidad anónima ingresada según P-0004: insensible a mayúsculas, acentos, espacios y guiones (medios y bajos). Esto ocurre sin que el ciudadano lo perciba.
3. *Verificación de credenciales*. El cliente calcula `HASH(frase_secreta)` con la función estándar de P-0009 y lo envía junto con la identidad anónima normalizada. El servidor busca el registro por identidad, aplica Argon2id con el salt y parámetros almacenados, y compara con el hash guardado.
4. *Chequeo de pendientes*. Si la verificación es exitosa, la aplicación consulta los flags `tutorial_completado` y `tc_aceptados` del registro. Si alguno es `false`, redirige a F-AP-01 con la indicación de qué mostrar. Si ambos son `true`, redirige a la página principal (F-AP-04).

**Bifurcaciones, errores y degradación**

- *Fallo de autenticación* (paso 3): mensaje genérico al ciudadano. En modo operativo no se distingue entre "pseudónimo no existe" y "frase incorrecta" (P-0020). El ciudadano puede reintentar sin límite explícito; la protección contra fuerza bruta se resuelve mediante rate limiting en capa de aplicación, fuera del alcance de este inventario.
- *Sigue link "olvidé mis credenciales"* (paso 1): la aplicación lo lleva a una pantalla informativa que recuerda que las credenciales son únicas, que perderlas implica perder permanentemente el acceso a esa identidad anónima, y que puede solicitar una nueva identidad pasado el período de cool-down. Resolución de H-4. La pantalla no ofrece formularios de recuperación porque no existen por diseño (P-0008, P-0015). Desde esa pantalla el ciudadano puede iniciar F-CI-01 o volver a F-AP-00.
- *M-DEBUG*: si el modo debug está activo, la pantalla del paso 1 muestra advertencia, y el envío de credenciales (paso 3, evento logueable) requiere confirmación explícita.

**Notas**

- La centralización del chequeo de pendientes en F-AP-02 garantiza que ningún ciudadano pueda llegar al resto de la plataforma con tutorial o T&C incompletos, independientemente de cómo terminó su sesión anterior.
- El motivo específico de fallo (pseudónimo inexistente vs. frase incorrecta) sí se registra en logs cuando el modo debug está activo, según P-0020. No es información visible al ciudadano en ningún modo.

### F-AP-03 — Cierre y expiración de sesión

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0005, P-0020.

**Condición de entrada**

Dos disparadores:

1. *Logout explícito*: el ciudadano elige cerrar sesión desde la interfaz, desde cualquier pantalla autenticada.
2. *Expiración por inactividad*: transcurrió el tiempo configurado de inactividad sin acciones del ciudadano (default 1 hora, P-0005).

Adicionalmente, el rechazo de T&C en F-AP-01 produce un cierre de sesión que entra por este flujo.

**Condición de salida**

- Sesión destruida completamente.
- El ciudadano queda en F-AP-00 (pantalla genérica de ingreso). La pantalla no precarga ninguna identidad, no muestra "continuar como X", no preserva ningún rastro visible de la sesión anterior (P-0005).

**Pasos / pantallas**

1. *Disparo del cierre*. Por logout explícito (botón en la interfaz autenticada) o por expiración detectada por la aplicación.
2. *Destrucción de sesión*. La aplicación invalida el estado de sesión en servidor y cliente.
3. *Redirección a F-AP-00*. El ciudadano aterriza en la pantalla pública de ingreso. Si el cierre fue por expiración, la aplicación puede mostrar un mensaje breve informando que la sesión expiró por inactividad (decisión UX); si fue por logout explícito, no se requiere mensaje. Si fue por rechazo de T&C, el mensaje breve previo a la redirección ya se mostró en F-AP-01.

**Bifurcaciones, errores y degradación**

- *M-DEBUG*: el logout explícito es un evento logueable (P-0020) y, con modo debug activo, requiere confirmación explícita. La expiración por inactividad no requiere confirmación (no es una acción del ciudadano).

**Notas**

- El umbral de inactividad de 1 hora es configurable, según lo deja explícito P-0005.
- La política de no mostrar "continuar como [identidad]" refuerza la percepción de anonimato y aplica uniformemente: ni siquiera en el caso de expiración por inactividad la aplicación insinúa que recuerda al ciudadano.

## Bloque 3 — Exploración y consulta de propuestas

### F-AP-04 — Exploración del ranking de propuestas

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0010, P-0019, P-0023.

**Condición de entrada**

El ciudadano queda en la página principal con sesión activa. Llega desde el final exitoso de F-AP-01 (primer aterrizaje tras onboarding), desde el final exitoso de F-AP-02 sin pendientes (login), o desde otros flujos que retornan al listado (cierre de F-AP-06 sin acción, finalización de F-AP-08 con publicación exitosa, etc.).

**Condición de salida**

El flujo F-AP-04 no tiene un final propio. Es el centro de operaciones del ciudadano autenticado y se abandona transicionando a otro flujo:

- A F-AP-05 al usar la barra de búsqueda o los controles de filtrado.
- A F-AP-06 al abrir una propuesta concreta.
- A F-AP-08 al iniciar la creación de una propuesta.
- A F-AP-03 al cerrar sesión.

**Pasos / pantallas**

El flujo en sí consiste en una única pantalla persistente con varios elementos interactivos.

1. *Listado principal de propuestas*. Cada propuesta se muestra como una fila con las columnas definidas en P-0010: Relevancia (score normalizado 0-100 con tooltip), Apoyos (conteo real sin ponderación), íconos 🔥 Tendencia y 🌱 Emergente cuando aplican (con sus respectivos tooltips). El orden por defecto es por Relevancia descendente.
2. *Control de ordenamiento*. El ciudadano puede cambiar el criterio de ordenamiento en cualquier momento (P-0010). Los criterios disponibles incluyen al menos los implícitos en las columnas visibles. La lista exacta de opciones es decisión UX.
3. *Acceso a búsqueda y filtros*. Elementos de UI en la misma pantalla permiten invocar F-AP-05.
4. *Acceso a creación de propuesta*. Botón o elemento equivalente que invoca F-AP-08. La verificación de cupo de P-0017 ocurre al iniciar F-AP-08, no acá.
5. *Identidad y menú del ciudadano*. La interfaz muestra el pseudónimo del ciudadano de forma persistente. La forma concreta y el contenido del menú asociado están pendientes (hueco H-6).

**Bifurcaciones, errores y degradación**

- *M-DEBUG*: si el modo debug está activo, la pantalla muestra la advertencia transversal. Como F-AP-04 no genera eventos logueables propios ni modifica estado, no requiere confirmación adicional.

**Notas**

- F-AP-04 es deliberadamente liviano como flujo. La mayor parte del valor del usuario en esta pantalla se realiza al transicionar a otros flujos. El listado en sí es estado, no acción.
- Si los visitantes no autenticados ven este listado o una variante pública del mismo es decisión no tomada (hueco H-1).

### F-AP-05 — Búsqueda y filtrado de propuestas

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0010, P-0019.

**Condición de entrada**

El ciudadano activa un control de búsqueda o un filtro desde F-AP-04, o desde otro flujo que ofrezca estos controles (por ejemplo, podría invocarse desde otras pantallas si la fase de diseño UX lo decide). Tiene sesión activa cuando aplica el filtro de "propuestas apoyadas por mí" (P-0019 lo establece como restricción para ese filtro específico); el resto de la funcionalidad es independiente del estado de autenticación, dependiente de la resolución de H-1.

**Condición de salida**

- *Listado filtrado mostrado*: el ciudadano permanece en el contexto de búsqueda/filtros con los resultados visibles. Desde acá puede iterar (modificar la búsqueda, agregar o quitar filtros), abrir una propuesta (F-AP-06) o limpiar para volver a F-AP-04.
- *Búsqueda sin resultados*: el listado queda vacío con indicación al ciudadano de que no hubo coincidencias.

**Pasos / pantallas**

1. *Ingreso de criterios*. El ciudadano usa la barra de búsqueda por texto, o activa uno o más filtros estructurados, o ambos. Los filtros estructurados disponibles son los de P-0019: emergente, tendencia, cantidad de vínculos, vínculos a propuestas específicas, rango de fechas, rango de apoyos, propuestas apoyadas por mí / no apoyadas por mí.
2. *Resolución de la consulta*. La aplicación procesa la búsqueda con full-text sobre título y cuerpo, con morfología y stopwords en español, insensible a mayúsculas y acentos, ponderando título sobre cuerpo (P-0019). Los filtros estructurados se combinan en AND con la búsqueda.
3. *Presentación de resultados*. Los resultados se muestran como una lista de propuestas con las mismas columnas que F-AP-04. El orden default puede ser por relevancia textual cuando hay búsqueda activa, o por el mismo criterio que F-AP-04 cuando solo hay filtros.

**Bifurcaciones, errores y degradación**

- *Filtro "apoyadas por mí" sin sesión activa*: el control no está disponible o muestra explicación de que requiere autenticación. La condición de visibilidad concreta depende de la resolución de H-1.
- *M-DEBUG*: sin requisitos especiales. F-AP-05 no genera eventos logueables propios.

**Notas**

- "Propuestas apoyadas por mí" funciona técnicamente como cualquier otro filtro estructurado, sobre el historial de apoyos del propio ciudadano que la aplicación ya tiene asociado a su `anon_id`. Su acceso desde un eventual menú del perfil con la opción "Mis apoyos" (parte de H-6) sería simplemente un atajo que llega a este filtro con el control ya activado.
- Una búsqueda por palabras del texto del tombstone (por ejemplo "removida") puede devolver propuestas retiradas (P-0023). Es ruido aceptado.
- F-AP-05 puede invocarse desde la página principal o desde otros puntos de entrada de UX. El flujo no cambia según el punto de entrada.

### F-AP-06 — Visualización de una propuesta y sus vínculos

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0010, P-0012, P-0017, P-0018, P-0019, P-0023. También `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`.

**Condición de entrada**

El ciudadano accede a una propuesta concreta. Las vías de entrada son: pulsar una fila del listado en F-AP-04 o F-AP-05, seguir un vínculo desde otra propuesta abierta (entrante o saliente), o llegar a la URL de la propuesta de forma directa. Hay sesión activa.

**Condición de salida**

- *Salida pasiva*: el ciudadano vuelve al listado, navega a otra propuesta, o cierra la sesión.
- *Salida hacia acción*: el ciudadano invoca F-AP-07 (apoyar / retirar apoyo) o F-AP-08 con vínculo preseleccionado (crear propuesta derivada que vincula a esta).

**Pasos / pantallas**

1. *Renderizado de la propuesta*. Se muestra el título, el cuerpo en Markdown renderizado, la fecha de publicación, el conteo de apoyos (valor real, P-0012) y eventualmente las señales de ranking si aplican (🔥, 🌱). Los links externos en el cuerpo se renderizan como texto plano no clickeable (P-0018 Dec 4).
2. *Vínculos salientes*. Las propuestas a las que esta propuesta vincula se muestran con suficiente información para que el ciudadano decida si seguirlas (al menos título; el detalle UX es pendiente).
3. *Controles de acción*. Botón de apoyar o retirar apoyo según el estado actual del ciudadano respecto a esta propuesta (invoca F-AP-07). Botón de crear propuesta derivada (invoca F-AP-08 con esta propuesta preseleccionada como vínculo).

**Bifurcaciones, errores y degradación**

- *Propuesta retirada (tombstone)*: la propuesta tiene el contenido del tombstone (título "Propuesta removida por <causal>", cuerpo con el texto largo de la causal, conteo de apoyos en cero, sin vínculos salientes propios pero potencialmente con vínculos entrantes desde otras propuestas, P-0023). La aplicación no tiene un flag dedicado para distinguir tombstones; el ciudadano los identifica por contenido.
- *M-DEBUG*: F-AP-06 en sí no genera eventos logueables (la visualización es una lectura). Los eventos logueables aparecen cuando se invoca F-AP-07 o F-AP-08 desde acá, y la advertencia y confirmación se manejan en esos flujos.

**Notas**

- La propuesta no muestra ningún identificador del autor (P-0001, P-0018).
- La aplicación no distingue propuestas derivadas de propuestas con vínculos cualquiera: a nivel de modelo de datos es lo mismo (`vinculacion_de_propuestas.md`). La presentación visual puede diferenciarlas con etiquetas descriptivas, pero esa es decisión UX y no afecta el flujo.

## Bloque 4 — Participación activa y modificador transversal

### F-AP-07 — Dar / retirar apoyo a propuesta

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0010, P-0012, P-0020.

**Condición de entrada**

El ciudadano está en F-AP-06 visualizando una propuesta concreta, con sesión activa. Pulsa el control de apoyar (si actualmente no apoya esta propuesta) o el control de retirar apoyo (si actualmente la apoya). El estado actual del apoyo del ciudadano respecto a esta propuesta es información que la aplicación ya tiene asociada al `anon_id` autenticado.

**Condición de salida**

- *Apoyo registrado*: el conteo público de apoyos de la propuesta aumenta en uno. El ciudadano vuelve al contexto de F-AP-06 con el estado del control actualizado.
- *Apoyo retirado*: el conteo público disminuye en uno. El ciudadano vuelve al contexto de F-AP-06 con el estado del control actualizado.
- *Cancelación de retiro*: el ciudadano rechaza la confirmación de retiro y vuelve a F-AP-06 sin cambios.

**Pasos / pantallas**

El flujo bifurca en dos ramas según la acción.

**Rama A — Dar apoyo**:

1. *Pulsación del control "apoyar"*. La aplicación registra el apoyo asociado al `anon_id` autenticado y a la propuesta. El conteo público se actualiza.

**Rama B — Retirar apoyo**:

1. *Pulsación del control "retirar apoyo"*.
2. *Confirmación explícita*. La aplicación pregunta al ciudadano si confirma el retiro. Esta confirmación responde a la resolución de H-8: dado que retirar no es una acción frecuente y tiene impacto sobre el conteo público de la propuesta, vale el cuidado adicional.
3. *Aceptación o rechazo*. Si acepta, la aplicación elimina el apoyo del ciudadano sobre la propuesta y actualiza el conteo público. Si rechaza, vuelve a F-AP-06 sin cambios.

**Bifurcaciones, errores y degradación**

- *M-DEBUG*: dar apoyo y retirar apoyo son eventos logueables (P-0020). Con modo debug activo, ambas ramas requieren confirmación explícita. En la rama B la confirmación de retiro (paso 2) y la confirmación del modo debug pueden materializarse como una sola interacción que cumpla ambas funciones; la integración concreta es decisión UX.

**Notas**

- El sistema no expone en ninguna interfaz el concepto de rechazo, oposición o desaprobación (P-0012). El único estado posible del ciudadano respecto a una propuesta es apoyarla o no apoyarla.
- Retirar un apoyo no deja registro visible (P-0012). La interfaz simplemente vuelve al estado "no apoyada" y el conteo disminuye.

### F-AP-08 — Creación de propuesta (original o derivada)

**Componente**: aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0010, P-0011, P-0017, P-0018, P-0020, P-0022. También `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`.

**Condición de entrada**

Dos puntos de entrada equivalentes a nivel de flujo:

1. *Desde F-AP-04*: el ciudadano pulsa el control de crear propuesta. No hay vínculos preseleccionados.
2. *Desde F-AP-06*: el ciudadano pulsa el control de crear propuesta derivada desde una propuesta concreta. El id de esa propuesta queda preseleccionado como vínculo inicial.

Ambos puntos de entrada llevan al mismo flujo, con la única diferencia de cómo arranca el campo de vínculos. A nivel de modelo de datos no hay distinción entre una propuesta derivada y cualquier otra propuesta con vínculos (`vinculacion_de_propuestas.md`).

**Condición de salida**

- *Propuesta publicada*: el revisor de lenguaje aprobó el contenido, los vínculos son válidos, el cupo tenía slots disponibles, la publicación efectiva fue exitosa. La propuesta queda visible con un apoyo del autor registrado automáticamente (P-0012). El ciudadano queda en F-AP-06 sobre su propia propuesta o vuelve a F-AP-04, según decisión UX.
- *Cancelación por cupo agotado*: el chequeo inicial de cupo determinó que el ciudadano no tiene slots disponibles. La pantalla informativa intermedia (ver paso 1) muestra la fecha del próximo slot. El editor nunca se abre. El ciudadano vuelve al flujo desde donde entró.
- *Cancelación por el ciudadano*: el ciudadano abandona la redacción. El draft se pierde (resolución de H-10).
- *Fallo del revisor (servicio externo caído)*: agotados los reintentos server-to-server con backoff (P-0022 Dec 4), la publicación se rechaza con mensaje de fallo transitorio. El draft no se persiste del lado del servidor (P-0022 Dec 4 fail-closed); la responsabilidad de preservar el texto queda del lado del ciudadano.

**Pasos / pantallas**

1. *Chequeo de cupo y eventual mensaje informativo*. Antes de abrir el editor, la aplicación consulta el cupo del ciudadano según P-0017 (año móvil, default 2 propuestas). Si el ciudadano tiene slots disponibles, el flujo avanza al paso 2. Si no tiene slots, la aplicación muestra un mensaje informativo superpuesto con el estado de cupo (slots consumidos y, para cada uno, la fecha en que vuelve a estar disponible). El ciudadano puede cerrar el mensaje y permanece en el contexto desde donde inició el flujo (F-AP-04 o F-AP-06). El editor no se abre. La forma visual concreta del mensaje (modal, banner, otro) es decisión UX.
2. *Recordatorio de buscar antes de crear (solo desde F-AP-04)*. Antes del editor, y únicamente cuando el flujo se inició desde F-AP-04 sin contexto previo de una propuesta concreta, la aplicación muestra el recordatorio descrito en `docs/propuesta/03_Cómo_se_usaría.md`: invitar al ciudadano a verificar si una propuesta similar ya existe, dado que el cupo es limitado. El ciudadano puede continuar a la redacción o cancelar para volver a buscar (transición a F-AP-05). Cuando el flujo se inició desde F-AP-06 (creación de propuesta derivada), este paso se omite: se asume que el ciudadano ya buscó y encontró la propuesta de la que parte.
3. *Editor de propuesta*. Pantalla con dos campos principales: título (texto plano, longitud máxima configurable, default 200 caracteres) y cuerpo (Markdown, longitud máxima configurable, default 20.000 caracteres). P-0018 establece estos campos. La pantalla también muestra el control de vínculos, que arranca con la propuesta preseleccionada (si entró desde F-AP-06) o vacío (si entró desde F-AP-04). El ciudadano puede agregar vínculos ingresando el id de las propuestas a vincular (ver H-12 sobre una variante UX alternativa). El máximo de vínculos por propuesta es configurable (default 10, `vinculacion_de_propuestas.md`).
4. *Envío al revisor de lenguaje*. Al pulsar publicar, la aplicación envía el contenido al revisor según P-0011 (normalización previa contra ofuscación + texto original al modelo). El ciudadano ve una indicación de que la revisión está en curso. La llamada es server-to-server con reintentos y backoff exponencial según P-0022 Dec 4.
5. *Resultado del revisor*.
   - *Aprobación*: el flujo avanza al paso 6.
   - *Rechazo*: la aplicación muestra el mensaje de P-0011 con la categoría detectada expresada en términos ciudadanos. El ciudadano vuelve al editor con su texto preservado y puede corregir y reintentar. No hay límite de intentos (P-0011). El draft sigue vivo en memoria.
   - *Fallo de servicio agotados los reintentos*: mensaje de fallo transitorio según P-0022 Dec 4. El ciudadano no pierde el texto en pantalla mientras la sesión esté activa, pero el sistema no garantiza preservación si el navegador se cierra.
6. *Publicación efectiva*. La aplicación persiste la propuesta con sus campos (id generado, título, cuerpo, fecha de publicación, conteo de apoyos en uno por el autor, score inicial calculado por P-0010, vínculos). Registra también el evento `{anon_id, fecha_publicacion}` (granularidad de día) en la tabla separada de eventos de publicación que P-0018 establece para el control de cupo, sin asociación con qué propuesta fue publicada.
7. *Confirmación al ciudadano*. La aplicación muestra confirmación de publicación exitosa y aterriza al ciudadano en F-AP-06 sobre su propuesta recién publicada, o vuelve a F-AP-04 según decisión UX.

**Bifurcaciones, errores y degradación**

- *Cupo agotado* (paso 1): pantalla informativa intermedia con fecha del próximo slot. El editor no se abre.
- *Vínculo a propuesta inexistente* (paso 3 al publicar, paso 6 al validar): la aplicación rechaza la publicación con indicación específica de qué id no fue resuelto. El ciudadano corrige y reintenta.
- *Rechazo del revisor con corrección* (paso 5): iteración sin límite hasta que el ciudadano corrige o cancela.
- *Fallo del revisor agotados los reintentos* (paso 5): rechazo con mensaje de fallo transitorio. Draft no preservado por el servidor.
- *Cancelación del ciudadano*: en cualquier momento antes del paso 6 el ciudadano puede abandonar; el draft se pierde.
- *M-DEBUG*: con modo debug activo, los pasos 4 (envío al revisor) y 6 (publicación efectiva) son eventos logueables independientes y cada uno requiere confirmación explícita propia. Esta duplicación es deliberada: el modo debug puede activarse en medio del flujo entre un paso y el siguiente, y no puede asumirse que una sola confirmación al inicio cubra eventos logueables posteriores.

**Notas**

- El autor queda automáticamente registrado como un apoyo de su propuesta (P-0012). No hay paso adicional para esto.
- El modelo de datos no distingue propuestas derivadas de propuestas con vínculos cualquiera. El "punto de entrada derivada" desde F-AP-06 es solo una ayuda UX que preselecciona vínculos y puede sugerir un título alusivo. La propuesta resultante es indistinguible internamente de una propuesta con vínculos creada desde F-AP-04.
- La tabla de eventos de publicación que se actualiza en el paso 6 registra solo `{anon_id, fecha_publicacion}`, sin qué propuesta fue publicada (P-0018 Dec 1). Esto preserva la imposibilidad de listar las propuestas de un ciudadano, manteniendo la coherencia con P-0001.

## Expiración de sesión (comportamiento transversal)

P-0005 establece que la sesión de la aplicación destino expira por inactividad, con default de 1 hora configurable. La expiración puede ocurrir en cualquier momento durante cualquier flujo autenticado donde el ciudadano pase tiempo en pantalla.

**Regla general**

Cuando el ciudadano pulsa un control que requiere sesión activa y la aplicación detecta que la sesión expiró, la aplicación transita a F-AP-03 (cierre de sesión) con el comportamiento visible que F-AP-03 define, eventualmente con mensaje breve informando de la expiración (decisión UX).

**Flujos afectados**

Aplica a todos los flujos autenticados de la aplicación destino: F-AP-04, F-AP-05, F-AP-06, F-AP-07, F-AP-08. No aplica a F-AP-00 (sin sesión), F-AP-01 (sesión en creación), F-AP-02 (sesión iniciándose), F-AP-03 (sesión cerrándose) ni a F-CI-01 (fuera del alcance de la sesión de la aplicación destino).

**Caso particularmente sensible: redacción larga en F-AP-08**

El editor de propuesta de F-AP-08 puede tener al ciudadano más de una hora en pantalla redactando sin interactuar con el servidor. Si la detección de actividad es solo del lado del servidor, una redacción extensa puede perderse al intentar publicar. Este caso está identificado como pendiente de resolver. Existen mecanismos del lado del cliente (detección de actividad local con heartbeats al servidor, re-login transparente, timers locales) que la decisión futura puede adoptar.

## M-DEBUG — Modo debug activo (modificador transversal)

**Componente**: aplicable a capa de identidad y aplicación destino.

**ADRs y documentos que lo gobiernan**: P-0020.

**Naturaleza**

M-DEBUG no es un flujo. Es una capa transversal de comportamiento de la interfaz que se activa cuando el operador habilita el modo debug del componente correspondiente. Modifica el comportamiento visible de todos los flujos del componente afectado mientras el modo está activo.

**Reglas de comportamiento**

Cuando el modo debug está activo en un componente:

1. *Advertencia prominente en todas las pantallas*. Toda interfaz del componente muestra una advertencia visible que informa al ciudadano que el modo debug está activo. La advertencia debe ser distinguible y no pasar desapercibida. La forma concreta (banner, ícono, color, ubicación, copy) es decisión UX pendiente.
2. *Confirmación explícita antes de cada acción logueable*. Cada paso de cada flujo del componente que genera un evento logueable (login, publicar propuesta, dar apoyo, retirar apoyo, emitir identidad anónima, aceptar pseudónimo) requiere una confirmación explícita adicional del ciudadano antes de proceder. La confirmación es por evento, no por flujo: un flujo con múltiples eventos logueables tiene múltiples confirmaciones independientes.
3. *Evaluación por paso, no por flujo*. La condición "modo debug activo" se evalúa al momento de ejecutar cada paso logueable, no una sola vez al inicio del flujo. Esto cubre el caso del modo debug activado en medio de un flujo en curso: por ejemplo, el operador activa el modo debug entre el envío al revisor de lenguaje (paso 4 de F-AP-08) y la publicación efectiva (paso 6), y la confirmación del paso 6 sí aplica aunque la del paso 4 no haya ocurrido.

**Aplicación a los flujos del inventario**

| Flujo | Evento logueable que requiere confirmación con modo debug activo |
|---|---|
| F-CI-01 | Aceptación del pseudónimo (paso 8), por materializar la emisión. |
| F-AP-01 | Persistencia de la frase secreta y registro inicial (paso 3). |
| F-AP-02 | Envío de credenciales (paso 3). |
| F-AP-03 | Logout explícito. La expiración por inactividad no requiere confirmación (no es acción del ciudadano). |
| F-AP-04 | Ninguno. F-AP-04 no genera eventos logueables propios. |
| F-AP-05 | Ninguno. F-AP-05 no genera eventos logueables propios. |
| F-AP-06 | Ninguno propio. Los eventos surgen al transitar a F-AP-07 o F-AP-08. |
| F-AP-07 | Dar apoyo (rama A paso 1) y retirar apoyo (rama B paso 3). En la rama B, la confirmación de retiro de H-8 y la confirmación del modo debug pueden materializarse como una sola interacción que cumpla ambas funciones. |
| F-AP-08 | Envío al revisor (paso 4) y publicación efectiva (paso 6), de forma independiente. |

**Estado de diseño**

El diseño visual concreto y el flujo de confirmación de M-DEBUG es decisión UX pendiente, registrada como hueco H-3. La especificación corresponde a la fase de diseño de interfaces y debe implementarse en cualquier despliegue del sistema según P-0020.

## Huecos de UX detectados

A medida que se recorrió el sistema se identificaron las siguientes preguntas de diseño que ningún ADR resolvió. Para cada hueco se indica su estado: *abierto* cuando no se decidió respuesta en el inventario; *decidido* cuando el inventario adoptó una respuesta concreta que conviene formalizar más adelante. Los huecos *decididos* describen la respuesta adoptada para que la fase de diseño de interfaces los implemente; los *abiertos* se elevan para discusión.

- **H-1 (abierto) — Visibilidad pública del listado de propuestas para visitantes no autenticados**. P-0019 implica que el filtro "apoyadas por mí" requiere sesión activa, pero el resto no lo requiere explícitamente. P-0012 dice que el conteo de apoyos es público. La narrativa de `docs/propuesta/03_Cómo_se_usaría.md` muestra al ciudadano accediendo a la página principal después del tutorial, no antes. No hay decisión sobre si un visitante no autenticado ve propuestas o no, ni en qué pantalla. Afecta el alcance de F-AP-00, F-AP-04, F-AP-05 y F-AP-06.
- **H-2 (decidido) — Orden y persistencia de tutorial y T&C en el onboarding**. Decisión adoptada en F-AP-01: tutorial bloqueante hasta completarlo; si el ciudadano abandona y vuelve, se reinicia desde el principio. Una vez completado, presentación de T&C bloqueante hasta aceptarlos. Rechazo de T&C muestra mensaje breve invitando a volver y cierra sesión (transición a F-AP-03); el registro queda con `tc_aceptados=false` y en próximos logins F-AP-02 redirige nuevamente a la presentación de T&C. El ciudadano no accede al resto de la plataforma hasta completar ambos. Conviene formalizar esta lógica en un ADR.
- **H-3 (abierto) — Diseño visual y mecánica concreta de M-DEBUG**. P-0020 establece el comportamiento esperado pero deja la especificación visual y de flujo a la capa de UX. Aplica transversalmente.
- **H-4 (abierto) — Pantalla informativa "olvidaste tus credenciales"**. F-AP-02 incluye un link visible a una pantalla informativa que recuerda la irrecuperabilidad y el cool-down. El contenido concreto, tono y eventuales acciones desde esa pantalla son decisión UX.
- **H-5 (abierto) — Mensaje y formato del rechazo por cool-down vigente en F-CI-01**. El paso de verificación de cool-down en F-CI-01 muestra al ciudadano la fecha del próximo slot. La presentación visual es decisión UX.
- **H-6 (abierto) — Presencia persistente del pseudónimo y menú asociado**. El pseudónimo es visible para el propio ciudadano (P-0001). La forma concreta (avatar temático, ubicación en la interfaz, menú desplegable con cupo y apoyos, acceso a logout) y el contenido del eventual menú son decisión UX. Aplica especialmente a F-AP-04 y a cualquier pantalla autenticada.
- **H-7 (abierto) — Redacción y diseño visual de los mensajes de P-0022 al ciudadano**. P-0022 establece tres categorías de mensajes ante fallos del verificador y categorías análogas para otros componentes, pero deja la redacción exacta y la realización visual a la capa de UX.
- **H-8 (decidido) — Confirmación al retirar apoyo**. Decisión adoptada en F-AP-07: retirar apoyo requiere confirmación explícita; dar apoyo no la requiere. La forma visual de la confirmación es decisión UX, integrable con M-DEBUG cuando aplique.
- **H-9 (decidido) — Mecánica de aceptación de pseudónimo en F-CI-01**. Decisión adoptada: una sola sugerencia visible por vez, con botón para generar otra. Iteración sin límite hasta que el ciudadano acepta.
- **H-10 (decidido) — Tratamiento del draft de propuesta durante la corrección iterativa**. Decisión adoptada en F-AP-08: el draft vive en memoria del navegador durante el ciclo de corrección. Se persiste en el backend solo al pasar el revisor. Si el navegador se cierra, el draft se pierde.
- **H-11 (decidido) — Manejo del anon_id con T&C no aceptados**. Originalmente reservado como hueco separado; resuelto e integrado en H-2.
- **H-12 (abierto) — UX alternativa para la vinculación en F-AP-08**. La decisión adoptada en F-AP-08 es ingreso del id de la propuesta a vincular. Se mencionó como alternativa una integración con el buscador principal (F-AP-05) que permita "vincular esta propuesta" como acción desde el listado de resultados, retornando al editor de F-AP-08. Esta variante implicaría agregar una condición de salida nueva a F-AP-05 y modificar el editor de F-AP-08. Queda como decisión UX a evaluar en la fase de diseño de interfaces.
