# P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino

**Estado:** Activo

## Contexto

Hasta este ADR, el sistema se diseñó asumiendo una única aplicación destino: la plataforma de participación ciudadana. La documentación, los ADRs existentes y la estructura del repositorio reflejan esa asunción. "La plataforma" se usa para referirse simultáneamente al sistema completo y a la aplicación participativa en particular, sin distinción.

Durante el diseño quedó progresivamente claro que varias decisiones técnicas no son específicas de la participación ciudadana. El emisor de identidad anónima (P-0015), su integración con AUTENTICAR (P-0013), la auditoría criptográfica mediante ZK (P-0014), el modelo de amenazas (P-0006) y otras decisiones relacionadas con la identidad anónima operan sobre el ciudadano, sin depender de qué haga la aplicación destino con la identidad anónima emitida.

El origen concreto de esta observación fue una conversación informal durante la difusión de la propuesta: una lectora planteó si el mismo mecanismo podría servir para canalizar denuncias anónimas. Al analizar esa pregunta quedó evidente que la respuesta era afirmativa —el sustrato de identidad es idéntico— y que el diseño ya lo permitía técnicamente, pero no estaba reconocido como tal en la documentación.

La pregunta de diseño que motivó este ADR es:

¿El sistema debe modelarse como una única aplicación con su infraestructura embebida, o como una infraestructura reutilizable sobre la cual puede apoyarse una o más aplicaciones destino?

Esta decisión es estructural y afecta cómo se organizan las decisiones futuras, cómo se nombran los componentes del sistema y cómo se presenta el proyecto.

## Opciones consideradas

### Opción A — Monolito: aplicación única

El sistema se define como una plataforma de participación ciudadana. La verificación de identidad, la emisión de identidad anónima y la participación son partes internas de ese único producto. Si en el futuro se quisiera construir una segunda aplicación del mismo tipo —por ejemplo, una plataforma de denuncias anónimas—, sería un proyecto separado que tendría que reimplementar la infraestructura de identidad.

Ventajas

- Corresponde al estado del repositorio antes de este ADR: no requiere refactorización.
- Modelo mental más simple: un único producto, un único conjunto de decisiones.
- Evita la necesidad de definir un contrato explícito entre capas.

Desventajas

- La infraestructura de identidad ya es técnicamente reutilizable pero la documentación no lo reconoce. Esa brecha entre diseño real y diseño documentado es una fuente de ambigüedad.
- Construir una segunda aplicación del mismo tipo obligaría a reimplementar trabajo ya hecho, o a un esfuerzo de extracción tardío más costoso que reconocer la modularidad ahora.
- Vocabulario acoplado: "la plataforma" significa a veces el sistema completo, a veces la aplicación participativa. Esto se nota especialmente en los ADRs transversales (P-0006, P-0020).

### Opción B — Capa de identidad + aplicación destino, una por despliegue

El sistema se reconoce como la composición de dos capas: una **infraestructura de identidad anónima verificada** (la "capa de identidad") reutilizable, y una **aplicación destino** construida sobre ella. Cada despliegue consiste en una capa de identidad asociada a exactamente una aplicación destino; ambas se instalan y operan juntas.

La aplicación de participación ciudadana es, en este modelo, una aplicación destino entre otras posibles. En el futuro podrían existir otras —por ejemplo, una aplicación de denuncias anónimas— como despliegues independientes, cada una con su propia capa de identidad. Un ciudadano que quiera participar en dos aplicaciones distintas debe registrarse por separado en cada una, obteniendo identidades anónimas independientes.

Ventajas

- Reconoce explícitamente una modularidad que ya existe en el diseño técnico, eliminando la brecha entre diseño real y diseño documentado.
- Deja la puerta abierta a futuras aplicaciones destino sin rediseño técnico. El trabajo es de refactorización documental y estructural, no de rearquitectura.
- Permite que un ciudadano elija en qué aplicaciones participar. Su decisión de registrarse en una aplicación no lo inscribe automáticamente en otras construidas sobre la misma tecnología.
- Identidades independientes por aplicación refuerzan la separación entre contextos: la actividad en una aplicación no es vinculable con la actividad en otra ni siquiera por el mismo ciudadano.
- Preserva la simplicidad operativa: cada despliegue tiene operadores con responsabilidad clara sobre capa de identidad y aplicación destino.

Desventajas

- Requiere refactorización de vocabulario, estructura de carpetas y documentación descriptiva del repositorio.
- Introduce una distinción conceptual nueva (capa vs aplicación) que los lectores del proyecto deben internalizar.
- Si en el futuro se desea compartir una capa de identidad entre varias aplicaciones en un mismo despliegue, este modelo no lo contempla; requeriría un ADR posterior.

### Opción C — Capa de identidad + múltiples aplicaciones sobre la misma capa

Variante más ambiciosa de B: en un mismo despliegue, una única capa de identidad sostiene simultáneamente varias aplicaciones destino. El ciudadano se registra una sola vez y su identidad anónima es válida para todas las aplicaciones de ese despliegue.

Ventajas

- Registro único por despliegue, menor fricción si el ciudadano quiere participar en varias aplicaciones.
- Costos operativos compartidos entre aplicaciones.

Desventajas

