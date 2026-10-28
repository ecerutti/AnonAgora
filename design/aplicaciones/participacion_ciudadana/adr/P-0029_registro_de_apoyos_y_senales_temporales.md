# P-0029 — Registro de apoyos desvinculado de la identidad y señales temporales del ranking

**Estado:** Activo

## Contexto

El mecanismo de apoyo (P-0012) y el ranking (P-0010) imponen dos requisitos sobre el registro de apoyos que ningún documento del proyecto había especificado:

- **Saber quién apoyó qué.** El ciudadano debe poder retirar su apoyo (P-0012), el sistema debe impedir que apoye dos veces la misma propuesta, y el filtro "apoyadas / no apoyadas por mí" (P-0019) opera sobre su historial de apoyos.
- **Saber cuándo ocurrieron los apoyos.** La señal 🔥 tendencia (P-0010) se define por el crecimiento de apoyos de cada propuesta en las últimas 48 horas, lo que exige conocer el momento de cada apoyo o, al menos, la evolución temporal del conteo.

P-0018 establece que el registro de apoyos individuales es la fuente de verdad del conteo, pero no define qué campos tiene. P-0020 no registra los apoyos en los logs. La forma más directa de cumplir ambos requisitos —guardar cada apoyo con el `anon_id` y su fecha— crea una superficie de correlación que el resto del diseño evita:

- **Línea de tiempo por identidad.** Cada `anon_id` quedaría asociado a los momentos en que estuvo activo. Con poca actividad en la plataforma (por ejemplo, en sus primeros meses), esos momentos pueden cruzarse con los eventos de emisión del emisor o con los registros del verificador externo de identidad, que sí conoce el CUIL, y acercar la identidad anónima a la real.
- **Identificación del autor.** El autor de una propuesta queda registrado como su primer apoyo en el momento de publicarla (P-0012). Un apoyo con la misma hora que la `fecha_publicacion` de la propuesta revela qué `anon_id` la escribió, anulando la desasociación entre propuesta y autor de P-0018 (Decisión 1).
- **Backups.** Todo lo que se guarda en la base queda en sus backups durante el período de retención del despliegue. Comparar backups sucesivos revela qué `anon_id` agregó qué apoyo en cada intervalo, aunque la tabla no guarde fecha.

Para evaluar las alternativas conviene distinguir tres atacantes, dentro del modelo intermedio de P-0006:

- **Quien obtiene una copia de la base** (filtración, un backup o una serie de backups), sin acceso a la configuración del despliegue.
- **El operador**, o quien obtenga la base junto con la configuración del despliegue (variables de entorno, secretos). El diseño trata al operador como potencialmente malicioso (P-0006, P-0014) y prefiere que sus límites sean imposibles por diseño antes que restricciones auditables (`design/modelo_operativo.md`).
- **Un observador del servidor en vivo**, que ve la memoria y el tráfico del proceso. Es un compromiso completo de la infraestructura, fuera del alcance del modelo de amenazas (P-0006).

Hay además una observación que ordena todo el análisis: **el conteo de apoyos de cada propuesta es público** (P-0010, P-0012). Cualquiera que consulte el listado periódicamente puede reconstruir la evolución temporal del conteo de cada propuesta. Lo sensible no es el momento de los apoyos, sino la unión entre ese momento y la identidad que apoyó.

Por último, P-0010 deja sin valor concreto el "percentil superior" que define 🔥, y no prevé qué ocurre cuando la actividad de la plataforma es tan baja que un crecimiento mínimo ya resulta "inusual".

Las preguntas de diseño que motivaron este ADR son:

- ¿Qué guarda cada registro de apoyo?
- ¿Cómo se identifica al ciudadano que apoyó, sin dejar en la base un vínculo entre su identidad y sus apoyos?
- ¿Cómo se calcula 🔥 sin abrir correlaciones temporales, en especial con poca actividad?
- ¿Qué valores concretos tienen los umbrales de 🔥 y 🌱?
- ¿Qué identificadores del ciudadano pueden aparecer en los logs de la aplicación?

## Opciones consideradas

### Decisión 1 — Registro de apoyos y de su momento

