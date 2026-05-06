# P-0023 — Moderación de contenido y retiro de propuestas

**Estado:** Activo

## Contexto

La aplicación de participación ciudadana permite a cualquier ciudadano verificado publicar propuestas de forma libre, sujetas únicamente al revisor automático de lenguaje (P-0011) que evalúa la forma del texto sin intervenir sobre el contenido ideológico. La documentación conceptual declara la ausencia de moderación editorial como una propiedad del diseño, pero esa declaración nunca fue formalizada como decisión técnica con sus trade-offs reconocidos.

Al mismo tiempo, los documentos descriptivos del proyecto y varios ADRs (`vinculacion_de_propuestas.md`, `docs/architecture_overview.md`, P-0022) tratan la inmutabilidad de las propuestas publicadas como propiedad establecida, pero esa inmutabilidad tampoco fue formalizada en un ADR cerrado: aparece como propiedad implícita derivada del modelo de datos de P-0018, sin una decisión explícita que la sostenga ni reconozca sus excepciones.

Existe además un escenario que la documentación actual no cubre: una propuesta ya publicada puede necesitar retirarse por causas externas al diseño voluntario del sistema, principalmente requerimientos legales. Una orden judicial dirigida al operador del despliegue, o la aparición de contenido manifiestamente ilegal que pasó por el revisor de lenguaje (amenazas concretas, doxing, infracción de derechos de propiedad intelectual, etc.), son situaciones realistas que el sistema debe poder enfrentar sin contradecirse a sí mismo.

Este ADR formaliza tres cosas:

- la ausencia de moderación de contenido más allá del revisor automático de lenguaje;
- la inmutabilidad del contenido de las propuestas publicadas, distinguida del retiro;
- el mecanismo excepcional de retiro de propuestas por causales legales, sus condiciones, sus efectos en el modelo de datos y sus implicancias para el operador.

Las preguntas de diseño que motivaron este ADR son:

- ¿Existe moderación humana de propuestas más allá del revisor automático?
- ¿Existe un mecanismo de retiro de propuestas ya publicadas? Si existe, ¿bajo qué condiciones, quién puede activarlo y con qué efectos?
- ¿Cómo se concilia el retiro con la inmutabilidad de las propuestas publicadas?
- ¿Qué se muestra al público en lugar de una propuesta retirada?
- ¿Qué se preserva en el modelo de datos y qué se descarta?
- ¿Qué pasa con los apoyos, los vínculos salientes y los vínculos entrantes de una propuesta retirada?
- ¿Puede el autor solicitar el retiro de su propia propuesta?

A lo largo del ADR el término **operador** designa, en sentido operativo, a la entidad institucional responsable de desplegar y administrar la instancia de la aplicación. La formalización del rol del operador y de los actores institucionales del sistema corresponde a un documento de diseño separado.

## Opciones consideradas

### Decisión 1 — Moderación humana de contenido

#### Opción A — Sin moderación humana

La aplicación no incorpora roles humanos con capacidad de revisar, ocultar, eliminar o aprobar propuestas. El único filtro automático es el revisor de lenguaje (P-0011), que opera sobre la forma y no sobre el contenido. Las propuestas compiten entre sí por el apoyo ciudadano; las que reciben poco acompañamiento pierden visibilidad de forma natural a través del ranking (P-0010).

Ventajas

- Evita un punto central de control capaz de influir en la conversación pública.
- Elimina la sospecha estructural de que el contenido visible refleja decisiones editoriales con criterios políticos o ideológicos.
- Coherente con el rol del sistema como termómetro social y con la ausencia de reputación pública (P-0001).
- Hace que la dinámica de visibilidad dependa exclusivamente de la participación ciudadana.

Desventajas

- El sistema queda más expuesto a contenido de baja calidad o provocador. El ranking, el límite anual de propuestas (P-0017) y el revisor de lenguaje (P-0011) mitigan esa exposición pero no la eliminan.
- Contenido manifiestamente ilegal que escape al revisor de lenguaje queda visible.

#### Opción B — Moderación post-publicación reactiva

Existen perfiles humanos con capacidad para ocultar o eliminar propuestas tras su publicación, en respuesta a denuncias de ciudadanos o por iniciativa propia, según un criterio editorial declarado.

Desventajas

