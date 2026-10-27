# P-0027 — Actividad y expiración de la sesión

**Estado:** Activo

## Contexto

P-0005 establece que la plataforma usa sesiones temporales que expiran por inactividad (default 1 hora). Al expirar, la sesión se elimina completamente, el ciudadano es redirigido a la pantalla de ingreso y el sistema no muestra la identidad anónima utilizada. P-0005 protege dos cosas: el acceso indebido en dispositivos compartidos, cuando el ciudadano se aleja sin cerrar sesión, y la percepción de que el sistema no recuerda al ciudadano una vez terminada la sesión.

P-0005 no define qué cuenta como actividad ni qué ocurre en la pantalla en el momento de la expiración. El inventario de flujos (`notas/inventario_de_flujos_ui_ux.md`) asumió que la expiración se detecta recién cuando el ciudadano pulsa un control que requiere sesión, y dejó identificado un problema: si solo las peticiones al servidor cuentan como actividad, un ciudadano que pasa más de una hora redactando una propuesta llega a publicar con la sesión vencida y pierde el texto. El borrador vive solo en la memoria del navegador (inventario, H-10) y el sistema no garantiza su preservación (P-0022 Decisión 4).

Hay un segundo problema, menos visible. Si la expiración se detecta recién en la próxima acción, la página abierta sigue mostrando, hasta que alguien la toque, lo que el ciudadano veía: el estado de sus apoyos ("Retirar apoyo"), los resultados del filtro "apoyadas por mí", su cupo y eventualmente su identidad anónima. En un dispositivo compartido, quien se sienta después ve qué propuestas apoya la persona anterior. Eso contradice el propósito de P-0005.

Las preguntas que este ADR responde son:

- ¿Qué cuenta como actividad para renovar la sesión?
- ¿En qué pantallas se aplica?
- ¿Qué ocurre en la pantalla cuando la sesión expira?
- ¿Se preserva de algún modo un borrador sin publicar?

## Opciones consideradas

### Decisión 1 — Definición de actividad

#### Opción A — Solo peticiones al servidor

Únicamente las acciones que generan una petición al servidor (navegar, apoyar, publicar) renuevan la sesión.

Ventajas

- Implementación trivial.

Desventajas

- Leer o redactar durante más de una hora sin generar peticiones hace expirar la sesión de un ciudadano que está usando activamente la plataforma. La redacción de una propuesta larga se pierde.

#### Opción B — Pulso periódico mientras la página está abierta

El navegador avisa periódicamente al servidor mientras haya una pantalla con sesión abierta.

Ventajas

- Ninguna redacción se pierde por expiración.

Desventajas

- Una pantalla abierta en una computadora abandonada mantiene la sesión viva indefinidamente. Anula la protección de P-0005 en dispositivos compartidos.

#### Opción C — Interacción real del ciudadano

Cuentan como actividad las interacciones del ciudadano con la interfaz: teclear, hacer clic o tocar, desplazarse. Mientras haya interacción, el navegador avisa al servidor con una frecuencia acotada y la sesión se renueva. Sin interacción, la sesión expira aunque la página siga abierta.

Ventajas

- Un ciudadano que lee o redacta nunca pierde la sesión.
- Una pantalla abandonada expira igual: se preserva la protección de P-0005.

Desventajas

- Agrega peticiones periódicas mientras el ciudadano interactúa. Se acotan en frecuencia y no llevan contenido.

### Decisión 2 — Alcance

#### Opción A — Solo el editor de propuestas

Ventajas

- Mínimo indispensable para resolver el problema de la redacción larga.

Desventajas

- "Actividad" significa cosas distintas en distintas pantallas. Leer una propuesta extensa desplazándose sin hacer clic puede hacer expirar la sesión.

#### Opción B — Todas las pantallas con sesión

Ventajas

- Una sola definición de actividad en toda la aplicación, fácil de entender y de auditar.

Desventajas

- Más peticiones de renovación que la Opción A.

### Decisión 3 — Comportamiento de la pantalla al expirar

#### Opción A — Detección en la próxima acción

La pantalla queda como estaba hasta que el ciudadano pulsa un control que requiere sesión; recién entonces se lo lleva a la pantalla de ingreso.

Ventajas

- Implementación simple.

Desventajas

- La página abierta sigue mostrando el estado de los apoyos, el cupo y eventualmente la identidad anónima del ciudadano a quien se siente después frente al dispositivo.

#### Opción B — Limpieza inmediata

El navegador conoce el plazo de inactividad. Al vencer, borra por iniciativa propia todo el contenido de la sesión y lleva a la pantalla de ingreso, sin esperar ninguna acción.

Ventajas

- Cumple literalmente P-0005: después de la expiración no queda a la vista nada de la sesión anterior.

Desventajas

- Un borrador sin publicar desaparece en el momento de la expiración. Se perdería igual en la próxima acción.

### Decisión 4 — Preservación del borrador ante la expiración

#### Opción A — Reingreso sin perder el borrador

Al expirar con un borrador abierto, un diálogo pide pseudónimo y frase secreta y, tras el reingreso, devuelve el texto al editor.

Ventajas

- Ninguna redacción se pierde.

