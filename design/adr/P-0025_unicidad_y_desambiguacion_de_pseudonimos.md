# P-0025 — Unicidad y desambiguación de pseudónimos

**Estado:** Activo

## Contexto

El pseudónimo amigable que el emisor asigna al ciudadano en cada emisión cumple dos funciones distintas. Por un lado es la representación visible de la identidad anónima ante el propio ciudadano (P-0002, P-0003). Por otro lado, dentro de la aplicación destino, opera como identificador de login junto con la frase secreta (P-0004): la combinación `{pseudónimo, frase_secreta}` es lo que la aplicación usa para autenticar a un ciudadano en visitas posteriores.

La primera función no exige unicidad: que dos ciudadanos vean "Hola Lobo Azul 714" al ingresar no rompe nada, porque el pseudónimo no es visible para nadie más. La segunda función sí la exige: si dos identidades activas terminaran con el mismo pseudónimo, la combinación `{pseudónimo, frase_secreta}` dejaría de identificar unívocamente un registro y el modelo de autenticación se rompería.

P-0015 Decisión 2 estableció que el emisor almacena exclusivamente `{anon_seed, fecha_emision}`, sin retener ningún registro de los pseudónimos emitidos. Esto deja al emisor sin capacidad de chequear localmente si un pseudónimo candidato ya fue asignado. La aplicación destino, en cambio, sí tiene los pseudónimos en su base como parte del registro de cada ciudadano: es ahí donde se usan como login.

El espacio de pseudónimos sin sufijo (+letra al final del número) con las listas actuales es de 1.858.140 combinaciones (60 animales × 31 colores/adjetivos × 999 números). La probabilidad de colisión por nueva emisión es del orden de n/m, pero la probabilidad acumulada crece cuadráticamente (paradoja del cumpleaños): con ~1.600 ciudadanos registrados ya hay 50% de probabilidad de tener al menos una colisión activa, y con 10.000 ciudadanos las colisiones esperadas son del orden de decenas. Por lo tanto, a escala objetivo del sistema (termómetro social a escala ciudadana), las colisiones naturales son esperables, no excepcionales, si no se amplía el espacio de psuedónimos sin sufijo. Afortunadamente este escenario se contenpló, permitiendo **agregar opcionalmente** un sufijo de una letra mayúscula al final del número para ampliar el espacio de identidades anónimas a 39.020.940 combinaciones (60 animales × 31 colores/adjetivos × 999 números x 21 letras). 

Adicionalmente, P-0016 establece que la aplicación destino no invalida identidades anónimas: el conjunto de pseudónimos ocupados crece monótonamente. Cualquier estrategia que dependa de "liberar slots con el tiempo" queda descartada por esta propiedad.

Las preguntas de diseño que motivaron este ADR son:

- ¿Cómo se garantiza unicidad de pseudónimo entre identidades activas, dado que el emisor no almacena el conjunto de pseudónimos emitidos?
- ¿La verificación de unicidad debe vivir en el emisor o en la aplicación destino?
- ¿Cómo se evita una condición de carrera durante la deliberación del ciudadano frente al pseudónimo propuesto?
- ¿Cuando conviene aplicar el sufijo opcional para ampliar el espacio de pseudónimos?

## Opciones consideradas

### Decisión 1 — Mecanismo de unicidad de pseudónimos

#### Opción A — Permitir colisiones y desambiguar en login con la frase secreta

La aplicación destino acepta múltiples registros con el mismo pseudónimo. En el login recupera todos los registros que coinciden con el pseudónimo normalizado y aplica Argon2id con el salt y los parámetros de cada uno hasta encontrar match o agotarlos.

Ventajas

- No requiere mecanismo nuevo en el emisor ni en el contrato capa↔aplicación.
- Implementación marginalmente simple en la aplicación destino.

Desventajas

- Eleva el costo del login a O(N) sobre Argon2id, no O(1). Con Argon2id calibrado a los valores recomendados, la latencia se vuelve perceptible para el ciudadano honesto incluso con N=2.
- Amplifica el costo del brute-force sobre la frase secreta: un atacante que sabe (o sospecha) que dos identidades comparten pseudónimo tantea contra ambas en paralelo, duplicando su probabilidad de éxito por intento.
- Habilita un ataque de colisión deliberada que interactúa mal con P-0003. La regeneración de pseudónimos durante la creación es sin límite por diseño: un atacante con un objetivo concreto autentica una vez con AUTENTICAR y regenera hasta acertar el pseudónimo de su objetivo. Con el espacio actual, ~650.000 regeneraciones para 50% de éxito sobre un objetivo específico; server-side rápido, es factible. No obtiene la frase secreta de la víctima, pero infla N para esa identidad y habilita la amplificación del brute-force descrita arriba.
- A escala objetivo, las colisiones naturales no son excepcionales sino esperables.

