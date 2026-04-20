# P-0019 — Búsqueda y filtrado de propuestas

## Contexto

La plataforma permite a cualquier ciudadano explorar el conjunto de propuestas publicadas. A medida que el volumen de propuestas crece, navegar la lista principal ordenada por relevancia puede ser insuficiente para encontrar propuestas específicas. El ciudadano necesita poder buscar por contenido y filtrar por características estructurales de las propuestas.

La documentación conceptual menciona un buscador basado en palabras clave como mecanismo para que el ciudadano verifique si su idea ya fue planteada antes de crear una propuesta nueva. Este ADR formaliza las capacidades de búsqueda y filtrado del sistema.

Las preguntas de diseño que motivaron este ADR son:

- ¿Sobre qué campos opera la búsqueda por texto?
- ¿Qué tan sofisticado es el motor de búsqueda?
- ¿Qué filtros estructurados ofrece el sistema?

## Opciones consideradas

### Decisión 1 — Alcance de la búsqueda por texto

#### Opción A — Solo título

La búsqueda opera únicamente sobre el título de la propuesta.

Ventajas

- Implementación simple.
- Resultados más acotados y precisos.

Desventajas

- Una propuesta con contenido relevante en el cuerpo pero con un título que no contiene el término buscado no aparecería en los resultados.

#### Opción B — Título y cuerpo

La búsqueda opera sobre el título y el cuerpo de la propuesta, con relevancia ponderada: las coincidencias en el título pesan más que las coincidencias en el cuerpo.

Ventajas

- Cubre más casos de uso: propuestas cuyo contenido relevante está desarrollado en el cuerpo y no solo en el título.
- Reduce el riesgo de que el ciudadano no encuentre una propuesta.

Desventajas

- Puede traer resultados menos precisos si el término buscado aparece en el cuerpo de propuestas poco relacionadas.

### Decisión 2 — Motor de búsqueda

#### Opción A — Búsqueda normalizada

La búsqueda es insensible a mayúsculas y acentos, pero busca el texto literalmente. "jubilacion" encuentra "jubilación" y "Jubilación", pero no "jubilaciones" ni "jubilado".

Ventajas

- Simple de implementar.
- Comportamiento predecible para el ciudadano.

Desventajas

- El ciudadano que escribe "jubilaciones" no encuentra propuestas que usan "jubilación", y viceversa.

#### Opción B — Full-text con morfología y stopwords

El motor maneja variantes morfológicas de la misma raíz y elimina palabras vacías del español ("el", "de", "en", "la") antes de buscar. "jubilacion" encuentra "jubilación", "jubilaciones", "jubilado", "jubilados". Una búsqueda como "el sistema de jubilaciones" busca efectivamente "sistema jubilaciones".

Ventajas

- El ciudadano no necesita escribir la forma exacta de una palabra para encontrar propuestas relevantes.
- Las stopwords no interfieren con los resultados ni requieren que el ciudadano las evite.

Desventajas

- Depende de un tokenizador con soporte para español, disponible de forma nativa en los motores de base de datos principales.
- Puede traer más resultados que la búsqueda exacta, aunque la ponderación por relevancia mitiga este efecto.

#### Opción C — Búsqueda semántica

El sistema usa embeddings para encontrar propuestas similares conceptualmente aunque no compartan palabras. "colectivo" puede encontrar propuestas que usan "autobús" o "transporte público".

Ventajas

- Encuentra propuestas relacionadas conceptualmente aunque usen vocabulario distinto.

Desventajas

- Introduce complejidad considerable: requiere modelos de embeddings, infraestructura adicional y dependencia de servicios externos o modelos locales.
- El comportamiento es menos predecible para el ciudadano: puede ser difícil entender por qué ciertos resultados aparecen.

## Decisión

### Búsqueda por texto

La búsqueda opera sobre título y cuerpo de la propuesta, con relevancia ponderada a favor del título. Se adopta full-text con morfología y stopwords en español. La búsqueda es insensible a mayúsculas y acentos.

### Filtros estructurados

El sistema ofrece los siguientes filtros, combinables entre sí y con la búsqueda por texto:

- **Emergente:** propuestas que cumplen el criterio de emergente según P-0010.
- **Tendencia:** propuestas que cumplen el criterio de tendencia según P-0010.
- **Cantidad de vínculos:** propuestas con menos de, exactamente, o más de una cantidad de vínculos determinada.
- **Vínculos a propuestas específicas:** propuestas que referencian a una o más propuestas indicadas por el ciudadano. Cuando se indican múltiples, el filtro devuelve propuestas que referencian a todas ellas simultáneamente (AND).
- **Rango de fechas:** propuestas publicadas antes de, después de, fecha específica, o entre dos fechas.
- **Rango de apoyos:** propuestas con menos de, exactamente, o más de una cantidad de apoyos determinada.
- **Apoyadas por el ciudadano / no apoyadas:** propuestas que el ciudadano autenticado apoyó o no apoyó. Este filtro requiere sesión activa y opera sobre el historial de apoyos del propio ciudadano.

## Justificación

La búsqueda sobre título y cuerpo responde al propósito central del buscador: que el ciudadano pueda verificar si su idea ya fue planteada. Limitar la búsqueda al título haría que propuestas con contenido relevante en el cuerpo no aparecieran en los resultados, frustrando ese propósito.

Full-text con morfología y stopwords se prefiere sobre búsqueda normalizada porque el ciudadano no debería necesitar conocer la forma exacta de una palabra para encontrar propuestas relevantes. "jubilacion" debe encontrar "jubilaciones" y "jubilado" porque refieren a la misma idea. Las stopwords evitan que palabras vacías interfieran con los resultados sin requerir que el ciudadano las evite manualmente. La búsqueda semántica fue descartada por complejidad desproporcionada al caso de uso.

Los vínculos se tratan como filtro estructurado y no como campo de búsqueda por texto porque son identificadores, no contenido semántico. Mezclar búsqueda de texto libre con búsqueda por ID en el mismo campo introduce ambigüedad y requiere que el ciudadano aprenda una sintaxis especial. Un filtro explícito es más claro y menos propenso a errores.

El filtro por apoyos propios no introduce conflicto de privacidad porque el ciudadano consulta exclusivamente su propio historial, información que el sistema ya asocia a su sesión activa y que le es útil para gestionar sus apoyos a lo largo del tiempo.

## Consecuencias

- El motor de búsqueda debe soportar full-text con morfología y stopwords en español.
- La búsqueda pondera relevancia a favor del título sobre el cuerpo.
- Los filtros son combinables entre sí y con la búsqueda por texto.
- El filtro por apoyos propios requiere que el ciudadano tenga sesión activa; no está disponible para visitantes no autenticados.
- La interfaz decide cómo presentar la búsqueda y los filtros al ciudadano; este ADR no prescribe si se presentan como herramientas separadas o unificadas.
