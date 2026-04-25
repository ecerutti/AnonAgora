# P-0011 — Revisor automático de lenguaje en propuestas

**Estado:** Activo

## Contexto

La plataforma permite a cualquier ciudadano verificado publicar propuestas
de forma libre. Este modelo abierto introduce el riesgo de que algunas
propuestas contengan lenguaje ofensivo, discriminatorio, violento o
inapropiado que degrade la calidad del espacio participativo y dificulte
que las ideas sean leídas y consideradas seriamente por otros ciudadanos
y por los destinatarios institucionales de la plataforma.

El sistema debe incorporar un mecanismo que detecte este tipo de lenguaje
antes de la publicación y solicite al autor que revise su redacción, sin
intervenir sobre el contenido ideológico de las propuestas ni generar
censura política.

El principio rector es moderar la forma, no el contenido. El objetivo no
es filtrar ideas sino promover un estándar mínimo de expresión que permita
que cualquier propuesta, independientemente de su orientación ideológica,
pueda ser leída y considerada en igualdad de condiciones.

## Opciones consideradas

### Opción 1 — Lista de palabras prohibidas

El sistema mantiene una lista curada de palabras y expresiones prohibidas.
Si el texto de una propuesta contiene alguna de ellas, se solicita
corrección antes de publicar.

**Ventajas**

- Simple de implementar y completamente local, sin dependencia de
  servicios externos.
- Comportamiento predecible y auditable: la lista es pública y
  verificable.
- Sin latencia adicional ni costos por llamadas a APIs externas.

**Desventajas**

- Muy fácil de eludir mediante ofuscación deliberada: m1erda, m.i.e.r.d.a,
  mjerd4, etc.
- Requiere mantenimiento continuo de la lista para cubrir nuevas
  expresiones y variantes.
- Alta tasa de falsos positivos: puede bloquear palabras legítimas que
  contienen secuencias prohibidas.
- No detecta lenguaje ofensivo contextual que no usa palabras explícitas.

### Opción 2 — Modelo de moderación mediante API externa

El texto de la propuesta se envía a un servicio externo de moderación
basado en inteligencia artificial, que analiza el contenido y devuelve
una evaluación por categorías de riesgo.

**Ventajas**

- Alta capacidad de detección, incluyendo lenguaje ofensivo contextual
  y variantes ofuscadas cuando se combina con normalización previa.
- No requiere mantenimiento de listas de palabras.
- Mejora continua del modelo por parte del proveedor del servicio.
- Devuelve scores de probabilidad por categoría, permitiendo ajustar
  umbrales de sensibilidad.

**Desventajas**

- Dependencia de un servicio externo: disponibilidad, latencia y
  condiciones de uso quedan fuera del control de la plataforma.
- El texto de la propuesta sale del sistema antes de ser publicado,
  lo que introduce un riesgo de privacidad que debe evaluarse y
  documentarse explícitamente.

### Opción 3 — Modelo de moderación local

Se despliega un modelo de inteligencia artificial de moderación
directamente en la infraestructura de la plataforma, sin enviar datos
a servicios externos.

**Ventajas**

- Sin dependencia de servicios externos.
- Sin riesgo de privacidad asociado al envío de datos a terceros.
- Control total sobre el modelo y su comportamiento.

**Desventajas**

- Mayor complejidad operativa: requiere recursos de infraestructura
  adicionales para correr el modelo.
- Los modelos locales disponibles tienen menor capacidad de detección
  que los modelos de frontera ofrecidos por servicios externos.
- Requiere actualización y mantenimiento del modelo por parte del
  equipo operativo.

## Decisión

Se adopta la **Opción 2**, utilizando la API de moderación de OpenAI
(`omni-moderation-latest`) como servicio externo de detección.

### Categorías activadas

La API devuelve scores por categoría. Se activan las siguientes:

| Categoría | Descripción | Activada |
|-----------|-------------|----------|
| `harassment` | Lenguaje agresivo o acosador hacia cualquier persona | Sí |
| `harassment/threatening` | Amenazas directas | Sí |
| `hate` | Discurso de odio por raza, género, religión, orientación sexual u otras características | Sí |
| `hate/threatening` | Discurso de odio que incluye amenazas | Sí |
| `violence` | Contenido que promueve o describe violencia | Sí |
| `violence/graphic` | Descripciones gráficas de violencia | Sí |
| `illicit` | Instrucciones o consejos para cometer actos ilegales | Sí |
| `illicit/violent` | Instrucciones para actos ilegales que incluyen violencia | Sí |
| `sexual` | Contenido sexualmente explícito | No |
| `sexual/minors` | Contenido sexual que involucra menores | No |
| `self-harm` | Contenido que promueve autolesiones | No |
| `self-harm/intent` | Expresión de intención de autolesión | No |
| `self-harm/instructions` | Instrucciones para autolesionarse | No |

