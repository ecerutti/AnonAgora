# DI-0003 — Datos iniciales de la demo

**Estado:** Activo

## Contexto

La demo se publica en un sitio accesible por cualquier persona (DP-0002). Un visitante que llega a una plataforma vacía no puede apreciar el ranking con decaimiento temporal (P-0010), las señales 🔥 tendencia y 🌱 emergente, la búsqueda con sus siete filtros (P-0019), los vínculos y las propuestas derivadas (`vinculacion_de_propuestas.md`) ni el tombstone de una propuesta retirada (P-0023). La demo necesita entonces un conjunto de datos iniciales que ejercite todas esas funciones.

Lo ya decidido condiciona esos datos:

- DP-0002 (Decisión 1) exige que los datos iniciales incluyan al menos una propuesta retirada y que la demo use los valores por defecto de producción de los parámetros: G = 1.0, MT = 2.0, ME = 1.5, ventanas de 48 horas para 🔥 y de 7 días para 🌱 (P-0010).
- DP-0002 (Decisión 3) descarta los reinicios periódicos: las identidades y propuestas de los usuarios de la demo persisten.
- DI-0002 establece que los datos de la base son ficticios, porque sus backups se retienen 30 días.
- P-0018 establece que el registro de apoyos individuales es la fuente de verdad del conteo: los apoyos iniciales deben existir como registros, no como contadores inflados.
- P-0029 define cada apoyo como `{etiqueta, propuesta_id, hora}`, sin `anon_id`, y apaga 🔥 y 🌱 mientras la plataforma tenga menos de 20 apoyos nuevos en la ventana de tendencia.
- P-0023 define el tombstone: la propuesta retirada conserva solo su `id`, su fecha pasa a ser la del retiro, sus apoyos y vínculos salientes se eliminan y los vínculos entrantes se mantienen.

Las señales temporales plantean un problema propio de una demo con poco tráfico: un conjunto de datos cargado una sola vez envejece. A las 48 horas sin visitas no queda ningún apoyo nuevo en la ventana de tendencia, la actividad cae bajo el mínimo de P-0029 y desaparecen 🔥 y 🌱; a los 7 días ninguna propuesta del conjunto es emergente.

Las preguntas que motivaron este ADR son:

- ¿De dónde salen los apoyos de los datos iniciales?
- ¿Cómo se fechan los datos y cómo se evita que la demo se apague con el tiempo?
- ¿Cómo se cargan sin romper las reglas del modelo?
- ¿Cómo se informa al visitante que el contenido es ficticio?

La composición concreta del conjunto (propuestas, apoyos, vínculos, señales esperadas) se registra en `demo/implementation/aplicaciones/participacion_ciudadana/datos_iniciales.md`.

## Opciones consideradas

### Decisión 1 — Origen de los apoyos ficticios

#### Opción A — Identidades sintéticas completas

El seed crea identidades anónimas ficticias en la aplicación, con pseudónimo y una credencial inutilizable, y cada una apoya propuestas.

Ventajas

- Cada apoyo corresponde a una identidad registrada, como en el uso real.

Desventajas

- Con P-0029 los apoyos no referencian identidades, así que las identidades sintéticas no aportan nada al modelo de apoyos.
- Ocupa el espacio de pseudónimos (P-0025) y la tabla de identidades con registros que nadie puede usar.
- Para que las identidades sintéticas pudieran volver a apoyar, el seed tendría que conservar sus claves de apoyos.

#### Opción B — Apoyantes ficticios efímeros

Cada apoyante ficticio es una clave de apoyos generada al azar: el seed calcula con ella la etiqueta de P-0029, inserta el apoyo y descarta la clave. No se crean identidades.

Ventajas

- Los apoyos tienen exactamente la forma de los apoyos reales y cumplen P-0018 y P-0029.
- No crea registros de identidad, pseudónimos ni credenciales.
- Ninguna clave queda almacenada: los apoyos ficticios son tan imposibles de atribuir como los reales.

Desventajas

- Las propuestas del seed no tienen un autor que pueda iniciar sesión; ningún visitante verá una propuesta inicial como "apoyada por mí".

#### Opción C — Solo contadores

El seed fija `conteo_apoyos` de cada propuesta sin crear registros de apoyo.

Desventajas

- Contradice P-0018: el registro de apoyos es la fuente de verdad, y una reconciliación dejaría los conteos en cero.
- 🔥 no puede calcularse, porque requiere la hora de cada apoyo (P-0029).

