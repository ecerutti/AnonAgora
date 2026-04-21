# 02 — Fundamentos y Lógica de funcionamiento

## Introducción

Este documento explica las decisiones que dan forma al sistema.

Cada una de ellas responde a problemas habituales en plataformas de participación ciudadana: falta de confianza, manipulación mediante cuentas falsas, saturación de propuestas o miedo a expresar opiniones políticas.

El objetivo no fue diseñar una plataforma perfecta, sino un sistema que encuentre equilibrios razonables entre participación abierta, anonimato, responsabilidad cívica y claridad en las señales sociales.

Las decisiones que se describen a continuación forman parte de esa lógica.

## Identidad verificada: evitar el problema de las cuentas falsas

Uno de los mayores problemas de cualquier plataforma participativa es la facilidad con la que pueden crearse cuentas falsas.

Si una persona puede registrarse muchas veces, también puede:

- votar múltiples veces  
- manipular rankings  
- simular apoyo social inexistente  

Por esa razón, el sistema requiere una verificación inicial de identidad. En un contexto como el argentino, esta verificación podría realizarse utilizando la **Plataforma de Autenticación Electrónica Central del Estado (AUTENTICAR)**, que actúa como intermediario entre los sistemas que necesitan validar la identidad de una persona y los proveedores de identidad del Estado, como ANSES, AFIP/ARCA, MiArgentina o RENAPER.

Esta verificación cumple una única función: confirmar que detrás de cada cuenta hay un ciudadano real y único.

Una vez completado ese paso, la identidad real queda desacoplada del resto de la plataforma y deja de participar en el funcionamiento cotidiano del sistema.

## Anonimato persistente: eliminar el miedo a participar

Muchas personas evitan expresar opiniones políticas cuando saben que sus decisiones quedan registradas con su nombre.

Ese temor puede surgir por múltiples razones:

- presiones sociales  
- entornos laborales  
- conflictos personales o comunitarios  
- desconfianza hacia el gobierno de turno  

Para reducir esa barrera, el sistema separa completamente la identidad real del comportamiento dentro de la plataforma.

Después de la verificación inicial, el ciudadano recibe una **identidad anónima persistente** que utiliza para todas sus acciones.

Esto significa que puede:

- votar propuestas  
- apoyar ideas  
- cambiar su opinión con el tiempo  
- proponer nuevas iniciativas  

sin que su identidad real sea visible o necesaria para participar dentro del sistema.

La persistencia de esa identidad anónima también permite observar cómo evolucionan las opiniones con el tiempo, algo que las encuestas tradicionales rara vez capturan.

### Opinión política sin identidad política

Buena parte de la conversación pública digital actual está condicionada por la identidad visible de quien habla. Antes de evaluar una idea, muchas veces se evalúa a la persona, su grupo, su pertenencia o su posicionamiento previo.

La plataforma propone invertir esa lógica. En lugar de exponer una identidad política pública y luego opinar, el ciudadano participa mediante una identidad anónima persistente. Eso permite que las propuestas circulen con mayor independencia de la tribu, el cargo, el prestigio o el rechazo que suele despertar quien las enuncia.

El resultado buscado es simple: que las ideas puedan reunir apoyo o rechazo más por su contenido que por la identidad pública de quien las impulsa.

## Apoyo sin voto negativo

La plataforma permite a los ciudadanos apoyar propuestas, pero no ofrece ningún mecanismo para rechazarlas, votarlas en contra ni expresar oposición.

Esta decisión es deliberada y responde a una preocupación concreta sobre las dinámicas sociales que habilitan los espacios de discusión pública digitales.

Cuando un sistema permite el voto negativo, rápidamente aparecen comportamientos que degradan la señal social que el sistema busca capturar. El voto bronca, el rechazo por afinidad partidaria y las campañas organizadas de descalificación ideológica son fenómenos bien conocidos en plataformas que ofrecen mecánicas de oposición explícita. En esos espacios, las propuestas dejan de competir por atraer apoyo y empiezan a competir por evitar rechazo, lo que favorece ideas blandas y desincentiva la expresión de posiciones originales o minoritarias.

La plataforma propone una lógica distinta. Cada ciudadano puede expresar apoyo a las ideas que le resultan valiosas, y nada más. Si una propuesta no le interesa o no está de acuerdo con ella, simplemente no la apoya. Esa falta de apoyo es, en sí misma, toda la oposición que el sistema necesita registrar.