Desventajas

- Contradice P-0005, que establece que al expirar la sesión se elimina completamente y el ciudadano es redirigido a la pantalla de ingreso.
- El borrador quedaría en el dispositivo después de la expiración, en contra de H-10 y de P-0022 Decisión 4, que no garantizan su preservación.
- El diálogo no puede verificar que reingrese la misma identidad sin recordarla, lo que P-0005 prohíbe.

#### Opción B — Guardado automático del borrador

En el navegador (almacenamiento local) o en el servidor.

Desventajas

- En el navegador, el borrador sobrevive al cierre de la sesión: en un dispositivo compartido lo ve el siguiente usuario. Es memoria entre sesiones.
- En el servidor, crea contenido no publicado asociado a un `anon_id`, en contra de H-10 y de P-0022 Decisión 4.

#### Opción C — Sin preservación

El borrador se pierde si la sesión expira. Con la Decisión 1, eso solo ocurre si el ciudadano dejó de interactuar durante todo el plazo de inactividad.

Ventajas

- Coherente con P-0005, H-10 y P-0022.
- Con la interacción real como actividad, el caso queda reducido a quien abandonó la pantalla, que es justamente el caso que P-0005 protege.

Desventajas

- Quien se ausenta más que el plazo con un texto a medio escribir lo pierde.

## Decisiones

**Decisión 1:** Opción C. Cuentan como actividad las interacciones del ciudadano con la interfaz: teclado, clic o toque, y desplazamiento. Las peticiones normales al servidor también renuevan la sesión. Mientras haya interacción sin peticiones, el navegador envía una notificación de actividad como máximo cada 5 minutos (configurable), y solo si hubo interacción desde la notificación anterior. Temporizadores, pestañas abiertas o el foco de la ventana no cuentan como actividad.

**Decisión 2:** Opción B. La definición de actividad aplica a todas las pantallas con sesión. La interacción en cualquier pestaña de la aplicación renueva la sesión.

**Decisión 3:** Opción B. Al vencer el plazo de inactividad, el navegador elimina todo el contenido de la sesión de la pantalla y lleva a la pantalla de ingreso, en todas las pestañas de la aplicación. La pantalla de ingreso puede informar de forma genérica que la sesión se cerró por inactividad, sin mencionar la identidad anónima (P-0005).

**Decisión 4:** Opción C. No hay preservación del borrador ante la expiración.

No hay aviso previo a la expiración. Con la Decisión 1, la sesión solo expira cuando el ciudadano dejó de interactuar durante todo el plazo, es decir, cuando abandonó la pantalla; un aviso no tendría a quién avisarle.

## Justificación

La interacción real como actividad resuelve el problema de la redacción larga sin debilitar P-0005: lo que P-0005 protege es la pantalla abandonada, y una pantalla abandonada no registra interacción. Un pulso por temporizador resolvería la redacción a costa de anular esa protección; la interacción real distingue los dos casos.

Una sola definición de actividad para toda la aplicación es más simple de entender, implementar y auditar que una regla especial para el editor, y cubre también la lectura extensa.

La limpieza inmediata no es una regla nueva sino la aplicación literal de P-0005, que establece que tras la expiración el sistema no muestra la identidad anónima previamente utilizada. La página abierta revela más que la identidad: revela qué propuestas apoya el ciudadano, que es información de opinión.

La preservación del borrador se descarta porque todas sus variantes contradicen decisiones cerradas (P-0005, H-10, P-0022) o introducen memoria entre sesiones. Con las Decisiones 1 a 3, la pérdida queda reducida al caso de abandono de la pantalla.

## Consecuencias

- La notificación de actividad no lleva contenido, no se registra en logs (no es un evento de P-0020) y solo actualiza en el servidor la marca de última actividad de la sesión. El cierre de sesión por expiración se registra como "sesión destruida" según P-0020.
- El navegador evalúa la expiración contra el tiempo real, no solo contra temporizadores: al reanudarse un dispositivo suspendido, una sesión vencida se limpia de inmediato.
- El servidor sigue siendo la fuente de verdad de la sesión. Si el navegador no limpia la pantalla (por ejemplo, sin JavaScript), la sesión expira igual en el servidor y la próxima acción lleva a la pantalla de ingreso.
- El plazo de inactividad (P-0005) y la frecuencia máxima de notificación son configurables por el operador.
- Un borrador sin publicar se pierde solo si el ciudadano dejó de interactuar durante todo el plazo de inactividad. El tutorial o la pantalla de redacción pueden informarlo, en línea con el mensaje de P-0022 Decisión 4.
- Se resuelve el recordatorio sobre redacciones largas identificado en el inventario de flujos (F-AP-08).

## Referencias

- P-0001 — Visibilidad pública de la identidad anónima
- P-0005 — Gestión de sesiones de usuario
- P-0020 — Política de logs y retención de metadatos
- P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos
- P-0028 — Visibilidad del pseudónimo durante la sesión
- `notas/inventario_de_flujos_ui_ux.md` — expiración de sesión, F-AP-03, F-AP-08 y H-10
- `design/capa_de_identidad/identity_model.md` — No sugerir memoria entre sesiones
