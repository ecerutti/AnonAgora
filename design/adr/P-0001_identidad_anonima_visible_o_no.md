# P-0001 — Visibilidad pública de la identidad anónima

## Contexto

Uno de los principios centrales de la plataforma es que cada ciudadano recibe una **identidad anónima persistente** que le permite participar en el sistema sin revelar su identidad real.

Durante el diseño del sistema surgió una decisión importante: determinar si esa identidad anónima debía ser **visible públicamente dentro de la plataforma** o si debía permanecer **visible únicamente para el propio usuario**.

La decisión afecta varios aspectos del funcionamiento del sistema, entre ellos:

- la dinámica social entre participantes
- la posibilidad de generar reputación dentro de la plataforma
- el riesgo de correlación o deanonimización de usuarios
- el tipo de incentivos que se generan para la participación

Por lo tanto, fue necesario analizar distintas alternativas.

## Opciones consideradas

### Modelo 1 — Identidad pseudónima pública persistente

En este modelo, cada ciudadano recibe un alias pseudónimo visible públicamente dentro de la plataforma.

Ejemplos de alias posibles:

*Anon#1729*  
*User-8F3A91*  
*Lobo Azul 714*  

Todas las propuestas y participaciones del usuario quedarían asociadas a ese alias.

**Ventajas**

- Permite generar reputación dentro de la plataforma.
- Incentiva a algunos usuarios a producir propuestas de mayor calidad.
- Permite seguir a autores interesantes.
- Favorece la sensación de comunidad entre participantes.

**Desventajas**

- Introduce la posibilidad de correlación entre distintas acciones de un mismo usuario.
- Si un alias llegara a asociarse con una persona real (por filtración o auto-revelación), todo el historial de actividad de ese usuario quedaría expuesto.
- Puede generar dinámicas de seguidores o liderazgo informal dentro de la plataforma.
- Puede introducir sesgos ideológicos: los ciudadanos podrían tender a apoyar o rechazar propuestas según el historial del autor, en lugar de evaluar cada propuesta por su contenido.

### Modelo 2 — Anonimato total sin identidad visible

En este modelo, los ciudadanos no tienen una identidad pseudónima dentro de la plataforma.

Las identidades anónimas otorgadas se manejarían de manera interna dentro de la plataforma con identificadores invisibles para los usuarios.

**Ventajas**

- Maximiza el anonimato entre participantes.
- Elimina completamente la posibilidad de construir perfiles de usuarios dentro de la plataforma.

**Desventajas**

- El usuario no tiene ninguna forma visible de reconocer su propia identidad dentro del sistema.
- Se pierde la noción de continuidad de participación.
- Puede generar confusión para el propio usuario respecto a su identidad dentro de la plataforma.

### Modelo 3 — Identidad anónima visible solo para el propio usuario

En este modelo, el sistema genera un alias pseudónimo para cada ciudadano (por ejemplo: *Lobo Azul 714*), pero ese alias **solo es visible para el propio usuario dentro de su sesión**.

El alias puede aparecer, por ejemplo, en el encabezado de la interfaz:

`Participando como: Lobo Azul 714`

Sin embargo, las propuestas publicadas en la plataforma **no muestran ningún identificador del autor**.

Desde el punto de vista de otros ciudadanos, las propuestas aparecen simplemente como contenido anónimo.

**Ventajas**

- Permite que el ciudadano reconozca su identidad anónima persistente dentro del sistema.
- Evita la creación de reputaciones públicas o perfiles rastreables.
- Reduce el riesgo de correlación entre distintas acciones de un mismo usuario.
- Evita la aparición de influencers o líderes informales dentro de la plataforma.
- Incentiva que las propuestas se evalúen por su contenido y no por la identidad del autor.

**Desventajas**

- No existe reputación pública entre participantes.
- No es posible seguir autores específicos dentro de la plataforma.
- Se pierde parte del incentivo de reconocimiento personal asociado a la autoría.

## Decisión

Se adopta el **Modelo 3 — Identidad anónima visible solo para el propio usuario**.

El sistema genera un alias pseudónimo para cada identidad anónima, pero dicho alias **no es visible públicamente dentro de la plataforma**.

Las propuestas, votos y demás interacciones no exponen ningún identificador del autor frente a otros ciudadanos.

## Justificación

La plataforma está concebida principalmente como un **termómetro social**, orientado a medir el nivel de apoyo que distintas ideas reciben dentro de la ciudadanía.

El objetivo no es construir una red social de autores ni incentivar la formación de liderazgos individuales dentro del sistema.

Mostrar identidades pseudónimas persistentes podría favorecer dinámicas de seguidores, polarización ideológica o sesgos hacia determinados autores, lo que distorsionaría la señal social que la plataforma busca capturar.

Al ocultar las identidades pseudónimas frente a otros participantes:

- las propuestas tienden a evaluarse por su contenido
- se reduce el riesgo de formación de identidades políticas persistentes dentro del sistema
- se limita la posibilidad de correlación pública entre acciones de un mismo usuario

Al mismo tiempo, el alias visible para el propio usuario preserva la sensación de continuidad de su identidad anónima dentro del sistema.

## Consecuencias

- Las propuestas publicadas en la plataforma no muestran información sobre el autor.
- Los alias pseudónimos existen únicamente como representación interna de la identidad anónima del ciudadano.
- El sistema evita deliberadamente mecanismos que fomenten reputación pública de usuarios dentro de la plataforma.