#### Opción B — Consultar a la aplicación destino al generar

El emisor consulta a la aplicación destino la disponibilidad del pseudónimo candidato antes de presentarlo al ciudadano. La verificación final de unicidad ocurre atómicamente en el commit de la entrega (P-0022 Decisión 3): la aplicación destino persiste el registro solo si el pseudónimo sigue libre en ese momento.

Ventajas

- Resuelve el problema por invariante de protocolo: no hay colisiones activas posibles si la verificación atómica final se cumple.
- Aprovecha el canal emisor↔aplicación destino que el flujo de emisión ya necesita por P-0022 Decisión 3. No abre un canal nuevo: extiende el contenido del existente.
- No modifica P-0015. El emisor sigue almacenando exclusivamente `{anon_seed, fecha_emision}`.
- El login en la aplicación destino conserva su costo O(1) sobre Argon2id.
- Neutraliza el ataque de colisión deliberada de la Opción A: el atacante no puede observar pseudónimos que no le fueron ofrecidos.

Desventajas

- Extiende el contrato capa↔aplicación destino con operaciones adicionales.
- Introduce una dependencia adicional en el flujo de emisión cuya política de fallos requiere definir (alineable con P-0022 Decisión 3).

#### Opción C — Tabla disjunta en el emisor de pseudónimos usados

El emisor mantiene un conjunto de pseudónimos usados separado de `{anon_seed, fecha_emision}`, sin asociación entre ambos. Chequea contra ese conjunto antes de presentar un candidato.

Ventajas

- Sin dependencia adicional con la aplicación destino.

Desventajas

- Modifica P-0015 Decisión 2 (qué almacena el emisor) y requiere supersesión parcial. P-0015 había decidido no centralizar en el emisor información sobre las identidades emitidas más allá de la mínima necesaria para unicidad y cool-down; esta opción agrega el conjunto completo de pseudónimos emitidos.
- La disjunción no protege contra un atacante con acceso simultáneo al emisor y a la aplicación destino, que es el escenario contemplado por el modelo de amenazas intermedio de P-0006: el conjunto del emisor coincide con el conjunto de pseudónimos de la aplicación.

#### Opción D — Ampliar el espacio de pseudónimos para reducir probabilidad de colisión

Ampliar el espacio de pseudónimos agregando el sufijo **opcional** de la letra al final del número para que la probabilidad de colisión natural a la escala objetivo (39.020.940 combinaciones) sea aceptablemente baja, **sin agregar mecanismo de verificación**.

Ventajas

- Sin cambios en mecanismos ni en contratos.

Desventajas

- Agrega un elemento al pseudónimo que lo vuelve más dificil de recordar como identidad anónima.

### Decisión 2 — Mecánica del chequeo atómico

Esta decisión sólo aplica si la Decisión 1 adopta la Opción B.

#### Opción A — Consulta sin reserva más verificación atómica en el commit

El emisor consulta sólo para evitar presentar candidatos colisionados al ciudadano, pero la unicidad recién se cierra al persistir el registro en el commit de entrega. Entre la consulta y el commit no hay reserva: una emisión paralela puede tomar el mismo pseudónimo durante la ventana de deliberación del ciudadano. Si eso ocurre, el commit final detecta la colisión y rechaza la persistencia.

Ventajas

- Sin estado transitorio adicional en la aplicación destino.

Desventajas

- La ventana de carrera cubre toda la deliberación del ciudadano (potencialmente minutos). La probabilidad acumulada crece bajo alta concurrencia de registros.
- El caso de fallo en el commit final requiere de todos modos manejo UX: presentar un pseudónimo nuevo al ciudadano. La opción no evita ese paso; sólo lo hace más probable.

#### Opción B — Reserva con TTL más verificación atómica en el commit

La consulta inicial reserva el pseudónimo en la aplicación destino con un TTL configurable. La reserva se libera explícitamente cuando el ciudadano regenera, mediante un identificador efímero del flujo de emisión generado por el emisor. La verificación atómica en el commit de entrega vuelve a chequear unicidad. Si falla por colisión post-TTL (caso patológico residual), el flujo retoma con un pseudónimo nuevo y un mensaje informativo, sin consumo de cool-down.

Ventajas

- Reduce la ventana de carrera al residuo casi nulo de los casos en que el TTL expira durante la deliberación del ciudadano.
- El caso patológico se maneja con el mismo flujo UX que cualquier regeneración voluntaria: presentar un pseudónimo nuevo.
- Mantiene la independencia respecto del estado en régimen de la aplicación destino: las reservas son estado transitorio acotado a la emisión.

