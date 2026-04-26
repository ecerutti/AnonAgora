# P-0020 — Política de logs y retención de metadatos

**Estado:** Activo

## Contexto

P-0006 establece que el sistema debe adoptar retención limitada de metadatos operativos y minimización de datos almacenados en cada componente, pero no fija qué eventos se registran, con qué granularidad temporal, ni por cuánto tiempo se retienen.

Esta omisión es intencional: P-0006 define el principio, este ADR lo concreta.

Los principios de esta política son transversales al sistema. Las listas concretas de eventos y campos hacen referencia a los dos componentes que generan logs hoy: el **emisor**, parte de la capa de identidad; y la **aplicación destino**, que en el despliegue actual es la aplicación de participación ciudadana (P-0021). En las tablas de este ADR se usan los nombres operativos "Emisor" y "Plataforma" para referirse a esos dos componentes.

Los logs operativos son necesarios para diagnosticar errores, detectar comportamiento anómalo y auditar el funcionamiento del sistema. Al mismo tiempo, representan una superficie de correlación real: un atacante con acceso a logs de múltiples componentes podría inferir la relación entre identidad real y actividad del ciudadano, particularmente mediante correlación temporal entre eventos del emisor y eventos de la aplicación destino.

El emisor y la aplicación destino pueden correr en infraestructuras completamente separadas y bajo administradores distintos, lo cual es coherente con la separación de funciones de P-0006. La política de logs debe ser compatible con ese escenario.

Las preguntas de diseño que motivaron este ADR son:

- ¿Qué eventos se registran en cada componente?
- ¿Con qué granularidad temporal?
- ¿Qué campos están prohibidos en logs?
- ¿Cómo se gestiona el modo debug?
- ¿Por cuánto tiempo se retienen los logs y quién es responsable de eliminarlos?

## Opciones consideradas

### Decisión 1 — Granularidad temporal en modo operativo

El riesgo principal de los logs en este sistema es la correlación temporal cruzada entre componentes. Un timestamp preciso en el log del emisor y otro en el log de la aplicación destino, tomados en el mismo momento, permiten inferir que ambos eventos corresponden al mismo ciudadano, colapsando la separación de funciones establecida en P-0006.

La pregunta es con qué granularidad registrar los timestamps en modo operativo normal.

#### Opción A — Timestamp completo con precisión de segundos

Los logs registran timestamps con precisión de segundos en todos los componentes y en todos los modos.

Ventajas

- Comportamiento estándar, familiar para administradores.
- Máxima utilidad para diagnóstico de incidentes técnicos.

Desventajas

- Permite correlación temporal cruzada entre logs del emisor y la plataforma con alta precisión.
- Contradice el principio de minimización de metadatos de P-0006.

#### Opción B — Granularidad mínima por tipo de evento en modo operativo, segundos en modo debug

En modo operativo, cada tipo de evento se registra con la granularidad temporal mínima suficiente para el diagnóstico que justifica su existencia. En modo debug, los timestamps incluyen precisión de segundos para facilitar el diagnóstico detallado.

Ventajas

- Reduce significativamente la superficie de correlación temporal cruzada entre componentes.
- Mantiene utilidad diagnóstica para los casos que la requieren.
- El modo debug preserva la capacidad de diagnóstico detallado cuando es necesario, bajo condiciones controladas.

Desventajas

- Requiere que la implementación aplique el recorte de timestamp según el tipo de evento, lo cual introduce complejidad respecto a un timestamp uniforme.

### Decisión 2 — Campos prohibidos en logs

Ciertos datos nunca deben aparecer en logs, independientemente del modo de operación, porque su presencia permitiría correlación directa entre identidad real y actividad dentro de la plataforma, o comprometería la seguridad de las credenciales del ciudadano.

La pregunta es definir esa lista con precisión, teniendo en cuenta que algunos identificadores son legítimos en un componente pero prohibidos en otro.

