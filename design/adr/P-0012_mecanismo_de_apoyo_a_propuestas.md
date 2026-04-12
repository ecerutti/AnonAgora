# P-0012 — Mecanismo de apoyo a propuestas

## Contexto

La plataforma permite a los ciudadanos expresar su posición respecto de las propuestas publicadas por otros. Este mecanismo es el núcleo del sistema: es la señal que alimenta el termómetro social y la que determina la visibilidad de las propuestas a través del ranking definido en `P-0010`.

El diseño del mecanismo de apoyo involucra varias preguntas interrelacionadas:

- ¿Qué tipos de posición puede expresar un ciudadano frente a una propuesta?
- ¿Un apoyo dado puede retirarse, o es definitivo?
- ¿Qué ocurre con la propuesta propia del autor al momento de publicarla?
- ¿Quién puede ver el conteo de apoyos de una propuesta?

Estas preguntas son distintas pero forman parte de una misma decisión de diseño: cómo se modela la participación de un ciudadano sobre una propuesta. Por ese motivo se resuelven conjuntamente en este ADR.

La decisión afecta directamente la simplicidad del sistema, su resistencia al abuso, la coherencia con el modelo conceptual del proyecto y las dinámicas sociales que la plataforma habilita o evita.

## Opciones consideradas

### Opción 1 — Apoyo binario irreversible

Un ciudadano puede apoyar una propuesta o no apoyarla. Una vez dado, el apoyo no puede retirarse.

**Ventajas**

- Máxima simplicidad en la mecánica.
- Las señales históricas son estables: lo que una propuesta acumuló nunca baja.

**Desventajas**

- Un apoyo dado por error queda registrado para siempre.
- Contradice un principio explícito de la documentación conceptual del proyecto, que celebra la posibilidad de que los ciudadanos cambien de opinión a lo largo del tiempo.
- Incoherente con el ranking definido en `P-0010`, que ya asume señales dinámicas mediante decaimiento temporal.

### Opción 2 — Apoyo binario retractable

Un ciudadano puede apoyar una propuesta o no apoyarla. Si apoyó, puede retirar su apoyo más adelante. Retirar no es una acción con peso propio: simplemente deja la propuesta en el mismo estado que si el ciudadano nunca la hubiera apoyado.

**Ventajas**

- Refleja naturalmente la evolución de opiniones en el tiempo.
- Permite corregir apoyos dados por error.
- Mantiene la simplicidad conceptual: el estado de un ciudadano respecto de una propuesta sigue siendo binario.
- Coherente con el ranking con decaimiento temporal de `P-0010` y con la documentación conceptual del proyecto.

**Desventajas**

- El conteo público de una propuesta puede bajar con el tiempo, lo que algunos observadores podrían percibir como inestabilidad. Sin embargo, esa variación es exactamente la señal que un termómetro social debería capturar.

### Opción 3 — Apoyo y rechazo (ternario)

Un ciudadano puede apoyar, rechazar, o permanecer neutral frente a una propuesta.

**Ventajas**

- Captura más información sobre la opinión ciudadana.

**Desventajas**

- Introduce el voto negativo como mecánica de primera clase, habilitando dinámicas de voto bronca, rechazo por afinidad partidaria o campañas de descalificación ideológica.
- Contradice el principio del sistema de que las propuestas deben evaluarse por su contenido y no por la identidad o alineamiento de quien las impulsa.
- Complica el ranking y la interpretación del termómetro social: ¿qué significa una propuesta con 500 apoyos y 500 rechazos?

### Opción 4 — Gradiente de apoyo (escala)

El ciudadano expresa su posición en una escala (por ejemplo, de 1 a 5 estrellas, o niveles tipo "apoyo fuerte / apoyo / neutral / ...").

**Ventajas**

- Captura matices de opinión.

**Desventajas**

