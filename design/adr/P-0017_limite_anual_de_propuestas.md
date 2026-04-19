# P-0017 — Límite anual de propuestas por ciudadano

## Contexto

La plataforma permite a cualquier ciudadano verificado crear propuestas. Sin un límite sobre la frecuencia de creación, un ciudadano podría inundar el sistema con propuestas de baja calidad, contenido provocador o consignas partidarias, degradando el valor del espacio común.

La documentación conceptual menciona un límite de 2 propuestas por año como ejemplo ilustrativo. Este ADR formaliza la existencia del límite, sus propiedades configurables y las reglas de conteo.

## Opciones consideradas

### Decisión 1 — Existencia del límite

#### Opción A — Sin límite

Cualquier ciudadano puede crear propuestas sin restricción de frecuencia.

Ventajas

- Sin fricción para el ciudadano.

Desventajas

- Permite inundar la plataforma con propuestas de baja calidad o contenido diseñado para provocar. El ranking mitiga la visibilidad de ese contenido, pero no el volumen de ruido que genera para quienes navegan la plataforma.

#### Opción B — Límite configurable por el operador

El sistema establece un límite anual de propuestas por ciudadano, configurable por el operador con un valor por defecto.

Ventajas

- Introduce un costo de oportunidad que incentiva al ciudadano a usar sus slots con cuidado.
- Desincentiva el uso de la plataforma como herramienta de spam o ruido político.
- Incentiva buscar propuestas existentes antes de crear una nueva.
- La configurabilidad permite adaptar el límite a distintos contextos de despliegue.

Desventajas

- Introduce fricción para ciudadanos con muchas ideas genuinas.

### Decisión 2 — Método de conteo del año

#### Opción A — Año calendario

El límite se reinicia el 1 de enero de cada año. Todos los ciudadanos tienen sus slots disponibles al mismo tiempo.

Ventajas

- Simple de entender y comunicar.

Desventajas

- Crea una ventana de abuso en diciembre: un ciudadano podría publicar varias propuesta antes del 31 diciembre y varias más el 1 de enero. Este comportamiento es completamente previsible y fácil de explotar sistemáticamente.

#### Opción B — Año móvil (rolling)

Cada propuesta publicada inicia un período de 365 días durante el cual ese slot queda consumido. El sistema muestra al ciudadano la fecha exacta en que cada slot vuelve a estar disponible.

Ventajas

- Elimina la distorsión del año calendario.
- El ciudadano siempre puede saber con precisión cuándo podrá crear su próxima propuesta.

Desventajas

- Más complejo de comunicar que el año calendario. Se resuelve mostrando fechas concretas en la interfaz.

### Decisión 3 — Las propuestas derivadas y el cupo

#### Opción A — Las propuestas derivadas no consumen cupo

Las propuestas derivadas se tratan como un mecanismo de mejora y no se descuentan del límite anual.

Ventajas

- No penaliza al ciudadano que quiere refinar una idea existente.

Desventajas

- Abre un vector de evasión: un ciudadano puede crear propuestas derivadas de forma indefinida, eludiendo el límite.

#### Opción B — Las propuestas derivadas consumen cupo

Toda propuesta publicada cuenta contra el límite anual, independientemente de si es original o derivada.

Ventajas

- Cierra el vector de evasión descrito en la opción A.
- Coherente con el principio de costo de oportunidad: una propuesta derivada es una propuesta nueva que entra al sistema y compite por apoyos.

Desventajas

- El ciudadano debe decidir con cuidado si prefiere crear una derivada o apoyar la propuesta original.

## Decisión

El sistema establece un límite anual de propuestas por ciudadano, configurable por el operador, con un valor por defecto de 2. El valor 0 es válido e indica que no existe límite.

El conteo usa año móvil: cada propuesta publicada consume un slot por 365 días contados desde su fecha de publicación. La interfaz muestra al ciudadano la fecha exacta en que cada slot vuelve a estar disponible.

Las propuestas derivadas consumen cupo en las mismas condiciones que las propuestas originales.

## Justificación

El límite existe porque el costo de oportunidad es el mecanismo central del sistema para desincentivar el spam y el ruido político. Sin él, el ranking mitiga la visibilidad del contenido de baja calidad pero no reduce el volumen de ruido que genera en la plataforma.

El año móvil se prefiere al año calendario porque el año calendario crea una distorsión predecible y fácilmente explotable: un ciudadano puede obtener varios slots en pocos días creando propuestas a fines de diciembre y principio de enero. El año móvil elimina esa asimetría sin complejidad técnica adicional; la complejidad comunicacional se resuelve mostrando fechas concretas.

Las propuestas derivadas consumen cupo porque son propuestas nuevas que entran al sistema y podrían usarse para evadir el límite si no se contaran. El mecanismo de propuestas derivadas existe para mejorar ideas, no para multiplicar slots.

La configurabilidad con valor 0 permite que despliegues académicos, de prueba o con contextos operativos distintos puedan desactivar el límite sin modificar la lógica del sistema.

## Consecuencias

- La plataforma lleva un registro de la fecha de publicación de cada propuesta por `anon_id`.
- Al intentar crear una propuesta, el sistema verifica si el ciudadano tiene slots disponibles según el año móvil y el límite configurado. Si no los tiene, informa la fecha del próximo slot disponible.
- La interfaz muestra al ciudadano su estado de cupo: cuántos slots tiene disponibles y, para los consumidos, cuándo se liberan.
- El valor por defecto de 2 debe documentarse en la guía de operación junto con los criterios para ajustarlo.
- Si el operador configura el límite en 0, el sistema no aplica ninguna restricción de frecuencia.