#### Opción A — Lista uniforme prohibida en todos los componentes

Un único conjunto de campos prohibidos aplica a todos los componentes sin distinción.

Ventajas

- Regla simple de implementar y auditar.

Desventajas

- Imprecisa: el `anon_seed` es un dato interno legítimo del emisor pero catastrófico si aparece en logs de la plataforma. El `anon_id` es un identificador operativo legítimo de la plataforma pero no debería aparecer en logs del emisor porque revelaría la asociación entre ambos identificadores. Una lista uniforme no captura esta distinción.

#### Opción B — Lista diferenciada por componente

Los campos prohibidos se definen por componente, reconociendo que el riesgo de cada dato depende del contexto en que aparece.

Ventajas

- Precisa: refleja el modelo de separación de funciones de P-0006.
- Evita tanto la permisividad excesiva como la restricción innecesaria.

Desventajas

- Requiere que cada componente conozca y aplique su propia lista, lo cual exige documentación clara y verificación en implementación.

### Decisión 3 — Modo debug

El diagnóstico de ciertos problemas requiere información que no es seguro registrar en operación normal. La pregunta es si el sistema ofrece un modo de logging más detallado y bajo qué condiciones.

#### Opción A — Sin modo debug; logs siempre uniformes

El sistema no tiene modo debug. Los logs son siempre los mismos. El diagnóstico de problemas complejos se realiza exclusivamente por otros medios: métricas agregadas, reproducción en entorno de prueba.

Ventajas

- Elimina el riesgo de que el modo debug quede activado inadvertidamente en producción.
- Implementación más simple.

Desventajas

- En la práctica, cuando algo falla en producción y los logs operativos no alcanzan para diagnosticarlo, la alternativa es peor: el administrador interviene directamente en el código o la infraestructura para obtener más información, sin ningún control sobre qué datos expone.

#### Opción B — Modo debug activable explícitamente, con visibilidad obligatoria

El sistema ofrece un modo debug activable por el operador. Cuando está activo, los timestamps incluyen precisión de segundos y se habilitan campos adicionales útiles para diagnóstico que no se registran en modo operativo.

El modo debug debe ser visible para todos los usuarios del sistema. La especificación de cómo se comunica este modo corresponde a la capa de UX de la plataforma.

Ventajas

- Preserva la capacidad de diagnóstico detallado cuando es necesario.
- La visibilidad obligatoria protege al ciudadano casual que accede al sistema mientras el modo está activo.
- Es preferible a la alternativa real: intervención ad-hoc sin controles.

Desventajas

- Introduce complejidad de implementación.
- El riesgo de que el modo quede activado inadvertidamente existe, aunque la visibilidad obligatoria lo mitiga.

### Decisión 4 — Responsabilidad y mecanismo de retención de logs

Los logs deben eliminarse transcurrido el período de retención definido. La pregunta es quién es responsable de esa eliminación y mediante qué mecanismo.

#### Opción A — Gestión interna por cada componente

Cada componente implementa su propio mecanismo de eliminación de logs vencidos, sin depender de herramientas o configuraciones externas del entorno de despliegue.

Ventajas

- El comportamiento es predecible e independiente del entorno de despliegue y del sistema operativo utilizado.
- Una actualización de la plataforma que introduce un nuevo archivo de log queda automáticamente cubierta por el mecanismo interno, sin requerir cambios en la configuración del entorno.
- Coherente con la separación de funciones: cada componente es responsable de sus propios datos, incluso para su eliminación.

Desventajas

- Agrega complejidad a la implementación de cada componente.
- No elimina el riesgo de un operador malintencionado, que puede copiar logs antes de su eliminación. Este riesgo es una limitación conocida del modelo de amenazas intermedio de P-0006 y no es resoluble en este nivel.

#### Opción B — Delegación en mecanismos externos del entorno de despliegue