- Introduce carga cognitiva para el ciudadano en cada interacción.
- Complica la interfaz y el cálculo del ranking.
- Los niveles inferiores de la escala reintroducen, de hecho, una forma de voto negativo por la puerta de atrás.
- Rompe la simplicidad conceptual del termómetro social.

## Decisión

Se adopta la **Opción 2 — Apoyo binario retractable**, junto con las siguientes reglas operativas:

### Naturaleza del apoyo

El apoyo es binario: un ciudadano apoya una propuesta o no la apoya. No existe voto negativo, rechazo ni escala. No hay forma dentro de la plataforma de expresar oposición a una propuesta.

### Retractabilidad

Un ciudadano puede retirar su apoyo a una propuesta en cualquier momento. Al retirarlo, la propuesta vuelve al estado en el que estaba antes de ese apoyo: el contador público disminuye en uno. No se registra ni se muestra ninguna señal adicional sobre el retiro.

### Apoyo del autor

Al momento de publicar una propuesta, el autor queda automáticamente registrado como un apoyo de esa propuesta. No se requiere una acción adicional para apoyarla.

El autor puede retirar su apoyo como cualquier otro ciudadano, siguiendo exactamente la misma mecánica. No existen apoyos especiales ni apoyos que no se puedan retirar. Una propuesta puede quedar con cero apoyos si el autor retira el suyo y nadie más la apoyó; la propuesta sigue existiendo en el sistema.

### Visibilidad del conteo

El conteo de apoyos de una propuesta es público. Todos los ciudadanos pueden ver cuántos apoyos tiene cada propuesta. Esto es coherente con el rol del conteo como corazón del termómetro social y con lo establecido en `P-0010`, que define el conteo de apoyos como una de las columnas visibles en la lista principal de propuestas.

El conteo visible es siempre el valor real, sin ponderación, tal como lo fija `P-0010`.

## Justificación

La decisión responde a tres objetivos principales.

**Simplicidad mecánica.** Un estado binario retractable es el modelo más simple posible que a la vez refleja evolución temporal de opiniones. Tratar el apoyo del autor como un apoyo más (en lugar de un caso especial) refuerza esa simplicidad: toda la plataforma opera con una sola regla.

**Resistencia al abuso y dinámicas saludables.** La ausencia de voto negativo es deliberada. Un mecanismo de rechazo habilitaría comportamientos que el sistema busca explícitamente evitar: voto bronca, rechazo partidario, campañas de descalificación ideológica. Al ofrecer únicamente apoyo como forma de participación, las propuestas compiten por atraer acompañamiento en lugar de por evitar rechazo, y el termómetro social mide qué ideas reúnen interés en lugar de qué ideas generan oposición.

**Coherencia con decisiones previas.** La retractabilidad es coherente con `P-0010`, que define un ranking basado en decaimiento temporal y asume señales dinámicas. También es coherente con el modelo conceptual del proyecto, que celebra la posibilidad de que los ciudadanos cambien de opinión como una ventaja del anonimato persistente frente a las encuestas tradicionales.

La fundamentación conceptual más amplia de por qué el sistema no incorpora voto negativo se desarrolla en `docs/propuesta/02_Fundamentos_y_Lógica_de_funcionamiento.md`.

## Consecuencias

- Cada ciudadano tiene en todo momento, para cada propuesta, un estado binario: la apoya o no la apoya.
- Retirar un apoyo disminuye el contador público de la propuesta en uno, sin generar registro visible adicional.
- El sistema no expone en ninguna interfaz un concepto de "rechazo", "oposición", "desaprobación" ni equivalentes.
- Al crear una propuesta, el sistema registra automáticamente un apoyo del autor. La propuesta nace con un apoyo.
- Un autor puede retirar el apoyo de su propia propuesta. Una propuesta puede existir con cero apoyos.
- El conteo público de apoyos refleja siempre el valor real, sin ponderación, consistente con `P-0010`.
- Cualquier decisión futura sobre la visualización, el ranking o el análisis de propuestas debe operar sobre este modelo binario retractable.