- Introduce un punto central de control capaz de influir en la conversación pública con criterios discrecionales.
- Erosiona la confianza en que la visibilidad de las propuestas refleja preferencia ciudadana y no decisiones administrativas.
- Requiere infraestructura de moderación (perfiles, denuncias, criterios, apelaciones) que excede el alcance del sistema.

#### Opción C — Pre-moderación humana antes de publicación

Las propuestas pasan por revisión humana antes de hacerse visibles.

Desventajas

- Convierte al operador en árbitro del discurso público.
- Introduce latencia y fricción incompatibles con el flujo de publicación diseñado.
- Contradice frontalmente el principio de ausencia de moderación editorial.

### Decisión 2 — Inmutabilidad del contenido de las propuestas publicadas

La pregunta es si el contenido de una propuesta puede modificarse después de la publicación.

#### Opción A — Mutable

El autor puede editar el texto de su propuesta tras publicarla.

Desventajas

- Permite la manipulación clásica de plataformas participativas: una propuesta acumula apoyo bajo una redacción y luego cambia su contenido.
- Compromete la integridad de la señal social: los apoyos quedan asociados a contenido que ya no existe.
- Requiere mecanismos accesorios (versionado, notificación a quienes apoyaron, política de retiro automático de apoyos) que complican el modelo sin aportar valor proporcional.

#### Opción B — Inmutable, con propuestas derivadas como mecanismo de evolución

Una propuesta publicada no puede modificarse. Para mejorar, ampliar o reformular una idea existe el mecanismo de propuestas derivadas (`vinculacion_de_propuestas.md`): una propuesta nueva con vínculo declarado a la original, que comienza sin apoyos.

Ventajas

- Preserva la integridad de la señal social: cada apoyo refiere a un contenido específico que no cambia.
- Coherente con la ausencia de identificadores de autoría en el modelo de datos (P-0001, P-0018): no hay a quién notificar de una edición ni a quién auditar.
- Permite la evolución de ideas mediante un mecanismo explícito (la derivada) que mantiene la trazabilidad sin alterar contenido apoyado.

Desventajas

- El autor que detecta un error tipográfico tras publicar no puede corregirlo. Debe convivir con el error o crear una propuesta derivada.

### Decisión 3 — Existencia de un mecanismo de retiro

La inmutabilidad de la Decisión 2 se refiere al **contenido** de la propuesta. Una operación distinta es el **retiro**: hacer que la propuesta deje de aportar contenido al sistema, sin alterar la inmutabilidad del contenido original (que se descarta) ni convivir con él modificado. La pregunta es si el sistema admite esa operación.

#### Opción A — Sin mecanismo de retiro

El operador no tiene ninguna capacidad técnica para retirar propuestas publicadas. Toda propuesta permanece visible indefinidamente.

Desventajas

- Una orden judicial dirigida al operador no puede cumplirse aunque el sistema esté técnicamente en posición de hacerlo. El operador queda expuesto a sanciones por una limitación auto-impuesta sin contrapartida de privacidad relevante.
- Contenido manifiestamente ilegal que escape al revisor de lenguaje (amenazas concretas, doxing, infracción de copyright) queda visible sin posibilidad de remoción.
- La rigidez frente a obligaciones legales realistas vuelve el sistema desplegable solo en jurisdicciones permisivas, restringiendo el alcance del proyecto sin razón de diseño que lo justifique.

#### Opción B — Mecanismo de retiro acotado a causales excepcionales

El operador dispone de una capacidad técnica de retiro limitada a causales legales, con efectos sobre el modelo de datos y la presentación pública definidos en las decisiones siguientes. El retiro no es edición y no contradice la Decisión 2: descarta el contenido en lugar de modificarlo.

Ventajas

- Permite responder a obligaciones legales reales sin contradecir la inmutabilidad del contenido publicado.
- Mantiene el retiro como operación excepcional con causales explícitas, no como herramienta de moderación discrecional.
- Compatible con la Decisión 1: la ausencia de moderación humana refiere a juicio editorial sobre el contenido; el retiro responde a obligaciones externas, no a criterio del operador sobre la calidad o conveniencia del contenido.

Desventajas

- Introduce una operación técnica en manos del operador que, mal usada, podría convertirse en mecanismo de moderación encubierta. Las decisiones siguientes acotan el riesgo mediante causales excepcionales y catálogo controlado.