Desventajas

- Agrega estado transitorio en la aplicación destino (tabla de reservas con TTL).
- Requiere un identificador efímero del flujo de emisión para la liberación explícita.

### Decisión 3 — Ampliación del espacio de pseudónimos

#### Opción A — Mantener rango numérico 1..999

Espacio: 1.858.140 combinaciones. Válido para entornos chicos a moderados < 1.858.140.

Ventajas

- Las identidades anónimas (pseudónimos) son más simples y sencillos de recordar.

Desventajas

- Espacio total acotado a ~1,8M de identidades, no escala a poblaciones mayores.

#### Opción B — Ampliar usando el sufijo opcional de letra mayúscula

El sistema agrega el sufijo de una letra mayúscula al número (`Lobo Azul 714H`, `Tigre Audaz 23A`). El espacio total con sufijo opcional es: 60 × 31 × 999 x 21 = 39.020.940 combinaciones.

Ventajas

- Espacio total del orden de 40M de combinaciones: holgura amplia para escala objetivo.

Desventajas

- Los pseudónimos generados son menos amigables y sufijo es más díficil de recordar que un simple número.

#### Opción C — Ampliar usando el sufijo opcional de letra mayúscula al agotarse el espacio sin letra

Como Opción B, pero cuando el espacio sin sufijo de letra empieza a saturarse, el sistema agrega el sufijo de una letra mayúscula al número.

El emisor no mantiene estado explícito de agotamiento. La activación del sufijo es empírica: si tras un umbral configurable de intentos sin letra todos los candidatos generados resultan ocupados (Decisión 1 Opción B), el emisor genera candidatos con sufijo de letra. El sistema degrada gradualmente: con el espacio sin letra mayormente disponible, los ciudadanos reciben pseudónimos sin sufijo; con el espacio sin letra saturado, los reciben con sufijo.

Ventajas

- Espacio total del orden de 40M de combinaciones.
- Degradación gradual y empírica, sin necesidad de mantener estado de agotamiento en el emisor.
- Coherente con el patrón de reintentos con backoff de P-0022.
- Pseudónimos sin sufijo siguen siendo lo normal mientras haya espacio disponible.

Desventajas

- Pequeña inconsistencia visual entre pseudónimos con sufijo y sin sufijo (asumida).
- Requiere documentar el umbral de activación del sufijo. El umbral de "demasiados intentos sin letra" tiene que ser razonable. Demasiado bajo y se habilitan letras antes de tiempo; demasiado alto y la última persona registrable espera mucho. Default propuesto: 10 intentos.

## Decisiones

**Decisión 1:** Se adopta la **Opción B**. El emisor consulta a la aplicación destino la disponibilidad del pseudónimo candidato antes de presentarlo al ciudadano. La verificación final de unicidad ocurre atómicamente en el commit de la entrega definido por P-0022 Decisión 3. Esta decisión no modifica P-0015: el emisor sigue almacenando exclusivamente `{anon_seed, fecha_emision}`.

**Decisión 2:** Se adopta la **Opción B**. La consulta inicial reserva el pseudónimo en la aplicación destino con un TTL configurable. El valor por defecto es 10 minutos. La reserva se libera explícitamente cuando el ciudadano regenera el pseudónimo, identificándola mediante un identificador efímero del flujo de emisión generado por el emisor. La verificación atómica en el commit de entrega vuelve a chequear unicidad. Si el commit falla por colisión post-TTL, el flujo retoma con un pseudónimo nuevo y un mensaje informativo al ciudadano, sin consumo de cool-down (consistente con P-0022 Decisión 3).

**Decisión 3:** Se adopta la **Opción C**. El sufijo de letra se activa empíricamente: si tras un umbral configurable de intentos consecutivos sin sufijo (default 10) todos los candidatos resultan ocupados por reservas activas o registros definitivos en la aplicación destino (Decisión 1), el emisor genera candidatos con sufijo de letra.

## Justificación

El pseudónimo no es solo representación visible: opera como identificador de login (P-0004). La unicidad activa es una propiedad funcional, no cosmética. Aceptar colisiones traslada el problema a la verificación de credenciales y arrastra tres costos: encarece el login a O(N) sobre Argon2id, amplifica el costo del brute-force sobre la frase secreta cuando se conoce que dos identidades comparten pseudónimo, y habilita un vector nuevo de colisión deliberada vía la regeneración sin límite de P-0003. Ninguno de esos costos es despreciable a la escala objetivo del sistema.

