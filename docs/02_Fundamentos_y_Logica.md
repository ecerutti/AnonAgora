# Documento Ampliatorio

## Funcionamiento del sistema y mitigación de riesgos

(Este documento amplía la propuesta introductoria. Está orientado a decisores políticos y asesores no técnicos que desean comprender con mayor profundidad cómo funciona el sistema, qué riesgos existen y cómo se mitigan, sin entrar en detalles criptográficos complejos.)

## 1. Propósito de este documento

Este texto tiene como objetivo profundizar la propuesta de Plataforma de Participación Ciudadana Digital con Identidad Anónima Persistente, explicando:

- cómo funciona el mecanismo de participación,
- cómo se protege el anonimato de los ciudadanos,
- qué riesgos o críticas pueden surgir,
- y cómo esos riesgos se mitigan desde el diseño.

No busca reemplazar el documento introductorio, sino complementarlo para quienes desean una comprensión más completa antes de avanzar.

## 2. Principio central: identidad verificada, participación anónima

El sistema se apoya en un principio simple pero clave:

> **El Estado verifica que una persona es un ciudadano real, pero luego pierde toda capacidad de saber qué hace ese ciudadano dentro de la plataforma.**

La verificación de identidad y la participación posterior están deliberadamente desacopladas.

Esto permite:

- evitar votos múltiples,
- garantizar igualdad (una persona, una identidad),
- y al mismo tiempo proteger completamente la identidad política del ciudadano.

## 3. Cómo se garantiza el anonimato (explicado sin tecnicismos)

Para comprender cómo se garantiza el anonimato, puede pensarse el sistema como el ingreso a un edificio público. En la puerta hay un guardia que pide el DNI: no para verificar que la persona existe —eso ya es evidente— sino para confirmar que se trata de un ciudadano habilitado a participar, y no de un turista de paso u otra persona sin derecho a intervenir. En términos digitales, este paso equivale a identificarse mediante sistemas oficiales del Estado (ANSES, AFIP/ARCA, MiArgentina u otros).

Una vez verificada esa condición, la persona atraviesa la puerta y el guardia se queda afuera. La identidad real no vuelve a circular dentro del edificio. Ya en el interior, al ciudadano se le entrega una identificación anónima —como una pulsera numerada— que **no** contiene datos personales y que es la única referencia utilizada por el sistema.

Esa identificación anónima permite registrar votaciones, acompañar propuestas y volver a ingresar en el tiempo para revisar o modificar decisiones, sin que sea posible conocer quién está detrás de cada acción. De este modo, el sistema garantiza que cada participación corresponde a un ciudadano real y único, pero sin que administradores, desarrolladores del sistema o autoridades puedan asociar acciones concretas con identidades reales.\
\
Esa credencial anónima:

- no contiene nombre, DNI ni datos personales,
- no puede ser revertida para identificar al ciudadano,
- es la única forma de interactuar con la plataforma.

Desde ese momento:

- el sistema ya “no sabe” quién es la persona,
- solo reconoce una identidad anónima válida.

Incluso quienes diseñaron, administran o poseen control total del sistema **no pueden reconstruir la identidad real** del participante.

## 4. Persistencia y cambio de opinión

A diferencia de votaciones anónimas tradicionales (que son únicas y estáticas), este sistema permite:

- que el ciudadano vea sus participaciones pasadas,
- que cambie su voto o apoyo con el tiempo,
- que acompañe la evolución de una propuesta.

Esto refleja mejor la realidad social: las opiniones no son fijas y pueden cambiar ante nueva información o contextos distintos.

## 5. Pérdida de credencial y prevención de abusos

La credencial anónima se basa en una frase o palabra clave conocida solo por el ciudadano.

Si esa credencial se pierde:

- la identidad anónima se considera irrecuperable,
- los votos asociados permanecen, pero ya no pueden modificarse,
- el ciudadano puede solicitar una nueva identidad, pero con una penalización temporal (por ejemplo, 6 meses).

Esta penalización existe para desalentar **abusos sistemáticos del mecanismo**, como intentar crear múltiples identidades para votar más de una vez.

## 6. Riesgos y críticas habituales (y cómo se mitigan)

### Uso malicioso, ruido político o propuestas provocativas

El diseño del sistema incorpora mecanismos preventivos que buscan desalentar el uso deliberado de la plataforma para generar ruido político, sin recurrir a censura de contenido ni moderación ideológica.

En primer lugar, cada ciudadano cuenta con un **cupo límitado de propuestas anuales** (por ejemplo, una o dos por año). Este límite introduce un costo simbólico a la acción de proponer, incentivando a:

- buscar si una propuesta similar ya existe antes de crear una nueva,
- reducir la duplicidad innecesaria de iniciativas,
- y desalentar el uso impulsivo o provocador del sistema (spam, trolls).

Como mecanismo de refuerzo, el sistema contempla que aquellas propuestas que alcancen niveles extraordinarios de apoyo ciudadano (votos de la propuesta) puedan "recompensar" al autor con la posibilidad de realizar una propuesta adicional fuera de su cupo anual (propuesta anual extra), premiando así la capacidad de representar el interés común.

En segundo lugar, el sistema incorpora un **filtro automático de lenguaje**, apoyado en tecnología actual, que actúa exclusivamente sobre la forma y no sobre el contenido de las ideas. Este filtro evita insultos, calificativos ofensivos o expresiones violentas, pero **no evalúa ni bloquea posturas ideológicas, críticas al gobierno ni propuestas incómodas para el poder**.

De este modo, se preserva la libertad de expresión política, al mismo tiempo que se mantiene un marco mínimo de lenguaje institucional que favorece la lectura, la participación y la convivencia democrática.

### “La gente puede mentir, manipular o coordinarse”

Como en cualquier sistema democrático, la coordinación de ideas es inevitable.

Lo que se evita es:

- la suplantación de identidad,
- el voto múltiple no autorizado,
- la presión directa sobre individuos identificables.

### “Esto reemplaza al Congreso o al sistema representativo”

No. La plataforma **no legisla ni decide**.

Funciona como:

- un canal de escucha estructurado,
- un termómetro social continuo,
- una herramienta de feedback directo.

Las decisiones siguen estando en manos de las instituciones democráticas existentes.

## 7. Valor estratégico para el Poder Ejecutivo y Legislativo

La plataforma permitiría:

- conocer con mayor precisión qué reformas generan apoyo real,
- detectar prioridades sociales emergentes,
- validar o corregir agendas políticas antes de avanzar.

En determinados contextos, puede funcionar como un **mecanismo de consulta ciudadana orientativa**, brindando una señal clara de tendencia y legitimidad social adicional a iniciativas clave, sin pretender representar a la totalidad del electorado.

## 8. Por qué este enfoque es distinto a experiencias previas

A diferencia de encuestas, foros o plataformas abiertas:

- la identidad está validada (no hay bots),
- el anonimato es real y persistente,
- la participación no es un evento aislado, sino continua.

Esto genera datos más confiables y una participación más honesta.

## 9. Cierre

Esta plataforma no promete soluciones mágicas.

Propone algo más simple y poderoso: **escuchar mejor**, sin miedo, sin exposición y sin intermediarios innecesarios.

Es una herramienta moderna para una democracia que necesita adaptarse a una sociedad más informada, más exigente y también más desconfiada.
