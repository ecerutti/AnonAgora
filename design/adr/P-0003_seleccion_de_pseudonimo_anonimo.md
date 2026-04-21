# P-0003 — Selección del pseudónimo de identidad anónima

**Estado:** Activo

## Contexto

Cada ciudadano que se registra en la plataforma recibe una **identidad anónima persistente**.  
Como parte de esa identidad, el sistema genera un pseudónimo amigable (por ejemplo: *Lobo Azul 714*) que permite al usuario reconocer su identidad dentro del sistema sin exponer su identidad real.

Durante el diseño surgió la pregunta de si los ciudadanos deberían poder **elegir o modificar su pseudónimo**, o si el pseudónimo debería ser asignado automáticamente por el sistema y permanecer fijo.

Esta decisión afecta varios aspectos del sistema, entre ellos:

- la experiencia inicial de creación de identidad
- la posibilidad de personalización por parte del usuario
- el riesgo de que los pseudónimos transmitan información personal o ideológica
- la estabilidad de la identidad anónima en el tiempo

Además, dado que los pseudónimos se generan automáticamente mediante combinaciones de palabras (por ejemplo: *animal + color o adjetivo + número*), existe la posibilidad de que algunas combinaciones resulten poco agradables o inapropiadas desde el punto de vista del usuario.

Por ejemplo, combinaciones como:

*Víbora Rastrera 28*

podrían resultar desagradables para quien recibe ese pseudónimo.

Esto introdujo la necesidad de analizar cómo permitir cierto grado de elección sin comprometer los principios de anonimato del sistema.

## Opciones consideradas

### Opción 1 — Pseudónimo completamente automático e inmutable

En esta opción, el sistema genera automáticamente el pseudónimo y el usuario no tiene posibilidad de modificarlo.

Ejemplo:

*Lobo Azul 714*

El usuario simplemente recibe el pseudónimo asignado por el sistema.

**Ventajas**

- Implementación simple.
- Garantiza neutralidad en los pseudónimos.
- Evita que los usuarios introduzcan referencias personales o ideológicas en el nombre.

**Desventajas**

- Algunas combinaciones generadas automáticamente pueden resultar desagradables o poco adecuadas para el usuario.
- El usuario puede sentirse poco identificado con el pseudónimo asignado.
- Puede generar una experiencia inicial menos amigable.

### Opción 2 — Pseudónimo completamente editable por el usuario

En esta opción, el ciudadano podría elegir libremente su pseudónimo al crear su identidad anónima.

**Ventajas**

- Permite personalización completa por parte del usuario.
- Puede aumentar la sensación de pertenencia.

**Desventajas**

- Introduce riesgo de pseudónimos que revelen información personal.
- Posibilidad de pseudónimos con contenido político, ofensivo o ideológico.
- Los usuarios podrían elegir pseudónimos que ya utilizan en otras plataformas o redes sociales.
- Esto podría facilitar correlaciones entre la identidad anónima dentro de la plataforma y la identidad real del ciudadano.
- Incluso si el pseudónimo solo fuera visible para el propio usuario, un administrador con acceso al sistema podría intentar correlacionar datos utilizando ese alias.

### Opción 3 — Pseudónimo generado por el sistema con posibilidad de regeneración antes de aceptar

En esta opción, el sistema genera automáticamente un pseudónimo inicial, pero el usuario puede solicitar que el sistema genere uno nuevo antes de aceptarlo.

Por ejemplo, durante la creación de la identidad anónima el usuario podría ver:

*Tu identidad anónima será:*

*Lobo Azul 714*

y disponer de un botón para solicitar otro pseudónimo generado por el sistema.

El usuario puede repetir este proceso hasta encontrar un pseudónimo que le resulte adecuado.

Una vez aceptado, el pseudónimo queda asociado permanentemente a su identidad anónima.

**Ventajas**

- Mantiene la neutralidad del sistema en la generación de pseudónimos.
- Evita que los usuarios introduzcan nombres ideológicos o personales.
- Permite evitar combinaciones que resulten desagradables o poco apropiadas.
- Mejora la experiencia de creación de identidad.
- Reduce el riesgo de correlación con identidades externas.

**Desventajas**

- Requiere implementar un mecanismo de regeneración de pseudónimos.
- Introduce una pequeña complejidad adicional en el flujo de registro.

## Decisión

Se adopta la **Opción 3 — Pseudónimo generado por el sistema con posibilidad de regeneración antes de aceptar**.

Durante la creación de la identidad anónima, el sistema genera automáticamente un pseudónimo amigable compuesto por:

*Animal + Color o Adjetivo + Número*

El ciudadano puede solicitar que el sistema genere nuevos pseudónimos antes de aceptar uno.

Una vez que el usuario acepta el pseudónimo propuesto, este queda **asociado permanentemente a su identidad anónima** y no puede modificarse posteriormente.

El sistema **no permite que los ciudadanos escriban o definan libremente su propio pseudónimo**.

## Justificación

Esta solución busca equilibrar tres objetivos:

- preservar la neutralidad del sistema en la generación de identidades anónimas
- ofrecer una experiencia inicial más amigable para el usuario
- minimizar riesgos de correlación con identidades externas

Permitir que los ciudadanos definan libremente su pseudónimo podría introducir referencias personales, ideológicas o incluso aliases utilizados en otras plataformas.

Esto podría facilitar intentos de correlación entre la identidad anónima dentro de la plataforma y la identidad real del ciudadano.

En cambio, permitir que el sistema genere múltiples opciones mantiene el control sobre el formato de los pseudónimos mientras brinda al usuario cierto grado de elección.

La decisión de que el pseudónimo sea permanente una vez aceptado preserva la estabilidad de la identidad anónima dentro del sistema.

## Consecuencias

- El pseudónimo es generado automáticamente por el sistema.
- El usuario puede solicitar nuevas opciones antes de aceptar una.
- El usuario no puede escribir o definir libremente su pseudónimo.
- Una vez aceptado, el pseudónimo no puede modificarse.
- El pseudónimo permanece asociado permanentemente a la identidad anónima del ciudadano.