#### Opción C — Mecanismo de retiro con catálogo abierto al criterio del operador

El operador puede retirar propuestas según su propio criterio, con causales libremente definidas en cada despliegue.

Desventajas

- Convierte el mecanismo de retiro en una vía indirecta de moderación editorial, contradiciendo la Decisión 1.
- Erosiona la confianza estructural del sistema: los ciudadanos no pueden distinguir un retiro por causal legal genuina de un retiro por desacuerdo del operador.

### Decisión 4 — Causales habilitantes y catálogo

Si el mecanismo de retiro existe (Decisión 3 = B), la pregunta es qué causales lo habilitan y cómo se gestiona la lista.

#### Opción A — Catálogo cerrado fijo

La aplicación trae un conjunto fijo de causales que el operador no puede modificar.

Desventajas

- El marco legal aplicable depende de la jurisdicción del despliegue. Una causal relevante en un país puede no aplicar en otro y viceversa. Un catálogo fijo o cubre demasiado o se queda corto.

#### Opción B — Catálogo configurable, acotado a obligaciones legales

La aplicación trae un catálogo por defecto con causales mínimas (orden judicial, contenido manifiestamente ilegal). El operador puede agregar o retirar entradas según el marco legal aplicable a su despliegue, con la restricción explícita de que las causales agregadas deben corresponder a obligaciones legales del marco jurídico de la jurisdicción del despliegue. El catálogo no es un mecanismo para introducir criterios editoriales o de moderación de contenido; esa restricción es parte del diseño y debe documentarse en la guía de operación.

Ventajas

- Permite adaptar el catálogo al marco legal real del despliegue sin abrir la puerta a moderación editorial encubierta.
- Mantiene un mínimo común reconocible (orden judicial, contenido manifiestamente ilegal) en cualquier despliegue.

Desventajas

- La restricción a obligaciones legales es declarativa: el sistema no puede impedir técnicamente que un operador agregue una causal disfrazada. La protección efectiva proviene de la auditabilidad del catálogo y de la coherencia entre causales declaradas y marco legal aplicable.

#### Opción C — Catálogo libremente configurable

El operador define el catálogo según su criterio sin restricción.

Desventajas

- Equivale a la Opción C de la Decisión 3 por la puerta de atrás: cualquier criterio puede declararse como causal.

### Decisión 5 — Actores legitimados para activar el retiro

#### Opción A — Solo autoridad judicial

El retiro se ejecuta exclusivamente en respuesta a un requerimiento judicial formal recibido por el operador.

Desventajas

- Contenido manifiestamente ilegal que escape al revisor de lenguaje y que el operador detecte por otros medios (denuncia, reportes externos, observación directa) queda visible hasta que un proceso judicial concluya, lo cual puede ser extenso.

#### Opción B — Autoridad judicial más operador ante contenido manifiestamente ilegal

El retiro puede ejecutarse en dos situaciones: en respuesta a un requerimiento judicial, o por iniciativa del operador cuando el contenido es manifiestamente ilegal según el marco legal aplicable. La definición concreta de "manifiestamente ilegal" depende de la jurisdicción y queda fuera del alcance de este ADR.

Ventajas

- Permite respuesta razonable ante contenido cuyo carácter ilegal es evidente sin esperar un proceso judicial completo.
- El operador asume la responsabilidad de calificar la ilegalidad bajo su propio riesgo legal, lo cual es coherente con su rol de responsable del despliegue.

Desventajas

- Otorga al operador un margen de juicio que requiere disciplina para no convertirse en moderación editorial. El catálogo controlado de la Decisión 4 acota esta superficie.

#### Opción C — Cualquier ciudadano vía denuncia

El retiro se activa por denuncia de cualquier ciudadano, con o sin revisión posterior.

Desventajas

- Convierte el mecanismo en herramienta de censura por reporte masivo.
- Requiere infraestructura de denuncias, revisión y apelación que el sistema explícitamente no incorpora (Decisión 1).

### Decisión 6 — Estado público de una propuesta retirada

#### Opción A — Eliminación sin rastro

La propuesta desaparece de listados, búsquedas, vínculos. Quien intenta acceder por URL recibe un error genérico.

Desventajas