El resultado es una señal social más limpia. El termómetro social mide qué ideas están reuniendo interés ciudadano, no qué ideas generan rechazo. Las propuestas que nadie apoya pierden visibilidad de forma natural gracias al ranking, sin necesidad de un mecanismo explícito de rechazo.

Esta lógica también refuerza el principio de que las propuestas deben evaluarse por su contenido y no por la identidad o alineamiento político de quien las impulsa. Sin voto negativo, desaparece la posibilidad de usar la plataforma como herramienta de castigo hacia determinadas ideas o sectores.

Los apoyos, además, son reversibles. Si un ciudadano cambia de opinión con el tiempo, puede retirar un apoyo previamente dado. Esta posibilidad es coherente con uno de los objetivos del sistema: permitir que las opiniones evolucionen naturalmente a lo largo del tiempo, algo que las encuestas tradicionales rara vez logran capturar.

## Límite anual de propuestas: evitar abuso y ruido político

La plataforma establece un límite anual de propuestas por ciudadano (por ejemplo, **2 por año**).

Este límite cumple un objetivo central: **evitar el abuso del sistema para generar ruido político, spam o campañas de provocación (trolls)**.

En un entorno abierto y anónimo, permitir la creación ilimitada de propuestas facilitaría que algunos actores utilicen la plataforma para inundarla con consignas partidarias, ataques políticos o contenido diseñado únicamente para generar confrontación. Ese tipo de comportamiento degradaría rápidamente el valor del sistema y desalentaría la participación de ciudadanos que desean aportar ideas genuinas.

El límite anual introduce un **costo de oportunidad**.  
Cada ciudadano dispone de pocas oportunidades para proponer ideas, por lo que se vuelve natural utilizarlas con cuidado y reservarlas para iniciativas que realmente considere importantes.

Este mecanismo también incentiva una conducta saludable dentro de la plataforma: antes de crear una propuesta nueva, el ciudadano tiene un incentivo real para **buscar si su idea ya existe y simplemente apoyarla**, reduciendo así la duplicación innecesaria.

De esta forma, el límite anual no busca restringir la participación, sino **proteger el espacio común** para que las propuestas que circulen tengan mayor probabilidad de representar inquietudes genuinas de la ciudadanía.

## Inmutabilidad de las propuestas

Una vez publicada, una propuesta no puede ser modificada.

Este criterio responde a un principio fundamental del sistema: preservar la integridad de la señal social generada por el apoyo ciudadano.

Si el contenido de una propuesta pudiera modificarse después de haber recibido apoyo, existiría la posibilidad de manipulación. Por ejemplo, una propuesta podría acumular apoyo bajo una idea determinada y luego cambiar su contenido por otro distinto.

Para evitar este tipo de situaciones, el texto de una propuesta que ha sido publicada permanece inmutable.

Sin embargo, esto no impide que las ideas evolucionen o se mejoren. Si el autor desea corregir la redacción, ampliar la propuesta o reformular su contenido, puede crear una **propuesta derivada** vinculada a la original.

Las propuestas derivadas comienzan sin apoyos, pero mantienen una referencia visible a la propuesta anterior. De esta manera, los ciudadanos pueden decidir si desean apoyar la nueva versión.

Este mecanismo permite mejorar las ideas sin comprometer la integridad de los apoyos ya emitidos.

## Buscador de propuestas: facilitar la convergencia de ideas

Antes de crear una propuesta, el ciudadano puede utilizar un buscador basado en palabras clave para revisar si su idea ya fue planteada por otra persona.

Esto ayuda a:

- reducir duplicaciones innecesarias  
- concentrar apoyos en propuestas existentes  
- mejorar la calidad del debate  

Sin embargo, la plataforma no bloquea la creación de propuestas similares.

Si alguien considera que una idea existente está incompleta o mal redactada, puede crear una nueva versión con mejoras.

Incluso puede referenciar la propuesta original en la que se inspiró.

Con el tiempo, las distintas versiones compiten entre sí y la comunidad termina impulsando la que considera más clara o más convincente.

## Duplicidad de propuestas y competencia natural

La plataforma no impide que existan propuestas similares o incluso duplicadas. Esta decisión es deliberada.

Si un ciudadano encuentra una propuesta cercana a su idea pero considera que puede expresarse mejor o incorporar mejoras, puede crear una nueva versión. Incluso puede referenciar la propuesta original como punto de partida.