### Decisión 2 — Fechas y envejecimiento de los datos

#### Opción A — Fechas absolutas

Los datos traen fechas fijas en el calendario.

Desventajas

- Según el momento en que se carguen, las propuestas pueden quedar todas viejas o con fechas futuras. Envejecen igual que en la Opción B.

#### Opción B — Fechas relativas al momento de la carga, carga única

Cada fecha se expresa como antigüedad respecto del momento en que se ejecuta el seed.

Ventajas

- Recién cargada, la demo muestra todas las señales.

Desventajas

- Envejece: sin visitas, a las 48 horas desaparecen 🔥 y 🌱.

#### Opción C — Fechas relativas más oleadas de actividad a pedido del operador

Como la Opción B, más un comando de operador que agrega una oleada de actividad ficticia: apoyos de apoyantes efímeros con hora dentro de la ventana de tendencia y, opcionalmente, alguna propuesta nueva de una reserva.

Ventajas

- El operador puede devolver la demo a un estado representativo cuando lo necesite, por ejemplo antes de una presentación.
- No borra nada: las identidades y propuestas de los usuarios se conservan (DP-0002, Decisión 3).
- Es una operación explícita y puntual del operador por línea de comando, coherente con P-0024.

Desventajas

- Fabrica actividad. Debe informarse al visitante (Decisión 4).

#### Opción D — Simulador continuo

Un proceso agrega actividad ficticia de forma permanente.

Desventajas

- Fabrica señal en todo momento, mezclada sin pausa con la actividad real de los visitantes.
- Agrega un componente que no existe en el diseño de la plataforma.

#### Opción E — Reinicio de la base y recarga

El operador vacía la base y vuelve a cargar el seed cuando la demo envejece.

Desventajas

- Elimina las identidades y propuestas de los usuarios, que DP-0002 (Decisión 3) decidió conservar.

### Decisión 3 — Forma de carga

#### Opción A — Inserciones directas en la base

Un script SQL o un volcado inserta los datos.

Ventajas

- Simple y rápido.

Desventajas

- Saltea las validaciones de la aplicación: vínculos hacia propuestas inexistentes o posteriores, más de 10 vínculos, etiquetas mal formadas o un tombstone que no respete P-0023 pasarían sin detectarse.

#### Opción B — Comando de operador que usa las reglas de la aplicación

El seed es una herramienta de línea de comando de la aplicación destino que reutiliza su capa de persistencia y sus validaciones, con un reloj inyectable para fechar los datos en el pasado. La propuesta retirada se produce publicándola y aplicándole después la operación real de retiro de P-0023.

Ventajas

- Los datos cumplen las mismas reglas que los creados por los usuarios.
- El tombstone sale del mecanismo de retiro verdadero, lo que además lo prueba.

Desventajas

- Requiere que el reloj de la aplicación sea inyectable.

### Decisión 4 — Información al visitante

#### Opción A — Sin aviso

Desventajas

- Un visitante puede creer que propuestas y apoyos fueron creados por personas reales.

#### Opción B — Aviso global y permanente

Toda pantalla de la demo informa que las propuestas iniciales y la actividad de las oleadas son ficticias.

Ventajas

- Informa sin modificar el modelo de datos de las propuestas.

Desventajas

- No permite distinguir, propuesta por propuesta, qué contenido es ficticio.

#### Opción C — Marca en cada propuesta ficticia

Desventajas

- Requiere agregar a la propuesta un campo de origen que el modelo de P-0018 no tiene, y tratar las propuestas de forma distinta según quién las creó.
- No resuelve los apoyos: una propuesta de un usuario puede no tener apoyos ficticios, pero el conteo de una propuesta del seed mezcla apoyos ficticios y reales.

## Decisión

**Decisión 1.** Se adopta la Opción B: cada apoyo de los datos iniciales lo da un apoyante ficticio efímero, representado por una clave de apoyos aleatoria que se usa una sola vez para calcular la etiqueta de P-0029 y se descarta. El seed no crea identidades. El apoyo del autor de cada propuesta (P-0012) es el primero de sus apoyos, con la hora de publicación, salvo donde la composición indica que el autor lo retiró. Las propuestas del seed no generan eventos de publicación para el cupo (P-0017, P-0018), porque no tienen un autor cuyo cupo deba controlarse.

**Decisión 2.** Se adopta la Opción C.