#### Opción A — Apoyo con `anon_id` y fecha

Cada apoyo se guarda como `{anon_id, propuesta_id, fecha}`, con la fecha truncada a la hora o al día.

Ventajas

- La implementación más simple: 🔥 se calcula contando filas recientes.

Desventajas

- Deja en la base una línea de tiempo de actividad por `anon_id`, expuesta a cualquiera de los tres atacantes.
- El apoyo del autor, con la hora de publicación, identifica al autor de cada propuesta.
- Con poca actividad, la línea de tiempo es altamente identificante al cruzarla con registros externos.

#### Opción B — Apoyo con `anon_id` sin fecha, más historial agregado persistido

Cada apoyo se guarda como `{anon_id, propuesta_id}`, sin fecha. Una tabla separada registra la evolución del conteo de cada propuesta por franjas horarias, sin `anon_id`.

Ventajas

- La tabla de apoyos no contiene fechas.
- El historial agregado no agrega información que no sea pública.

Desventajas

- Cuando una propuesta tiene pocos apoyantes, cada incremento del historial se atribuye a un conjunto pequeño de `anon_id` (los que figuran como apoyantes), y el momento del apoyo queda asociado a la identidad.
- La serie de backups conserva el historial y permite el análisis retrospectivo durante todo el período de retención.
- Comparar backups sucesivos sigue revelando qué `anon_id` agregó qué apoyo en cada intervalo. El orden físico de las filas también delata el orden de inserción.

#### Opción C — Apoyo con `anon_id` sin fecha, más historial agregado solo en memoria

Igual que la Opción B, pero el historial agregado vive en la memoria del proceso de la aplicación y nunca se persiste.

Ventajas

- El historial no llega a la base ni a los backups.

Desventajas

- Comparar backups sucesivos sigue revelando qué `anon_id` agregó qué apoyo en cada intervalo.
- Un reinicio de la aplicación borra el historial: 🔥 no se muestra hasta que se reconstruye la ventana de 48 horas.
- No funciona con más de una instancia de la aplicación sin un almacenamiento compartido en memoria.

#### Opción D — Apoyo desvinculado mediante una etiqueta con secreto del servidor

Cada apoyo se guarda como `{etiqueta, propuesta_id, hora}`, donde la etiqueta es un HMAC de la propuesta calculado con una clave derivada de un secreto del despliegue y del `anon_id`.

Ventajas

- Una copia de la base sin el secreto no permite vincular los apoyos con ninguna identidad, ni los apoyos entre sí.
- No depende de la frase secreta.

Desventajas

- Quien tenga el secreto (el operador, o quien obtenga la configuración) puede recalcular las etiquetas de todos los `anon_id` contra todas las propuestas y reconstruir quién apoyó qué y cuándo.
- Una filtración futura del secreto desbloquea retroactivamente todos los backups vigentes.
- Convierte la protección en una restricción auditable que depende de un secreto que hay que custodiar indefinidamente, en lugar de una imposibilidad por diseño.

#### Opción E — Apoyo desvinculado mediante una etiqueta con clave derivada de la frase secreta

Cada apoyo se guarda como `{etiqueta, propuesta_id, hora}`, donde la etiqueta es un HMAC de la propuesta calculado con una **clave de apoyos** propia de cada ciudadano. La clave se deriva de la frase secreta durante la verificación del login, vive solo en la sesión y nunca se almacena.

Ventajas

- Nadie puede vincular los apoyos con una identidad sin la frase de ese ciudadano: ni quien tenga una copia de la base, ni quien tenga además la configuración del despliegue, ni el operador.
- Las filas tampoco son vinculables entre sí: dos apoyos del mismo ciudadano a propuestas distintas tienen etiquetas no relacionadas.
- Como las filas no se pueden atribuir, guardar la hora de cada apoyo no expone más que el conteo público, y 🔥 se calcula directamente sobre la tabla.
- Para el ciudadano no cambia nada: la frase ya se ingresa en el login (P-0004) y todas las acciones sobre apoyos ya requieren sesión.

Desventajas