- Los ciudadanos que apoyaron o vincularon propuestas a la retirada no tienen forma de saber qué pasó. Vínculos rotos sin explicación generan confusión y sospecha.
- Más opaco que informativo: el sistema no distingue entre una propuesta inexistente y una retirada.

#### Opción B — Placeholder mudo

La propuesta queda accesible pero muestra un mensaje genérico ("Esta propuesta fue retirada") sin información sobre la causa.

Desventajas

- Comunica menos de lo posible sin reducir riesgos de forma proporcional.
- No diferencia entre causales relevantes para el ciudadano (orden judicial vs. infracción de copyright vs. otra) que podrían ser de interés legítimo.

#### Opción C — Tombstone con motivo categorizado, sin contenido original

La propuesta queda accesible pero su contenido se reescribe a un texto fijo derivado del catálogo de causales (Decisión 4). El título indica que la propuesta fue retirada y la causa categorizada; el cuerpo amplía esa información con un texto fijo predefinido por causal. No se conserva nada del contenido original ni de los datos asociados.

Ventajas

- Comunica al ciudadano qué ocurrió y por qué, en términos categorizados y verificables contra el catálogo público de causales.
- Preserva la integridad referencial de los vínculos entrantes desde otras propuestas: quien siga el vínculo aterriza en el tombstone y entiende la situación.
- No requiere tratamiento especial de las propuestas retiradas en otros mecanismos del sistema (ranking, búsqueda, filtros). El tombstone es una propuesta más con apoyos en cero que se ordena naturalmente al fondo del ranking.

Desventajas

- Una búsqueda por palabras del tombstone (por ejemplo, "removida") devuelve los tombstones existentes. Es ruido menor y aceptado: tratarlo como caso especial requeriría modificar P-0019 sin beneficio proporcional.

### Decisión 7 — Datos preservados y datos descartados

Si la propuesta retirada se presenta como tombstone (Decisión 6 = C), la pregunta es exactamente qué se preserva del registro original y qué se descarta.

#### Opción A — Preservación del registro completo con flag de retirada

La fila original se mantiene íntegra, marcada con un flag que indica que está retirada. La presentación pública renderiza el tombstone en lugar del contenido.

Desventajas

- Conserva contenido que el retiro buscó eliminar. Una orden judicial que ordena retirar una propuesta no se cumple manteniendo el contenido en la base con un flag.
- Mantiene los apoyos asociados a contenido retirado, que es información residual sin función dentro del sistema una vez ejecutado el retiro.

#### Opción B — Reescritura del registro: solo persiste el `id` y el contenido del tombstone

La fila se reescribe. Se conserva el `id` por integridad referencial de vínculos entrantes. El resto se reemplaza o resetea: `titulo` y `cuerpo` pasan a los textos derivados del catálogo según la causal; `fecha_publicacion` se actualiza a la fecha del retiro; `conteo_apoyos` se resetea a cero; los apoyos individuales asociados se eliminan; los vínculos salientes se eliminan; el `score` queda recalculado por los mecanismos normales del ranking.

Ventajas

- No quedan rastros del contenido retirado en la base de datos: ni texto, ni apoyos, ni vínculos salientes, ni fecha original.
- Coherente con el principio de minimización: el sistema solo conserva lo mínimo necesario para que la operación funcione (el `id` para integridad referencial; el tombstone para informar a quien aterrice por vínculo entrante).
- El registro retirado se procesa como una propuesta cualquiera por el resto del sistema, sin requerir tratamiento especial en ranking, búsqueda, listados ni filtros.

Desventajas

- Se pierde la fecha original de publicación. Esa pérdida es deliberada: es parte de "no quedan rastros".
- Se pierde la señal social acumulada (apoyos). Es trade-off aceptado: preservar conteos asociados a contenido retirado por ilegalidad es más problemático que perder la señal.

### Decisión 8 — Apoyos asociados a una propuesta retirada

La Decisión 7 ya determina el destino de los apoyos: se eliminan junto con el resto del contenido. Esta decisión existe únicamente para hacer explícita la consecuencia y dejar registrado el trade-off, dado que los apoyos representan señal social acumulada.

Una propuesta con apoyos significativos retirada por causal legal pierde esa señal. La alternativa —preservar un conteo agregado o mantener los apoyos individuales desvinculados— se descarta porque:

