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

Una vez completado ese paso, la identidad queda desacoplada del resto de la plataforma.

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

sin que nadie pueda vincular esas acciones con su identidad real.

La persistencia de esa identidad anónima también permite observar cómo evolucionan las opiniones con el tiempo, algo que las encuestas tradicionales rara vez capturan.

### Opinión política sin identidad política

Buena parte de la conversación pública digital actual está condicionada por la identidad visible de quien habla. Antes de evaluar una idea, muchas veces se evalúa a la persona, su grupo, su pertenencia o su posicionamiento previo.

La plataforma propone invertir esa lógica. En lugar de exponer una identidad política pública y luego opinar, el ciudadano participa mediante una identidad anónima persistente. Eso permite que las propuestas circulen con mayor independencia de la tribu, el cargo, el prestigio o el rechazo que suele despertar quien las enuncia.

El resultado buscado es simple: que las ideas puedan reunir apoyo o rechazo más por su contenido que por la identidad pública de quien las impulsa.

## Límite anual de propuestas: evitar abuso y ruido político

La plataforma establece un límite anual de propuestas por ciudadano (por ejemplo, **2 por año**).

Este límite cumple un objetivo central: **evitar el abuso del sistema para generar ruido político, spam o campañas de provocación (trolls)**.

En un entorno abierto y anónimo, permitir la creación ilimitada de propuestas facilitaría que algunos actores utilicen la plataforma para inundarla con consignas partidarias, ataques políticos o contenido diseñado únicamente para generar confrontación. Ese tipo de comportamiento degradaría rápidamente el valor del sistema y desalentaría la participación de ciudadanos que desean aportar ideas genuinas.

El límite anual introduce un **costo de oportunidad**.  
Cada ciudadano dispone de pocas oportunidades para proponer ideas, por lo que se vuelve natural utilizarlas con cuidado y reservarlas para iniciativas que realmente considere importantes.

Este mecanismo también incentiva una conducta saludable dentro de la plataforma: antes de crear una propuesta nueva, el ciudadano tiene un incentivo real para **buscar si su idea ya existe y simplemente apoyarla**, reduciendo así la duplicación innecesaria.

De esta forma, el límite anual no busca restringir la participación, sino **proteger el espacio común** para que las propuestas que circulen tengan mayor probabilidad de representar inquietudes genuinas de la ciudadanía.

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