- Solo se puede operar sobre los apoyos de un ciudadano mientras tiene una sesión activa.
- Cambiar los parámetros de Argon2id de un registro (re-hasheo) cambia la clave de apoyos y obliga a recalcular las etiquetas de ese ciudadano. Lo mismo ocurriría con un eventual cambio de frase, que hoy no existe (P-0008).
- Es diseño criptográfico propio que requiere revisión cuidadosa.

#### Opción F — Como la Opción E, con clave de apoyos aleatoria cifrada con la frase

La clave de apoyos se genera al azar en el registro y se guarda cifrada con una clave derivada de la frase. Un cambio de frase solo requiere volver a cifrar esa clave.

Ventajas

- Las mismas propiedades de seguridad que la Opción E.
- Un eventual cambio de frase o de parámetros no obliga a recalcular las etiquetas.

Desventajas

- Agrega un dato almacenado por identidad y una operación de cifrado, para resolver un cambio de frase que la plataforma no contempla (P-0008).

#### Opción G — Historial agregado con ruido estadístico

El historial agregado de la Opción B se perturba con ruido aleatorio (privacidad diferencial) para impedir atribuir incrementos individuales.

Ventajas

- Técnica reconocida para publicar estadísticas sin revelar contribuciones individuales.

Desventajas

- No protege nada: el conteo exacto es público y cualquiera puede reconstruir el historial sin ruido.
- Con poca actividad, el ruido necesario destruye la señal que 🔥 intenta mostrar.

### Decisión 2 — Derivación de la clave de apoyos

Si se adopta la Opción E de la Decisión 1, la pregunta es cómo se obtiene la clave de apoyos a partir de la frase sin debilitar su almacenamiento (P-0009).

#### Opción A — Salida extendida del mismo Argon2id

El Argon2id del login (P-0009) produce 64 bytes en lugar de 32. Los primeros 32 son el valor que se almacena y se compara, como hoy. Los 32 restantes nunca se almacenan y, pasados por HKDF con una etiqueta de propósito propia, dan la clave de apoyos.

Ventajas

- Un solo cálculo de Argon2id por login: sin costo adicional.
- El valor almacenado sigue siendo el resultado de Argon2id, como decide P-0009.
- Conocer la parte almacenada no permite calcular la clave de apoyos.

Desventajas

- Acopla la clave de apoyos a los parámetros de Argon2id de cada registro.

#### Opción B — HKDF sobre la salida completa

Se toma la salida de Argon2id completa y de ella se derivan con HKDF, con etiquetas distintas, el verificador almacenado y la clave de apoyos.

Ventajas

- Separación de claves canónica.

Desventajas

- Lo almacenado deja de ser el resultado de Argon2id y pasa a ser un derivado, lo que modifica lo decidido en P-0009 sin una ganancia de seguridad relevante sobre la Opción A.

#### Opción C — Segundo Argon2id independiente

La clave de apoyos se calcula con un Argon2id separado, con su propio salt y parámetros fijos.

Ventajas

- La clave de apoyos no cambia cuando se re-hashea el verificador.

Desventajas

- Duplica el costo de cada login.

### Decisión 3 — Definición operativa de 🔥 y 🌱

P-0010 define 🔥 como un crecimiento de apoyos en el "percentil superior" de la plataforma en las últimas 48 horas, y 🌱 como un ritmo de crecimiento "por encima del promedio" para propuestas de menos de 7 días, sin fijar el percentil ni prever el caso de actividad muy baja. Con actividad muy baja, además, una propuesta recién publicada puede superar el promedio que define 🌱 solo con el apoyo automático de su autor (P-0012).

#### Opción A — Percentil sin umbral mínimo

🔥 se asigna a las propuestas cuyo crecimiento en la ventana está en el percentil fijado, cualquiera sea la actividad total.

Ventajas

- Totalmente adaptativo, como pide P-0010.

Desventajas

- Con muy poca actividad, el percentil cae a cero o a valores mínimos y un solo apoyo basta para mostrar 🔥; del mismo modo, el apoyo del autor basta para mostrar 🌱. Las señales pierden significado justamente cuando la plataforma es más pequeña.

#### Opción B — Percentil más umbral mínimo de actividad de la plataforma