La plataforma loguea y delega la eliminación a herramientas del entorno de despliegue. La guía de instalación y operación especifica cómo configurarlas para cumplir la política definida en este ADR.

Ventajas

- Sin complejidad adicional en la aplicación.

Desventajas

- Si el entorno no está configurado correctamente, la plataforma no tiene forma de garantizar la retención.
- Una actualización de la plataforma que introduce un nuevo archivo de log queda desprotegida hasta que el operador actualice manualmente la configuración del entorno.
- El cumplimiento de la política queda en manos del despliegue, no del software.
- Asume características específicas del entorno de despliegue que no pueden garantizarse en todos los contextos donde la plataforma podría operar.

## Decisión

Se adoptan las opciones B, B, B y A en las decisiones 1, 2, 3 y 4 respectivamente.

### Granularidad temporal por tipo de evento en modo operativo

| Componente | Evento | Granularidad |
|---|---|---|
| Emisor | Error de integración con AUTENTICAR | Minuto |
| Emisor | Solicitud de emisión rechazada | Hora |
| Emisor | Emisión de identidad anónima exitosa | Hora |
| Plataforma | Error interno del sistema | Minuto |
| Plataforma | Fallo de autenticación (login) | Hora |
| Plataforma | Sesión iniciada o destruida | Hora |
| Plataforma | Publicación de propuesta | Día |
| Plataforma | Rechazo del revisor de lenguaje | Hora |
| Plataforma | Error de integración con revisor de lenguaje | Minuto |

En modo debug todos los eventos se registran con precisión de segundos.

### Campos prohibidos por componente

Los siguientes campos nunca deben aparecer en logs, en ningún modo de operación.

**Prohibidos en todos los componentes:**

- CUIL / CUIT del ciudadano
- Frase secreta del ciudadano o cualquier derivado de ella
- Token de AUTENTICAR completo
- Dirección IP

La dirección IP se excluye porque su presencia en los logs de la aplicación no agrega valor diagnóstico que no pueda obtenerse de otras fuentes, y sí aumenta la superficie de correlación entre identidad real y actividad. Su gestión corresponde a controles de infraestructura documentados en la guía de instalación y operación.

**Prohibidos específicamente en el emisor:**

- `anon_id`
- Nonce de derivación

**Prohibidos específicamente en la plataforma:**

- `anon_seed`

### Campos habilitados solo en modo debug

Los siguientes campos no se registran en modo operativo y solo se habilitan en modo debug.

**Emisor:**

- Motivo específico de rechazo de emisión (cool-down, unicidad, error técnico)
- Claim específico que causó rechazo del JWT de AUTENTICAR
- Parámetros técnicos de la prueba ZK generada

**Plataforma:**

- Motivo específico de fallo de login (pseudónimo inexistente o frase incorrecta). En modo operativo este detalle se omite para evitar que los logs faciliten enumeración de pseudónimos.
- Duración de la verificación Argon2id, útil para tuning de parámetros de costo.
- Identificador de propuesta involucrada en un error.
- Motivo específico de rechazo del revisor de lenguaje.
- Componentes intermedios del score de ranking.
- Duración individual de requests. En modo operativo el timing preciso de requests individuales se omite porque es correlacionable; las métricas de rendimiento se gestionan como agregados.

### Modo debug

El modo debug se activa explícitamente por el operador mediante un flag de configuración dedicado, independiente del nivel de log del framework de logging utilizado, para evitar activaciones accidentales. Cuando está activo, el sistema debe comunicarlo de forma visible a todos los usuarios. La especificación de cómo se comunica y qué comportamiento adicional se requiere de las interfaces corresponde a la capa de UX de la plataforma.

### Retención y eliminación de logs

Los plazos de retención son configurables por el operador, con los siguientes valores por defecto.

**Logs operativos:** 7 días. Cada componente elimina automáticamente los logs que superan el período de retención configurado.