Las categorías `sexual`, `sexual/minors` y las de `self-harm` se
desactivan porque el contexto de uso de la plataforma —propuestas
ciudadanas de carácter político y social— hace que su aparición sea
improbable y su activación genere falsos positivos en propuestas
legítimas sobre educación, salud pública o derechos civiles.

### Normalización previa

Antes de enviar el texto a la API, el sistema aplica una capa de
normalización para mejorar la detección de lenguaje ofuscado
deliberadamente. Esta normalización resuelve sustituciones comunes
como `@` por `a`, `1` o `!` por `i`, `3` por `e`, `4` por `a`,
`0` por `o`, puntos o guiones intercalados entre letras, y
mayúsculas intercaladas para disfrazar palabras.

El sistema envía a la API tanto el texto original como el texto
normalizado, permitiendo que el modelo tome decisiones con mayor
contexto.

La capa de normalización se implementa usando bibliotecas maduras
existentes, sin desarrollar soluciones propias.

### Comunicación al ciudadano

Cuando una propuesta es rechazada por el revisor, el sistema muestra
el siguiente mensaje, seguido de la categoría detectada expresada en
términos ciudadanos:

> "Tu propuesta es importante. Para que pueda ser publicada y leída
> correctamente por otros ciudadanos y autoridades, por favor revisá
> y corregí el uso de lenguaje ofensivo o inapropiado detectado."
>
> *Aspecto a revisar: [categoría en lenguaje ciudadano]*

Las categorías se comunican al ciudadano en los siguientes términos:

| Categoría técnica | Texto mostrado al ciudadano |
|-------------------|-----------------------------|
| `harassment`, `harassment/threatening` | Lenguaje agresivo o amenazante |
| `hate`, `hate/threatening` | Lenguaje discriminatorio |
| `violence`, `violence/graphic` | Contenido que promueve o describe violencia |
| `illicit`, `illicit/violent` | Instrucciones para actividades ilegales |

El ciudadano puede realizar tantos intentos de corrección como
necesite antes de publicar. No existe límite de intentos.

### Riesgo de privacidad asumido

El envío del texto de las propuestas a la API de OpenAI antes de su
publicación implica que ese contenido sale temporalmente del sistema.
Este riesgo se evalúa como aceptable por las siguientes razones:

- El texto enviado corresponde a propuestas ciudadanas de carácter
  público, no a datos de identidad ni información personal sensible.
- Según la política de datos de OpenAI para uso de API, los datos
  enviados no se utilizan para entrenar modelos y se retienen por un
  máximo de 30 días con fines de monitoreo de abuso, tras lo cual
  son eliminados.
- OpenAI recibe volúmenes masivos de texto de miles de aplicaciones
  distintas, lo que hace prácticamente imposible asociar un texto
  específico con un ciudadano particular de esta plataforma.
- El texto de la propuesta, una vez publicada, será de todas formas
  público dentro de la plataforma.

Este riesgo debe mencionarse en la documentación pública de la
plataforma para que los operadores que la desplieguen puedan
evaluarlo en su propio contexto institucional y legal.

## Justificación

La lista de palabras prohibidas es insuficiente frente a la ofuscación
deliberada y requiere mantenimiento continuo. Un modelo local ofrece
privacidad total pero a costa de mayor complejidad operativa y menor
capacidad de detección. La API de moderación de OpenAI ofrece el mejor
equilibrio entre capacidad de detección, simplicidad de integración y
costo operativo, siendo además gratuita.

El principio rector de moderar la forma y no el contenido se garantiza
mediante la selección cuidadosa de categorías: se activan únicamente
aquellas que detectan lenguaje que degrada la expresión independientemente
de la ideología, y se desactivan las que podrían interferir con propuestas
legítimas sobre temas sociales sensibles.

## Consecuencias

- El sistema requiere una clave de API de OpenAI para el servicio de
  moderación.
- Toda propuesta pasa por el revisor antes de ser publicada.
- El operador puede ajustar los umbrales de sensibilidad por categoría
  según el contexto de despliegue.
- La guía de instalación y operación de la plataforma debe documentar
  el riesgo de privacidad asociado al uso de la API externa y las
  condiciones bajo las cuales ese riesgo se considera aceptable.
- El sistema no puede detectar sarcasmo, ironía ni formas indirectas
  de expresión ofensiva. Esta limitación es conocida y aceptada.