🔥 se asigna por percentil y 🌱 por promedio, pero ambas señales se muestran solo cuando la actividad total de la plataforma en la ventana de tendencia alcanza un mínimo configurable. Por debajo de ese mínimo, ninguna propuesta muestra 🔥 ni 🌱.

Ventajas

- Conserva el carácter adaptativo de P-0010 cuando hay actividad suficiente.
- Evita mostrar como "inusual" un crecimiento insignificante.

Desventajas

- Introduce un parámetro absoluto que el operador debe calibrar según la escala del despliegue.

#### Opción C — Umbrales absolutos

🔥 se asigna a partir de un número fijo de apoyos nuevos en la ventana.

Desventajas

- Contradice el criterio de P-0010: un valor absoluto pierde sentido a medida que la plataforma crece o decrece.

### Decisión 4 — Identificadores del ciudadano en los logs de la aplicación

P-0020 prohíbe el `anon_id` en los logs del emisor pero lo considera un identificador operativo legítimo de la aplicación destino, porque analiza la correlación **entre** componentes. Registra además el inicio y cierre de sesión con granularidad de hora. Con la hora guardada en cada apoyo (Decisión 1), aparece una correlación **dentro** de la aplicación que P-0020 no contemplaba: si un log dice que una identidad tuvo una sesión en una hora determinada y la tabla de apoyos muestra apoyos en esa misma hora, con poca actividad esos apoyos se atribuyen a esa identidad.

#### Opción A — Mantener P-0020

El `anon_id` puede figurar en los logs de la aplicación.

Ventajas

- Permite rastrear en los logs los eventos de una identidad concreta.

Desventajas

- Reabre por los logs la atribución de apoyos que la Decisión 1 cierra en la base, durante todo el período de retención de los logs.

#### Opción B — Prohibir los identificadores del ciudadano en los logs de la aplicación

Ni el `anon_id` ni el pseudónimo aparecen en los logs de la aplicación, en ningún modo de operación.

Ventajas

- Ningún registro de la aplicación asocia una identidad con un momento.
- Ninguno de los eventos que P-0020 registra necesita identificar al ciudadano para cumplir su función de diagnóstico.

Desventajas

- No es posible rastrear en los logs los eventos de un ciudadano concreto. La aplicación no tiene un canal de soporte en el que un ciudadano se identifique, así que ese rastreo no tiene un uso legítimo previsto.

## Decisión

**Decisión 1.** Se adopta la Opción E: cada apoyo se guarda como `{etiqueta, propuesta_id, hora}`, sin `anon_id`.

- `etiqueta = HMAC-SHA-256(clave_de_apoyos, "apoyo-v1:" + propuesta_id)`. Es única por ciudadano y propuesta, de modo que la restricción de unicidad sobre la etiqueta impide apoyar dos veces la misma propuesta.
- `hora` es el momento del apoyo truncado a la hora.
- La tabla no tiene identificadores secuenciales ni ningún otro campo.
- La clave de apoyos se obtiene al verificar la frase en el login (F-AP-02) y al definirla en el registro (F-AP-01). Vive solo en la sesión del lado del servidor, nunca se almacena y se descarta al terminar la sesión (P-0005, P-0027).
- Apoyar inserta la fila; retirar el apoyo la elimina. El apoyo automático del autor (P-0012) se registra igual, dentro de la sesión en la que publica. Saber si el ciudadano apoyó una propuesta, o filtrar "apoyadas / no apoyadas por mí" (P-0019), consiste en calcular las etiquetas de las propuestas consideradas y buscarlas.
- El conteo de apoyos de una propuesta es la cantidad de filas con su `propuesta_id` (P-0018). El retiro por causal legal elimina las filas de la propuesta retirada (P-0023).

**Decisión 2.** Se adopta la Opción A. El Argon2id de P-0009 se calcula con una salida de 64 bytes, con el mismo salt y los mismos parámetros almacenados en el registro. Los primeros 32 bytes son el valor que se almacena y se compara en el login. Los 32 restantes se pasan por `HKDF-SHA-256` con la etiqueta de propósito `"clave-de-apoyos-v1"` y el resultado es la clave de apoyos.