**Logs de modo debug:** se eliminan automáticamente al desactivar el modo debug. Si el modo debug no se desactiva, los logs se eliminan en un plazo máximo de 1 día desde su generación.

Cada componente gestiona la eliminación de sus propios logs de forma independiente. Esta separación es consecuencia directa del modelo de separación de funciones de P-0006: el emisor y la aplicación destino pueden correr en infraestructuras completamente distintas y bajo administradores diferentes.

### Limitaciones conocidas

La política de retención define la intención del sistema y su implementación técnica, pero no puede garantizar cumplimiento en todos los escenarios posibles. Las siguientes situaciones son limitaciones conocidas y aceptadas dentro del modelo de amenazas intermedio de P-0006:

- **Backups que capturan el sistema completo.** Los backups realizados a nivel de máquina virtual, snapshot o imagen completa del sistema pueden incluir logs vigentes en el momento del backup. Estos logs quedan retenidos durante el período de retención del backup, que suele ser significativamente mayor al período de retención de logs definido en este ADR. La guía de instalación y operación debe documentar esta limitación y recomendar mecanismos para mitigarla cuando el sistema de backup lo permita.
- **Copia manual de logs.** Un operador con acceso al sistema puede copiar archivos de logs antes de su eliminación automática. Este riesgo no es mitigable en este nivel y es una consecuencia aceptada del modelo de amenazas intermedio.
- **Logs de infraestructura.** Los logs del proxy inverso, load balancer, firewall y proveedor de infraestructura están fuera del alcance de este ADR y pueden retener información sensible como direcciones IP y marcas de tiempo precisas. Su gestión corresponde a la guía de instalación y operación.

## Justificación

La granularidad mínima por tipo de evento reduce la superficie de correlación temporal cruzada entre componentes sin sacrificar la utilidad diagnóstica real. Los incidentes técnicos se diagnostican por secuencia de eventos, no por precisión de segundos. La detección de ataques de fuerza bruta se resuelve mejor mediante rate limiting en capa de aplicación que mediante análisis retrospectivo de logs con timestamps precisos.

La lista diferenciada de campos prohibidos por componente refleja el modelo de separación de funciones de P-0006: el riesgo de cada dato depende del contexto en que aparece. Prohibir el `anon_id` en el emisor y el `anon_seed` en la aplicación destino preserva la separación entre las dos capas de identificadores incluso ante un atacante con acceso a los logs de un único componente.

El modo debug con visibilidad obligatoria es preferible a la alternativa real: cuando los logs operativos no alcanzan para diagnosticar un problema en producción, la alternativa es la intervención ad-hoc del administrador sin ningún control sobre qué datos expone. Un modo debug explícito y visible es más seguro que esa alternativa.

La gestión interna de retención por cada componente es preferible a depender de mecanismos externos del entorno de despliegue porque garantiza que el comportamiento sea predecible independientemente del sistema operativo o las herramientas disponibles. Una actualización de la plataforma que introduce un nuevo archivo de log queda automáticamente cubierta sin intervención del operador.

## Consecuencias

- La implementación de cada componente debe aplicar el recorte de timestamp según el tipo de evento definido en este ADR.
- La implementación debe verificar mediante tests que los campos prohibidos no aparezcan en los paths de logging de cada componente.
- El modo debug debe implementarse como un flag de configuración explícito y dedicado.
- Cada componente debe implementar un mecanismo interno de eliminación de logs vencidos, independiente de herramientas externas del entorno de despliegue.
- La guía de instalación y operación debe documentar las limitaciones conocidas de la política de retención, en particular el riesgo de backups que capturan el sistema completo y los logs de infraestructura fuera del alcance de la plataforma.
- La especificación de cómo el sistema comunica el modo debug a los usuarios corresponde al diseño de interfaz de la plataforma.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- `design/threat_model.md` — Modelo de amenazas
- `notas/propuesta_guia_de_instalacion.md` — Material en gestación para la guía de instalación y operación