- Fuerza al ciudadano a aceptar el conjunto completo de aplicaciones para poder participar en cualquiera de ellas. Un ciudadano interesado en la participación ciudadana pero no en denuncias anónimas no puede elegir; al registrarse queda habilitado para ambas.
- La misma identidad anónima operando en múltiples contextos introduce superficie de correlación cruzada entre aplicaciones. Dos aplicaciones con acceso a la misma identidad pueden, por sí solas o por colusión, reconstruir historiales combinados que en el modelo B son imposibles.
- Introduce preguntas de diseño nuevas cuya evaluación no es prioridad actual: cómo se comparten las sesiones entre aplicaciones, cómo se maneja la frase secreta (¿única? ¿distinta por aplicación?), cómo se aplican límites cruzados como el límite anual de propuestas, cómo se audita una aplicación sin visibilidad sobre las otras.
- Complejidad significativamente mayor sin beneficio claro para el alcance actual del proyecto.

## Decisión

Se adopta la **Opción B**.

El sistema se estructura como la composición de una capa de identidad y una aplicación destino:

- **Capa de identidad** (forma extendida: *infraestructura de identidad anónima verificada*): responsable de la integración con el verificador externo, la emisión de identidades anónimas y la provisión de credenciales que el ciudadano usa para autenticarse en la aplicación destino. La capa encapsula la integración con el verificador externo (AUTENTICAR en el contexto argentino), pero el verificador en sí no forma parte del sistema construido.

- **Aplicación destino**: la aplicación concreta que el ciudadano usa y que consume identidades anónimas emitidas por la capa. La aplicación de participación ciudadana es la única aplicación destino actualmente diseñada; el modelo admite otras (por ejemplo, una aplicación de denuncias anónimas) como despliegues independientes.

Cada despliegue está compuesto por exactamente una capa de identidad y una aplicación destino. La variante de múltiples aplicaciones simultáneas sobre la misma capa (Opción C) queda fuera del alcance actual y, si se considera en el futuro, requerirá un ADR propio que aborde las preguntas de diseño que introduce.

## Justificación

La modularidad reconocida por este ADR no es una arquitectura nueva sino el reconocimiento explícito de una modularidad que ya existía en el diseño técnico. El emisor nunca fue parte de la aplicación participativa: opera con anterioridad a cualquier acción dentro de ella y entrega sus resultados sin saber qué se hace con ellos (P-0015). El modelo de amenazas ya separa tres funciones —verificación, emisión y aplicación— como requisito arquitectónico (P-0006). Las decisiones sobre pseudónimos amigables y logs tienen sentido con independencia de qué haga la aplicación destino.

Lo que este ADR cambia no es el diseño técnico sino el marco conceptual y el vocabulario con el que se presenta ese diseño. El costo es acotado —refactorización documental y estructural del repositorio— y el beneficio es que la puerta a futuras aplicaciones destino queda abierta sin necesidad de rediseño técnico posterior.

Se elige B sobre C porque respeta la decisión del ciudadano sobre en qué aplicaciones participar. Una persona que se registra en la aplicación de participación ciudadana puede tener razones legítimas para no querer estar presente en una aplicación de denuncias anónimas, y viceversa. El modelo de una capa por aplicación preserva esa elección como propiedad estructural del sistema. Además, mantener identidades anónimas independientes por aplicación refuerza el principio de minimización de correlaciones de P-0006: la actividad en una aplicación no es vinculable con la actividad en otra ni siquiera cuando corresponden a la misma persona real.

Se elige B sobre A porque mantener la descripción del sistema como aplicación única cuando técnicamente no lo es genera ambigüedad creciente a medida que el proyecto evoluciona. Los ADRs posteriores quedarían redactados en un vocabulario atado a la aplicación participativa, y cada aplicación destino futura forzaría una refactorización más extensa que la que se hace ahora.

Esta decisión se toma tardíamente respecto del inicio del proyecto. El disparador concreto fue una conversación informal en la que una lectora de la propuesta planteó si el mismo mecanismo podría usarse para canalizar denuncias anónimas. La pregunta hizo visible una propiedad del diseño que estaba presente pero no reconocida. Documentar esa historia es coherente con el principio de integridad verificable del sistema: los lectores futuros deben poder distinguir qué partes del diseño fueron intencionales desde el principio y cuáles se consolidaron después.

## Consecuencias

- La estructura del repositorio se reorganiza para reflejar este modelo. En `design/` aparecen subcarpetas para la capa de identidad y para cada aplicación destino (inicialmente, una sola: la de participación ciudadana). La misma reorganización aplica en `implementation/` y en las secciones de demo correspondientes. Los ADRs transversales —aquellos que deciden sobre la relación entre capa y aplicación o sobre el sistema como tal— quedan en la raíz de cada carpeta de ADRs. La estructura concreta se documenta en `AGENTS.md` tras este ADR.
- Los identificadores de ADRs existentes (`P-XXXX`, `I-XXXX`, `DP-XXXX`, `DI-XXXX`) se conservan. La capa a la que pertenece cada ADR queda identificada por la carpeta donde vive, no por el prefijo. Las referencias cruzadas existentes no se modifican.
- La aplicación de participación ciudadana queda reconocida como una aplicación destino entre otras posibles, no como la definición del sistema. Los documentos descriptivos (`AGENTS.md`, `docs/architecture_overview.md`, `design/README.md`, modelos en `design/`) se actualizan para reflejar esto.
- La documentación conceptual en `docs/propuesta/` mantiene su estructura pero introduce un marco inicial que deja claro que describe una aplicación destino específica, construida sobre una capa de identidad reutilizable.
- Un ciudadano que quiera participar en dos aplicaciones destino distintas debe registrarse por separado en cada una. Sus identidades anónimas en cada aplicación son independientes y no vinculables entre sí.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