Se establece como invariante que ningún dato almacenado permite verificar una conjetura sobre la frase con un costo menor que un cálculo de Argon2id.

Cuando un registro se re-hashea con parámetros nuevos, el sistema recalcula las etiquetas de los apoyos de ese ciudadano en el mismo login, mientras dispone de la clave anterior y de la nueva: para cada propuesta calcula la etiqueta anterior y, si existe, la reemplaza por la nueva. El hash nuevo y las etiquetas recalculadas se guardan en una única transacción.

**Decisión 3.** Se adopta la Opción B.

- Los **apoyos nuevos** de una propuesta en la ventana de tendencia son las filas vigentes cuya `hora` cae dentro de la ventana. Un apoyo retirado deja de contar porque su fila se elimina.
- Una propuesta muestra **🔥** cuando tiene al menos un apoyo nuevo en la ventana y su cantidad de apoyos nuevos alcanza el percentil `percentil_tendencia` de los apoyos nuevos de todas las propuestas.
- Si la suma de apoyos nuevos en la ventana de tendencia de toda la plataforma es menor que `minimo_actividad_senales`, ninguna propuesta muestra 🔥 ni 🌱.
- Una propuesta muestra **🌱** cuando fue publicada hace menos de `ventana_emergente` y su ritmo, `apoyos / (edad_en_dias + 1)` con la misma edad que usa la fórmula de P-0010, supera el promedio de ese ritmo sobre todas las propuestas. Esta definición sigue a P-0010 ("por encima del promedio") y no requiere la hora de los apoyos.
- Las propuestas retiradas participan de estos cálculos como cualquier otra, con apoyos en cero (P-0023).

| Parámetro | Descripción | Valor por defecto |
|---|---|---|
| `percentil_tendencia` | Percentil de apoyos nuevos a partir del cual una propuesta muestra 🔥 | `90` |
| `minimo_actividad_senales` | Apoyos nuevos totales de la plataforma en la ventana de tendencia por debajo de los cuales no se muestran 🔥 ni 🌱 | `20` |

**Decisión 4.** Se adopta la Opción B. Se agregan a los campos prohibidos en los logs de la aplicación, en todos los modos de operación, el `anon_id` y el pseudónimo. La clave de apoyos y las etiquetas ya están prohibidas por P-0020 como derivados de la frase secreta. Esta decisión modifica parcialmente P-0020 (Decisión 2, campos prohibidos de la plataforma).

## Justificación

Lo que el diseño necesita proteger no es el momento de los apoyos, que el conteo público ya expone, sino la unión entre ese momento y la identidad que apoyó. Las opciones que mantienen el `anon_id` en la tabla de apoyos (A, B, C) dejan esa unión en la base de alguna forma: directamente (A), por inferencia cuando hay pocos apoyantes (B) o por comparación de backups (B y C). La protección que dan es, en el mejor caso, de granularidad diaria, y con poca actividad un día alcanza para identificar.

Las etiquetas eliminan esa unión de la base. Entre las dos formas de calcularlas, la Opción D protege frente a una copia de la base pero no frente a quien tenga el secreto del despliegue, que es exactamente el actor que el modelo de amenazas no exime de sospecha; además, una filtración futura del secreto alcanzaría retroactivamente a todos los backups. La Opción E traslada la protección a la frase de cada ciudadano, que el sistema no conserva: vincular apoyos e identidades pasa a ser imposible por diseño, la misma clase de garantía que P-0015 aplica a la separación entre identidad real y `anon_id`.

Con las filas desvinculadas, guardar la hora de cada apoyo no expone más que el conteo público, y 🔥 se calcula sobre la propia tabla de apoyos, sin historiales adicionales y sin perderse ante un reinicio. La Decisión 4 cierra el único cruce que quedaba: los logs de sesión con identidad y hora.

La Opción F resuelve un cambio de frase que la plataforma no contempla (P-0008) y que, de decidirse en el futuro, se puede resolver con el mismo procedimiento de recálculo de etiquetas que la Decisión 2 establece para el re-hasheo. La Opción G no aporta protección porque el dato que perturba ya es público.