- Conservar apoyos asociados a contenido que se retiró por ilegalidad mantiene en el sistema datos que el retiro buscó eliminar.
- Un conteo agregado sin contenido al cual referirse no aporta señal interpretable.
- La consistencia con la Decisión 7 (no quedan rastros) requiere que los apoyos también se eliminen.

### Decisión 9 — Vínculos de la propuesta retirada

Hay dos clases de vínculos a considerar, y se tratan de forma asimétrica como consecuencia de las decisiones anteriores.

**Vínculos salientes** (los que la propuesta retirada declaraba hacia otras): se eliminan. Forman parte del contenido de la propuesta retirada (`vinculacion_de_propuestas.md`) y se descartan junto con el resto según la Decisión 7.

**Vínculos entrantes** (los que otras propuestas declararon hacia la retirada): se mantienen sin cambios. Pertenecen a propuestas terceras que son inmutables (Decisión 2) y no podrían modificarse aunque se quisiera. Quien siga un vínculo entrante aterriza en el tombstone y entiende la situación.

Esta asimetría es consecuencia técnica directa del modelo de vínculos: los vínculos se almacenan como referencias desde la propuesta que los declara hacia las propuestas referenciadas (`vinculacion_de_propuestas.md`). Los vínculos de salida son propiedad de la propuesta declarante; los de entrada son propiedad de terceras.

No corresponde una decisión de "cascada de retiro": retirar automáticamente todas las propuestas que vinculan a una retirada implicaría retirar contenido legítimo de propuestas terceras sin causal propia. Una propuesta derivada que tomó base de otra retirada por orden judicial puede ser perfectamente lícita por sí misma.

### Decisión 10 — Retiro a pedido del autor

P-0001 establece que las propuestas no muestran ningún identificador del autor. P-0018 lleva esa decisión a su conclusión lógica en el modelo de datos: la base de datos de propuestas no contiene información sobre autoría. El emisor de identidad anónima, por su parte, no almacena vínculo entre ciudadano e identidad anónima emitida (P-0015).

La pregunta es si la aplicación debe incorporar un mecanismo que permita al autor solicitar el retiro de su propia propuesta, incluyendo escenarios de "derecho al olvido".

#### Opción A — No admite

La aplicación no admite retiro a pedido del autor. El autor que desee dejar de respaldar su propuesta puede retirar su apoyo (P-0012); no puede retirar la propuesta. Un escenario de derecho al olvido se canaliza por la vía judicial general (Decisión 5).

Ventajas

- Coherente con el modelo de datos: el sistema no tiene forma de validar que quien pide es realmente el autor, porque no almacena autoría.
- Coherente con el principio de credencial irrecuperable como propiedad del diseño: el sistema no construye mecanismos que reabran caminos cerrados deliberadamente.
- No requiere infraestructura nueva (prueba de autoría opt-in, tokens, registros adicionales) que reintroduzca formas de correlación que el diseño descarta.

Desventajas

- El autor que se arrepiente no tiene una vía nativa de retiro. Debe convivir con la propuesta o canalizar el retiro por vía judicial.

#### Opción B — Admite vía prueba de autoría opt-in

Al publicar, se entrega al ciudadano un token de autoría que solo él conserva. Si lo presenta, puede retirar.

Desventajas

- Introduce un mecanismo nuevo cuya seguridad y trade-offs requieren análisis propio (¿qué pasa si el token se filtra? ¿se puede transferir? ¿qué garantía da realmente el token sobre la identidad del autor?).
- Excede el alcance natural de este ADR. Si en el futuro se considera necesario, debe abordarse en un ADR específico que evalúe alternativas, no como sub-decisión acá.

#### Opción C — Admite vía cambio del modelo de datos

Almacenar el `anon_id` del autor asociado a la propuesta para permitir validar pedidos de retiro.

Desventajas

- Contradice frontalmente P-0018, decisión cerrada.
- Reintroduce la asociación propuesta-autor que el diseño descarta deliberadamente.

## Decisión

**Decisión 1.** Se adopta la Opción A: sin moderación humana de contenido más allá del revisor automático de lenguaje (P-0011). La aplicación no incorpora roles humanos con capacidad de revisar, ocultar, eliminar o aprobar propuestas según criterios editoriales.