A partir de ese momento ambas propuestas **compiten naturalmente por el apoyo de la ciudadanía**. Con el tiempo, las versiones que resulten más claras, razonables o representativas tenderán a concentrar la mayor cantidad de votos.

Este mismo mecanismo actúa como **filtro natural frente al ruido político**. Propuestas creadas únicamente para provocar, trolear o introducir consignas partidarias suelen recibir poco acompañamiento ciudadano y, al depender su visibilidad del nivel de apoyo, **terminan perdiendo relevancia de forma orgánica**, sin necesidad de moderación editorial.

El límite anual de propuestas (por ejemplo, **2 por ciudadano al año**) refuerza este comportamiento: al no poder crear propuestas ilimitadas, se reduce fuertemente el incentivo para inundar el sistema con contenido de baja calidad o provocaciones.

## Vinculación de propuestas

Los ciudadanos pueden vincular su propuesta con otras propuestas existentes dentro de la plataforma.

Estos vínculos permiten indicar que una propuesta mejora, amplía o integra ideas previamente publicadas. De esta manera, las propuestas no existen como elementos aislados, sino que pueden formar parte de una red de ideas relacionadas que evoluciona con el tiempo.

Por ejemplo, un ciudadano puede presentar una propuesta que refine o mejore una propuesta anterior, o bien una propuesta que busque resolver conjuntamente problemas planteados en varias propuestas existentes.

Este mecanismo facilita la evolución colectiva de ideas sin necesidad de intervención editorial por parte de la plataforma.

En el caso particular de las **propuestas derivadas**, la vinculación se declara al publicarlas. Cuando un ciudadano crea una propuesta a partir de otra existente, puede indicar esa relación y el sistema la registra de forma explícita, permitiendo a los ciudadanos identificar fácilmente el origen de la nueva propuesta y compararla con la versión anterior.

## Ausencia de moderación editorial

La plataforma **no** incorpora mecanismos formales de moderación editorial sobre las propuestas publicadas.

Es decir, el sistema no contempla perfiles de administradores o moderadores con facultades institucionales para eliminar propuestas, ocultarlas o decidir cuáles pueden permanecer visibles y cuáles no.

Esta decisión de diseño busca evitar que exista un punto central de control capaz de influir en la conversación pública dentro de la plataforma. Cuando una autoridad tiene la capacidad de decidir qué contenido permanece y cuál se elimina, inevitablemente aparece la sospecha de que esa facultad podría utilizarse con criterios políticos o ideológicos.

En cambio, la plataforma se apoya en un principio más simple: las propuestas compiten por la atención y el apoyo de los ciudadanos. Aquellas que reciben mayor acompañamiento tienden a ganar visibilidad, mientras que las que generan poco interés o son percibidas como ruido terminan perdiendo relevancia con el tiempo.

Desde luego, como en cualquier sistema informático, siempre existirán administradores con acceso técnico a la infraestructura. Sin embargo, el diseño de la plataforma evita institucionalizar ese poder dentro de la lógica del sistema, de modo que la visibilidad de las propuestas dependa principalmente de la participación ciudadana y no de decisiones editoriales.

## Decantación natural de propuestas

El ranking de propuestas se organiza principalmente según el nivel de apoyo recibido.

Esto genera un fenómeno natural:

- las propuestas con mayor interés social se vuelven más visibles  
- las propuestas irrelevantes, duplicadas o poco claras quedan progresivamente más abajo  

Con el paso del tiempo, el sistema tiende a ordenar las ideas según el nivel real de apoyo ciudadano.

## Incentivos a la participación constructiva

En el futuro, el sistema podría incorporar mecanismos simples de incentivo.

Por ejemplo, si una propuesta alcanza un nivel significativo de apoyo ciudadano, su autor podría recibir **una propuesta extra ese mismo año**, ampliando su cupo anual como reconocimiento al impacto de su iniciativa.

Este tipo de incentivos busca reconocer la participación constructiva y estimular la elaboración de ideas que generen consenso.

## Conclusión

Todas estas decisiones forman parte de una misma lógica: crear un espacio donde los ciudadanos puedan participar con libertad, pero donde las ideas compitan de manera abierta y transparente.

El resultado no pretende ser una medición perfecta de la opinión pública.

Más bien funciona como un **termómetro social**: una forma de observar qué temas despiertan interés real, qué propuestas logran reunir apoyo y qué preocupaciones comienzan a emerger dentro de la sociedad.