La salida extendida de Argon2id (Decisión 2, Opción A) obtiene la clave sin costo adicional y sin modificar lo que P-0009 decide almacenar; la etiqueta de propósito en HKDF separa la clave de apoyos de cualquier otro uso futuro del mismo material. El recálculo de etiquetas al re-hashear preserva la práctica de migrar gradualmente los registros a parámetros más fuertes, en lugar de dejar las identidades antiguas con parámetros débiles.

El umbral mínimo de actividad (Decisión 3) protege el significado de ambas señales: en una plataforma con muy poca actividad, cualquier apoyo supera el percentil de 🔥 y el solo apoyo del autor supera el promedio de 🌱, de modo que las señales dejan de distinguir algo. Un único umbral para las dos evita sumar parámetros. El percentil 90 asigna 🔥 aproximadamente a la décima parte más activa de las propuestas cuando hay actividad suficiente.

## Consecuencias

- Ningún actor puede determinar qué propuestas apoya una identidad sin la frase de ese ciudadano: ni quien obtenga la base o sus backups, ni quien obtenga además la configuración del despliegue, ni el operador. Un requerimiento legal que pida esa información no puede cumplirse; esto se suma a las capacidades y limitaciones del operador enumeradas en P-0023.
- Todas las operaciones sobre los apoyos de un ciudadano requieren una sesión activa, lo que ya ocurría.
- Si un ciudadano pierde su frase, sus apoyos siguen contando y nadie puede retirarlos, igual que antes de esta decisión.
- El re-hasheo de un registro recalcula las etiquetas de ese ciudadano: una operación HMAC por propuesta existente, dentro del login. Un eventual cambio de frase requeriría el mismo procedimiento y su propio ADR, porque reabre P-0008.
- El filtro "apoyadas / no apoyadas por mí" calcula una etiqueta por propuesta considerada; el costo es despreciable a la escala prevista.
- Comparar backups o leer el orden físico de las filas revela cuándo hubo apoyos, pero no de quién: la misma información que el conteo público.
- Un observador del servidor en vivo puede obtener las claves de apoyos de las sesiones activas en ese momento. Es un compromiso de infraestructura fuera del alcance de P-0006.
- Como consecuencia de la Decisión 1, el cruce entre los eventos de publicación `{anon_id, día}` de P-0018 y los apoyos de cada propuesta ya no es posible, porque los apoyos no contienen el `anon_id`. Sigue abierta la correlación entre un evento de publicación y una propuesta cuando en un día se publica una sola, que se analiza por separado.
- La implementación debe verificar con tests que el `anon_id`, el pseudónimo, la clave de apoyos y las etiquetas no aparezcan en los paths de logging de la aplicación (extensión de la consecuencia equivalente de P-0020).
- Los parámetros `percentil_tendencia` y `minimo_actividad_senales` se documentan en la guía de instalación y operación junto con los de P-0010.
- El glosario debe incorporar los términos "clave de apoyos" y "etiqueta de apoyo", y alinear la definición de 🌱 con P-0010 (umbral por promedio, no por percentil).

## Referencias

- P-0004 — Autenticación de identidad anónima
- P-0005 — Gestión de sesiones de usuario
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0008 — Mecanismo de credencial de acceso: passphrase vs password
- P-0009 — Recepción y almacenamiento de la frase secreta en la plataforma
- P-0010 — Ranking y visualización de propuestas
- P-0012 — Mecanismo de apoyo a propuestas
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0018 — Modelo de datos de propuestas
- P-0019 — Búsqueda y filtrado de propuestas
- P-0020 — Política de logs y retención de metadatos
- P-0023 — Moderación de contenido y retiro de propuestas
- P-0027 — Actividad y expiración de la sesión
- `design/aplicaciones/participacion_ciudadana/protocolo_frase_secreta.md`
- `design/modelo_operativo.md`
- `design/threat_model.md`
- `notas/inventario_de_flujos_ui_ux.md` — F-AP-01, F-AP-02, F-AP-07
- `notas/propuesta_guia_de_instalacion.md` — semántica del cambio de parámetros de Argon2id