**Decisión 2.** Se adopta la Opción B: el contenido de las propuestas publicadas es inmutable. La evolución de ideas se canaliza mediante propuestas derivadas (`vinculacion_de_propuestas.md`).

**Decisión 3.** Se adopta la Opción B: la aplicación incorpora un mecanismo excepcional de retiro de propuestas, limitado a causales legales del catálogo definido en la Decisión 4 y activable por los actores definidos en la Decisión 5.

**Decisión 4.** Se adopta la Opción B: catálogo configurable acotado a obligaciones legales. El catálogo trae por defecto dos causales: "orden judicial" y "contenido manifiestamente ilegal". El operador puede agregar o retirar entradas según el marco legal aplicable a su despliegue, con la restricción declarativa explícita de que las causales agregadas deben corresponder a obligaciones legales del marco jurídico de la jurisdicción del despliegue. El catálogo no es un mecanismo para introducir criterios editoriales o de moderación de contenido. Cada entrada del catálogo se compone de un identificador, un texto corto utilizado en el título del tombstone y un texto largo utilizado en el cuerpo del tombstone.

**Decisión 5.** Se adopta la Opción B: el retiro puede ejecutarse en respuesta a un requerimiento judicial recibido por el operador, o por iniciativa del operador ante contenido manifiestamente ilegal según el marco legal aplicable. Ningún otro actor (ciudadano, autor de la propuesta, terceros) puede activar el mecanismo de retiro.

**Decisión 6.** Se adopta la Opción C: una propuesta retirada se presenta como tombstone con motivo categorizado, sin contenido original. El título y el cuerpo del tombstone se derivan de los textos del catálogo asociados a la causal aplicada.

**Decisión 7.** Se adopta la Opción B: la propuesta retirada conserva únicamente su `id`. El resto se reescribe o se descarta. Concretamente:

- `titulo` se reemplaza por el texto corto de la causal del catálogo, con el formato `"Propuesta removida por <causal>"`.
- `cuerpo` se reemplaza por el texto largo de la causal del catálogo.
- `fecha_publicacion` se actualiza a la fecha del retiro.
- `conteo_apoyos` se resetea a cero.
- Los registros individuales de apoyo asociados a la propuesta se eliminan.
- Los `vinculos` (salientes) se eliminan.
- El `score` queda recalculado por los mecanismos normales del ranking (P-0010).

Los textos del título y cuerpo del tombstone se almacenan en la propuesta retirada al momento del retiro y son inmutables a partir de allí. Modificaciones posteriores al catálogo de causales no afectan tombstones existentes; cada tombstone queda congelado con los textos vigentes en el momento de su creación.

**Decisión 8.** Los apoyos asociados a una propuesta retirada se eliminan, en consistencia con la Decisión 7.

**Decisión 9.** Los vínculos salientes de una propuesta retirada se eliminan en el momento del retiro. Los vínculos entrantes desde otras propuestas se mantienen sin cambios; quien los siga aterriza en el tombstone.

**Decisión 10.** Se adopta la Opción A: la aplicación no admite retiro a pedido del autor. Esta decisión es consecuencia derivada de que las propuestas se publican sin información de autoría en el modelo de datos (P-0001, P-0018) y de que el emisor no almacena vínculo entre ciudadano e identidad anónima (P-0015). Un escenario de "derecho al olvido" se canaliza por la vía judicial general (Decisión 5).

## Justificación

La ausencia de moderación humana es lo que permite que la visibilidad de las propuestas refleje preferencia ciudadana real y no decisiones administrativas. Cualquier perfil con capacidad de ocultar o eliminar contenido por criterio propio introduce una sospecha estructural que el sistema no puede deshacer. El ranking con decaimiento temporal (P-0010), el límite anual de propuestas (P-0017) y el revisor de lenguaje (P-0011) actúan como mitigadores naturales del ruido sin centralizar el poder editorial.

La distinción entre **inmutabilidad del contenido** (Decisión 2) y **retiro** (Decisiones 3 a 9) es la clave que permite que ambas decisiones convivan sin contradecirse. La inmutabilidad protege la integridad de la señal social: los apoyos siempre refieren a un contenido que no cambia. El retiro descarta la propuesta en lugar de modificarla; el contenido original no es editado, es eliminado y reemplazado por un tombstone que no pretende ser la propuesta, sino una marca de su ausencia. La operación "edición" y la operación "retiro" son distintas y tienen efectos distintos: una preserva la propuesta con contenido modificado y rompe la integridad de los apoyos; la otra preserva el lugar pero descarta el contenido y elimina los apoyos.

