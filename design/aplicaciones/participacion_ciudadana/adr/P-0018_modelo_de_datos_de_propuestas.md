# P-0018 — Modelo de datos de propuestas

**Estado:** Activo

## Contexto

La plataforma permite a ciudadanos verificados publicar propuestas de participación ciudadana. Este ADR define la estructura de datos de una propuesta, el formato admitido para su contenido, los límites configurables aplicables, y el modelo de registro de autoría compatible con el principio de minimización de datos.

Las decisiones sobre vinculación entre propuestas se tratan en un ADR separado. Las decisiones sobre apoyos están definidas en P-0012. Las decisiones sobre ranking y score están definidas en P-0010. Las decisiones sobre el límite anual de publicaciones están definidas en P-0017.

Las preguntas de diseño que motivaron este ADR son:

- ¿Qué campos tiene una propuesta?
- ¿Debe el sistema asociar internamente una propuesta con su autor?
- ¿Qué formato admite el cuerpo de la propuesta?
- ¿Admite la propuesta imágenes o links externos?
- ¿Qué límites de longitud aplican?

## Opciones consideradas

### Decisión 1 — Asociación entre propuesta y autor

P-0001 establece que las propuestas no muestran ningún identificador del autor frente a otros ciudadanos. Sin embargo, el sistema necesita registrar que un `anon_id` publicó una propuesta en una fecha determinada para aplicar el año móvil de P-0017. La pregunta es si ese registro debe vivir dentro de la fila de la propuesta o en una estructura separada.

#### Opción A — `anon_id` almacenado en la propuesta

El autor queda registrado como un campo de la propuesta.

Ventajas

- Implementación simple.
- Permite auditar fácilmente qué propuestas publicó un ciudadano.

Desventajas

- Un atacante con acceso a la base de datos puede listar todas las propuestas asociadas a un `anon_id`, reconstruyendo el historial de participación del ciudadano.
- Contradice el principio de minimización de datos: la propuesta no necesita saber quién la escribió para cumplir ninguna función dentro del sistema una vez publicada.

#### Opción B — `anon_id` desasociado de la propuesta, registro en tabla de eventos separada

La propuesta no almacena ninguna referencia a su autor. Al momento de publicación, el sistema registra en una tabla de eventos independiente `{anon_id, fecha_publicacion}`, utilizada exclusivamente para el control del año móvil de P-0017. La propuesta y ese registro son estructuras independientes y no vinculables entre sí.

Ventajas

- Un atacante con acceso a la base de datos de propuestas no puede determinar quién escribió qué.
- La tabla de eventos de publicación no expone qué propuesta fue publicada, solo que hubo una publicación en una fecha.
- Coherente con P-0001 y con el principio de minimización de datos.

Desventajas

- Implementación levemente más compleja: requiere una tabla adicional.
- El sistema no puede responder consultas del tipo "¿qué propuestas publicó este ciudadano?" porque esa información no existe.

### Decisión 2 — Formato del cuerpo de la propuesta

#### Opción A — Texto plano

El cuerpo de la propuesta admite únicamente texto sin formato.

Ventajas

- Implementación simple.
- Sin riesgo de abuso de formato para manipular la presentación visual.
- El revisor de lenguaje trabaja sobre texto limpio.

Desventajas

- No permite estructurar propuestas complejas con títulos, subtítulos, listas, referencias o énfasis.
- Limita la calidad de propuestas de carácter legislativo o técnico que requieren estructura formal.

#### Opción B — Markdown

El cuerpo de la propuesta se almacena en formato Markdown. La interfaz de redacción provee un editor amigable que gestiona el Markdown de forma transparente para el ciudadano, sin requerir conocimiento del formato.

Markdown permite títulos, subtítulos, listas, negritas, itálicas, texto tachado, notas, referencias y otros elementos de estructura que habilitan propuestas de calidad legislativa o técnica.

Ventajas

- Habilita propuestas bien estructuradas comparables a un proyecto de ley o documento técnico formal.
- Ampliamente soportado en herramientas de renderizado.
- La decisión de formato es del almacenamiento; la experiencia del ciudadano queda en manos de la interfaz.

Desventajas

- Requiere sanitización del Markdown renderizado para evitar abuso de formato en la presentación.
- El revisor de lenguaje debe aplicarse sobre el texto plano extraído del Markdown, no sobre el Markdown crudo.

### Decisión 3 — Imágenes

#### Opción A — Sin imágenes

La propuesta no admite imágenes adjuntas ni embebidas.