La elección entre resolver el problema en el emisor o en la aplicación destino no es simétrica. El emisor decidió en P-0015 no almacenar el conjunto de pseudónimos: la justificación de esa decisión fue minimizar la información disponible ante un atacante con acceso al emisor. La aplicación destino, en cambio, ya almacena los pseudónimos como parte natural del registro del ciudadano (es donde se usan para login). Mover la verificación a la aplicación destino aprovecha información que ya está ahí; moverla al emisor concentra información que P-0015 había decidido no concentrar, sin ganancia compensatoria. La Opción B respeta P-0015 sin supersesiones.

El canal de comunicación emisor↔aplicación destino que esta decisión requiere ya existe: P-0022 Decisión 3 establece la entrega de la tripla y la confirmación previa al consumo del cool-down. Esta decisión extiende el contenido del canal con dos operaciones adicionales (consulta-con-reserva y liberación), no abre un canal nuevo. La dependencia entre componentes no aumenta en su forma, solo en su volumen.

La reserva con TTL sobre la consulta sin reserva (Decisión 2) reduce la ventana de carrera al residuo de los casos en que el TTL expira durante la deliberación del ciudadano. Las dos variantes requieren idéntico manejo UX en caso de colisión final; con TTL, la probabilidad de llegar a ese caso es órdenes de magnitud menor. El TTL de 10 minutos como default es holgado para deliberación con varias regeneraciones y suficientemente corto para no acumular reservas zombi.

La ampliación del espacio de pseudónimos (Decisión 3) es complementaria a las Decisiones 1 y 2: reduce la frecuencia esperada de regeneraciones por colisión y extiende sustancialmente la cantidad de identidades activas históricas que el sistema puede acomodar. El sufijo opcional de letra al agotarse el espacio sin letra evita un techo duro de combinaciones y degrada gradualmente, sin requerir que el emisor mantenga estado explícito de agotamiento: la activación del sufijo es consecuencia natural del mecanismo de reintentos sobre la consulta-con-reserva.

## Consecuencias

- El contrato capa↔aplicación destino se extiende con dos operaciones, definidas en `design/capa_de_identidad/README.md`:

  - **Consulta-con-reserva.** El emisor envía un pseudónimo candidato y un identificador efímero del flujo de emisión. La aplicación destino, en una operación atómica, verifica disponibilidad, registra la reserva con TTL si está libre, y responde "libre" o "ocupado". Si responde "ocupado", el emisor regenera y reintenta.
  - **Liberación.** El emisor envía el identificador efímero del flujo de emisión y la aplicación destino libera la reserva asociada. Se invoca cuando el ciudadano regenera el pseudónimo.

- La aplicación destino mantiene una tabla de reservas con TTL, separada del registro definitivo de ciudadanos. El detalle del modelo de datos de esa tabla pertenece al diseño de cada aplicación destino que se construya sobre la capa.

- El flujo de emisión incorpora estos pasos. El paso de presentación del pseudónimo al ciudadano se descompone en consulta-con-reserva, presentación, y liberación al regenerar (o aceptación). El paso de entrega y confirmación gana una bifurcación de fallo nueva: colisión post-TTL en el commit final.

- En el caso patológico de colisión en el commit final, la aplicación destino rechaza la persistencia con un código específico. El emisor presenta un pseudónimo nuevo al ciudadano con un mensaje informativo del tipo "este pseudónimo fue tomado mientras decidías, te ofrecemos otro". El ciudadano puede aceptar o regenerar. No se consume cool-down, consistente con P-0022 Decisión 3.

- La política ante fallos de la consulta-con-reserva o de la liberación se alinea con P-0022 Decisión 3: reintentos server-to-server con backoff exponencial, sin consumo de cool-down hasta el éxito de la entrega completa. Los parámetros concretos de reintentos y backoffs son configurables por el operador.

- El identificador efímero del flujo de emisión es generado por el emisor para cada nuevo flujo, no se persiste tras la emisión y no se loguea (consistente con la política de logs de P-0020). Su único propósito es permitir la liberación explícita de la reserva asociada y la verificación atómica final.

- La duración del TTL es configurable por el operador. El valor por defecto es 10 minutos.

- Este ADR no modifica P-0015. El emisor sigue almacenando exclusivamente `{anon_seed, fecha_emision}`.

## Referencias

- P-0002 — Representación de identidades anónimas mediante pseudónimos amigables
- P-0003 — Selección del pseudónimo de identidad anónima
- P-0004 — Autenticación de identidad anónima
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0009 — Algoritmo de almacenamiento de la frase secreta
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0016 — Invalidación de identidades anónimas en la plataforma participativa
- P-0020 — Política de logs y retención de metadatos
- P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino
- P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos
- `design/capa_de_identidad/README.md` — Contrato capa↔aplicación destino
- `design/capa_de_identidad/identity_wordlists.md` — Generación de identidades anónimas