El mecanismo de retiro existe porque rechazarlo no aporta privacidad ni anonimato al ciudadano y sí expone al operador a un riesgo legal evitable. Una orden judicial recibida por el operador es una situación realista en cualquier jurisdicción razonable. Diseñar el sistema con la rigidez de no poder ejecutarla técnicamente convierte un problema legal en un problema de diseño, sin contrapartida de privacidad relevante: la propuesta es contenido público; nada del retiro afecta el anonimato del ciudadano.

El acotamiento del catálogo a obligaciones legales (Decisión 4) y la restricción a actores legales (Decisión 5) son las dos protecciones contra el riesgo de que el mecanismo de retiro se convierta en moderación editorial encubierta. Ambas son declarativas: el sistema no puede impedir técnicamente que un operador agregue una causal disfrazada o retire por iniciativa propia un contenido que no es manifiestamente ilegal. La protección efectiva proviene de la auditabilidad: el catálogo es público, las causales aplicadas son visibles en cada tombstone, y un patrón de retiros injustificados es detectable. Esta es la misma lógica que el resto del sistema aplica frente a operadores potencialmente malintencionados (P-0006): no se busca confianza ciega sino integridad verificable.

La elección del tombstone como estado público (Decisión 6) y de la reescritura mínima como estado de datos (Decisión 7) responde al mismo principio. Eliminar sin rastro genera vínculos rotos sin explicación. Conservar contenido con un flag preserva información que el retiro buscaba eliminar. El tombstone con reescritura mínima comunica al ciudadano qué ocurrió y por qué, en términos categorizados, sin conservar contenido residual. La decisión de no dar tratamiento especial a las propuestas retiradas en otros mecanismos del sistema (ranking, búsqueda, filtros) es deliberada: minimiza el cambio sobre decisiones cerradas (P-0010, P-0018, P-0019) y aprovecha la dinámica natural del ranking, que envía al fondo cualquier propuesta con apoyos en cero.

La pérdida de la señal social acumulada al resetear los apoyos a cero (Decisión 8) es un trade-off aceptado. Una propuesta retirada por causal legal acumulando apoyos visibles es una situación incómoda: el operador no puede cumplir el retiro a medias preservando un conteo. La consistencia interna del sistema requiere que el retiro sea total.

El congelamiento del tombstone al momento del retiro (Decisión 7, último párrafo) protege contra manipulación retroactiva. Si los textos del tombstone se renderizaran dinámicamente desde el catálogo vigente, una edición posterior del catálogo cambiaría retroactivamente el motivo declarado de retiros pasados. Almacenar los textos en la propuesta retirada en el momento del retiro mantiene la trazabilidad y es coherente con el principio general de inmutabilidad.

La asimetría en el tratamiento de vínculos (Decisión 9) es consecuencia técnica del modelo: los vínculos pertenecen a la propuesta que los declara. Los salientes son contenido de la propuesta retirada y se eliminan; los entrantes son contenido de propuestas terceras inmutables y se mantienen. No corresponde cascada de retiro: una derivada de una propuesta retirada por orden judicial puede ser perfectamente lícita por sí misma, y retirarla automáticamente sería extender la causal legal del original a contenido de terceros sin justificación propia.

La negativa al retiro a pedido del autor (Decisión 10) es coherente con el conjunto del diseño. El sistema no almacena autoría; cualquier mecanismo de "prueba de autoría" requiere infraestructura nueva cuya seguridad y trade-offs deben evaluarse aparte. El derecho al olvido en contextos donde aplica jurídicamente puede canalizarse por la vía judicial general, que ya está cubierta por el mecanismo de retiro. La fricción adicional para el autor arrepentido es el costo de un diseño que prioriza la separación entre identidad real y actividad por sobre la conveniencia individual.

## Consecuencias

- La aplicación no incorpora perfiles humanos con capacidad de revisar, aprobar o eliminar propuestas según criterios editoriales. El único filtro automático sobre el contenido sigue siendo el revisor de lenguaje (P-0011).