- Todas las fechas de los datos iniciales son relativas al momento de la carga. La hora de cada apoyo se distribuye entre la publicación de su propuesta y el momento de la carga según el perfil indicado en la composición.
- El comando de **oleada** agrega, por defecto, 60 apoyos de apoyantes efímeros con horas distribuidas dentro de las últimas 48 horas, repartidos entre 3 y 5 propuestas del seed elegidas al azar entre las no retiradas, y opcionalmente publica una propuesta de la reserva definida en la composición. Las oleadas solo apoyan propuestas del seed, identificadas por los `id` fijos que define el propio seed; nunca agregan apoyos a propuestas creadas por usuarios.
- Ni el seed ni las oleadas eliminan datos.

**Decisión 3.** Se adopta la Opción B. El seed y la oleada son comandos de operador de la aplicación destino (P-0024) que reutilizan sus validaciones con un reloj inyectable. El seed solo se ejecuta si la base no contiene propuestas; en caso contrario termina sin hacer cambios. Las propuestas del seed tienen `id` fijos en el formato de la demo (`identificador_propuestas.md`). La carga no invoca al revisor de lenguaje (P-0011): los textos del seed se validan contra el revisor una vez, durante el desarrollo.

**Decisión 4.** Se adopta la Opción B: un aviso global y permanente en todas las pantallas de la aplicación destino informa que las propuestas iniciales y la actividad de las oleadas son ficticias y fueron generadas para la demostración. El texto exacto y su ubicación se definen junto con los textos de la demo y la guía visual.

## Justificación

P-0029 vuelve innecesarias las identidades sintéticas: si los apoyos no referencian identidades, un apoyante ficticio no necesita existir más allá del cálculo de su etiqueta. La Opción B produce apoyos indistinguibles en forma de los reales, sin dejar en la base identidades inutilizables ni claves almacenadas.

Las oleadas son la única forma de mantener la demo representativa sin contradecir la persistencia de los datos de los usuarios (DP-0002, Decisión 3) y sin un componente permanente que fabrique señal. Restringirlas a las propuestas del seed evita alterar el conteo de propuestas creadas por personas reales, y hacerlas una operación explícita del operador mantiene la actividad ficticia acotada y deliberada.

Cargar los datos con las reglas de la aplicación garantiza que el seed no muestre situaciones que el sistema real no admitiría, y obtener el tombstone con la operación real de retiro lo convierte en una demostración fiel de P-0023.

El aviso global es honesto con el visitante sin agregar al modelo de datos un campo que el diseño de la plataforma no tiene.

## Consecuencias

- La aplicación destino necesita un reloj inyectable para fechar datos en el pasado; es útil también para los tests.
- Recién cargada, la demo muestra todas las señales. Si pasan 48 horas sin actividad suficiente, 🔥 y 🌱 desaparecen hasta que haya actividad real o el operador ejecute una oleada.
- El ranking de la demo no representa el comportamiento de una población real (ya aceptado en DP-0002), y las oleadas lo hacen explícitamente: el aviso de la Decisión 4 es obligatorio.
- Los textos de las causales del catálogo de retiro (P-0023) son necesarios para el tombstone del seed; se redactan junto con el tutorial y los términos y condiciones.
- Los textos de las propuestas del seed se redactan durante el desarrollo siguiendo la composición, y se versionan en el repositorio público junto con el código.
- La composición se diseñó con los valores por defecto de los parámetros. Si se modifican G, MT, ME, las ventanas, `percentil_tendencia` o `minimo_actividad_senales`, las señales esperadas cambian.

## Referencias

- DP-0002 — Alcance funcional del MVP de la demo
- DI-0001 — Stack tecnológico de la demo
- DI-0002 — Persistencia y despliegue de la demo
- P-0010 — Ranking y visualización de propuestas
- P-0011 — Revisor automático de lenguaje en propuestas
- P-0012 — Mecanismo de apoyo a propuestas
- P-0017 — Límite anual de propuestas por ciudadano
- P-0018 — Modelo de datos de propuestas
- P-0019 — Búsqueda y filtrado de propuestas
- P-0023 — Moderación de contenido y retiro de propuestas
- P-0024 — Ausencia de perfil administrativo
- P-0025 — Unicidad y desambiguación de pseudónimos
- P-0029 — Registro de apoyos desvinculado de la identidad y señales temporales del ranking
- `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`
- `demo/implementation/aplicaciones/participacion_ciudadana/identificador_propuestas.md`
- `demo/implementation/aplicaciones/participacion_ciudadana/datos_iniciales.md`