Ventajas

- Elimina el riesgo de publicación de imágenes con contenido inapropiado, pornográfico, violento o partidario, que el revisor automático de lenguaje no puede detectar.
- Elimina el riesgo de metadatos EXIF embebidos en imágenes que podrían revelar información del dispositivo o ubicación del autor.
- Sin necesidad de almacenamiento adicional ni moderación de contenido visual.

Desventajas

- Limita la capacidad de ilustrar propuestas con mapas, gráficos o documentos visuales.

#### Opción B — Con imágenes

La propuesta admite imágenes adjuntas.

Ventajas

- Permite ilustrar propuestas con material visual de apoyo.

Desventajas

- Requiere moderación de contenido visual, que el revisor de lenguaje actual no cubre.
- Introduce riesgo de metadatos EXIF con información sensible del autor.
- Aumenta la complejidad de almacenamiento y la superficie de abuso.

### Decisión 4 — Links externos

#### Opción A — Sin links

La propuesta no admite URLs.

Desventajas

- Impide citar fuentes externas relevantes como artículos de ley, estadísticas o documentos de referencia.

#### Opción B — Links renderizados como hipervínculos clickeables

Las URLs se renderizan como enlaces activos.

Desventajas

- Introduce riesgo de phishing y desvío de atención fuera de la plataforma.

#### Opción C — Links presentes pero renderizados como texto plano

Las URLs pueden incluirse en el texto y se almacenan tal cual, pero se renderizan como texto no clickeable. El ciudadano puede copiar la URL manualmente si desea visitarla.

Ventajas

- Permite citar fuentes externas sin facilitar activamente la navegación fuera de la plataforma.
- Elimina el riesgo de phishing por links embebidos.

Desventajas

- Menor comodidad para el lector que quiera consultar una fuente citada.

## Decisión

### Estructura de la propuesta

Una propuesta almacena los siguientes campos:

- `id` — identificador único de la propuesta, generado por el sistema
- `titulo` — texto plano, longitud máxima configurable por el operador, valor por defecto 200 caracteres
- `cuerpo` — texto en formato Markdown, longitud máxima configurable por el operador, valor por defecto 20.000 caracteres
- `fecha_publicacion` — timestamp del momento de publicación
- `conteo_apoyos` — entero actualizado por el sistema según P-0012
- `score` — valor calculado por el sistema según P-0010
- `vinculos` — referencias a otras propuestas, definidas en ADR posterior

La propuesta no almacena ninguna referencia a su autor.

### Formato

El cuerpo se almacena en Markdown.

Las propuestas no admiten imágenes.

Los links externos pueden incluirse en el texto y se almacenan tal cual, pero se renderizan siempre como texto plano no clickeable.

## Justificación

La desasociación entre propuesta y autor es la consecuencia directa de P-0001 llevada a su conclusión lógica: si la propuesta no debe mostrar al autor, tampoco debe almacenarlo. La propuesta publicada no necesita saber quién la escribió para cumplir ninguna función dentro del sistema.

Markdown se elige porque el objetivo de la plataforma no es solo recibir ideas simples sino también propuestas de calidad comparable a un proyecto de ley. Limitar el formato a texto plano reduciría innecesariamente la calidad posible de las propuestas. La decisión de formato es del almacenamiento; la experiencia del ciudadano queda en manos del editor de interfaz, que puede ser tan amigable como se requiera.

Las imágenes se excluyen porque el revisor de lenguaje no cubre contenido visual y el riesgo de abuso es alto y sin contramedida técnica disponible en el diseño actual.

Los links se permiten como texto plano porque citar fuentes es una práctica legítima en propuestas serias, pero renderizarlos como hipervínculos activos introduce riesgo de phishing sin beneficio proporcional.

## Consecuencias

- La base de datos de propuestas no contiene información sobre autoría. No es posible responder consultas del tipo "¿qué propuestas publicó este ciudadano?" a partir de la tabla de propuestas.
- El campo `conteo_apoyos` se mantiene desnormalizado por razones de performance. El registro de apoyos individuales es la fuente de verdad: ante cualquier desfase detectado entre el contador y el registro, el valor correcto es el derivado del registro. La implementación concreta del mantenimiento de la consistencia (transaccionalidad, reconciliación, mecanismo de detección de desfase) se documenta en la carpeta de implementación de la aplicación cuando se aborde esa etapa.
- Los valores por defecto de longitud máxima de título y cuerpo deben documentarse en la guía de operación junto con los criterios para ajustarlos.