- El modelo de datos de propuestas (P-0018) se mantiene íntegro. La inmutabilidad del contenido queda formalizada en este ADR como decisión explícita; los documentos descriptivos que la mencionaban como propiedad implícita (`vinculacion_de_propuestas.md`, `docs/propuesta/`, `docs/architecture_overview.md`, P-0022) quedan respaldados por una decisión cerrada.

- La aplicación incorpora un mecanismo de retiro de propuestas. Su implementación concreta —flujo administrativo, autenticación del operador, tipos de actor, integración con la persistencia— se aborda en el documento de implementación correspondiente cuando se llegue a esa etapa.

- La aplicación mantiene un catálogo configurable de causales de retiro. Cada entrada incluye identificador, texto corto y texto largo. El catálogo viene precargado con las causales "orden judicial" y "contenido manifiestamente ilegal". La guía de operación documenta el catálogo, los criterios de configuración y la restricción declarativa de que las entradas deben corresponder a obligaciones legales aplicables.

- La aplicación no almacena información sobre los actores que ejecutaron un retiro ni sobre los detalles del requerimiento judicial. Esa información, cuando exista, vive fuera del sistema (en la documentación administrativa del operador). Los logs del sistema relativos a operaciones de retiro se rigen por P-0020.

- La operación de retiro reescribe la fila de la propuesta según la Decisión 7, elimina los registros de apoyo asociados y elimina los vínculos salientes. Los vínculos entrantes desde otras propuestas se mantienen.

- Las propuestas retiradas no reciben tratamiento especial en los mecanismos de ranking, búsqueda y filtrado. Se procesan como propuestas cualesquiera con apoyos en cero, lo cual las ubica naturalmente al fondo del ranking. Las búsquedas pueden devolverlas si el texto del tombstone coincide con el término buscado; este ruido residual se acepta.

- El autor de una propuesta no puede solicitar su retiro mediante un mecanismo nativo de la aplicación. Si el contexto legal aplicable habilita un derecho al olvido, su ejercicio se canaliza por la vía judicial.

- El operador asume el riesgo legal de su despliegue. Este ADR no es una opinión jurídica; las obligaciones legales concretas (LPDP, código civil, normativa de protección de datos, normativa de copyright, normativa específica de la jurisdicción) dependen del marco legal aplicable y deben evaluarse por separado para cada despliegue. La guía de operación incluye un apartado sobre las capacidades y limitaciones del operador frente a requerimientos legales, con foco en lo que el sistema técnicamente puede y no puede cumplir.

- Capacidades y limitaciones técnicas del operador frente a requerimientos legales:
    - El operador **puede** retirar una propuesta concreta con efectos definidos en este ADR.
    - El operador **no puede** identificar al autor de una propuesta. La aplicación no almacena información de autoría (P-0001, P-0018) y el emisor no almacena vínculo entre ciudadano e identidad anónima (P-0015). Un requerimiento de identificación dirigido al operador no puede cumplirse aunque exista la voluntad de hacerlo, salvo en lo que respecta a información existente en el verificador externo de identidad fuera del alcance del sistema.
    - El operador **no puede** invalidar selectivamente el `anon_id` de un autor (P-0016) ni evitar que ese autor publique nuevamente.
    - El operador **puede** preservar el contenido de una propuesta antes de retirarla si una orden judicial lo exige expresamente, pero esa preservación queda fuera de la aplicación; el sistema, una vez ejecutado el retiro, no conserva el contenido original.

## Referencias

- P-0001 — Visibilidad pública de la identidad anónima
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0010 — Ranking de propuestas
- P-0011 — Revisor automático de lenguaje en propuestas
- P-0012 — Mecanismo de apoyo a propuestas
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0016 — Invalidación de identidades anónimas en la plataforma participativa
- P-0017 — Límite anual de propuestas por ciudadano
- P-0018 — Modelo de datos de propuestas
- P-0019 — Búsqueda y filtrado de propuestas
- P-0020 — Política de logs y retención de metadatos
- P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino
- P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos
- `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md` — Vinculación entre propuestas
- `design/threat_model.md` — Modelo de amenazas
- `notas/propuesta_guia_de_instalacion.md` — Material en gestación para la guía de instalación y operación
