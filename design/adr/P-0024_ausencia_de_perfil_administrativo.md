# P-0024 — Ausencia de perfil administrativo

**Estado:** Activo

## Contexto

El sistema tiene un operador: la entidad institucional responsable de desplegar y administrar una instancia. El operador debe poder configurar el sistema y ejecutar las operaciones excepcionales que le corresponden. Entre ellas: la configuración del modo debug y la gestión de la retención de logs (P-0020); el retiro de propuestas y la configuración del catálogo de causales (P-0023); la configuración del cool-down y la gestión de claves del verificador externo en el emisor (P-0014, P-0015); el ajuste de parámetros operativos como los del ranking.

Hasta este ADR ningún documento de diseño formalizó *por qué vía* el operador ejerce esas capacidades. La cuestión estaba enunciada de forma dispersa —la entrada "operador" del glosario ya anticipaba que la configuración se hace por archivos y herramientas de línea de comando— pero nunca se registró como decisión con sus alternativas y trade-offs. P-0023, además, difirió explícitamente la "autenticación del operador" a la etapa de implementación, dejando un cabo suelto declarado.

La necesidad de cerrar esta decisión se hizo visible durante la formalización del rol del operador. La decisión cumple los criterios para un ADR: hay alternativas reales, alguien razonable podría proponer lo contrario más adelante, afecta la arquitectura de forma transversal y tiene trade-offs claros. No puede quedar como un párrafo descriptivo dentro de un documento de diseño.

Las preguntas de diseño que motivaron este ADR son:

- ¿El operador ejerce su rol a través de un perfil administrativo con login privilegiado y una interfaz de administración propia, o por otra vía?
- ¿Cómo se ejercen la configuración del sistema y las operaciones excepcionales del operador?
- ¿La respuesta es uniforme para la capa de identidad y la aplicación destino, o hay matices por componente?

## Opciones consideradas

### Opción A — Perfil administrativo con login privilegiado e interfaz de administración

El sistema incorpora un usuario o perfil administrativo. El operador se autentica con credenciales privilegiadas y configura el sistema y ejecuta las operaciones excepcionales a través de una interfaz de administración propia de la aplicación.

Ventajas

- Ergonomía operativa: la configuración y las operaciones excepcionales se ejercen desde una interfaz, sin requerir acceso al servidor ni conocimiento técnico profundo.
- Comportamiento familiar para quien administra sistemas web.
- Permite registrar las acciones administrativas atadas a la identidad del perfil que las ejecutó, dando trazabilidad a nivel de aplicación cuando hay varios operadores.

Desventajas

- Introduce una superficie de ataque autenticada y expuesta en red. Un atacante que robe, adivine o phishee la credencial del perfil obtiene la capacidad de configurar el sistema, alterar datos y modificar su comportamiento, sin necesidad de comprometer la infraestructura.
- Concentra detrás de una credencial un conjunto amplio de capacidades sensibles, contrario al principio de minimización de puntos únicos que se deriva del modelo de amenazas de P-0006.
- La credencial es un objeto que circula, se reutiliza y se filtra; el control de acceso al sistema operativo y a la base de datos, en cambio, ya existe y el operador debe protegerlo de todos modos.

### Opción B — Sin perfil administrativo: configuración por archivos y herramientas de línea de comando

El sistema no tiene un perfil ni usuario administrativo. No existe login privilegiado ni interfaz de administración. Toda la configuración y toda operación excepcional del operador se realiza mediante archivos de configuración o herramientas de línea de comando que actúan sobre la base de datos o el entorno.

Ventajas

- No agrega ninguna superficie de ataque autenticada y expuesta en red. El control de acceso a las capacidades del operador es el control de acceso al sistema operativo y a la base de datos, que el despliegue ya debe proteger.
- Para ejercer las capacidades del operador, un atacante necesita una shell del sistema o acceso de lectura-escritura sobre las tablas críticas, no una credencial de aplicación. El conjunto de cosas que se pueden robar, adivinar o phishear no se amplía.
- Coherente con el modelo de amenazas de P-0006 y con el principio de integridad verificable: el sistema no agrega una vía de acceso privilegiada cuya seguridad dependa de la protección de una credencial.

Desventajas

- Menor comodidad operativa. Operar por archivos de configuración y herramientas de línea de comando es más incómodo y con más fricción que una interfaz; requiere acceso al servidor y conocimiento técnico.
- No hay un registro de acciones administrativas a nivel de aplicación atado a identidades de perfiles; la trazabilidad de las acciones del operador queda en la capa del sistema operativo y de la base de datos.

### Opción C — Esquema híbrido: interfaz de observabilidad de solo lectura

El sistema no admite escritura por interfaz —toda configuración y operación excepcional va por archivos y herramientas de línea de comando, como en la Opción B— pero incorpora una interfaz de administración de *solo lectura* para observabilidad: estado del sistema, métricas, registros.

Ventajas

- Da ergonomía de consulta sin habilitar la modificación del sistema por interfaz.
- Reduce el botín de una credencial comprometida respecto de la Opción A: no permite escribir.

Desventajas

- Sigue introduciendo una superficie de ataque autenticada y expuesta en red. En el modelo de amenazas de P-0006 el acceso de *lectura* a datos operativos —logs, métricas, marcas de tiempo— es de por sí sensible: es la materia prima de la correlación. "Solo lectura" no lo vuelve inocuo.
- La observabilidad ya está cubierta por herramientas de la capa de infraestructura, a las que el operador accede con el mismo acceso que necesita para la Opción B. Una interfaz de observabilidad dentro de la aplicación es mayormente redundante y agrega un vector de autenticación.
- Una interfaz de solo lectura tiende a erosionarse hacia la escritura con el tiempo, por presión de conveniencia.

## Decisión

Se adopta la **Opción B**.

El sistema no tiene un perfil ni usuario administrativo. No existe login privilegiado ni interfaz de administración. Toda la configuración del sistema y toda operación excepcional que corresponde al operador se ejerce mediante archivos de configuración o herramientas de línea de comando que actúan sobre la base de datos o el entorno.

La decisión aplica de forma uniforme a la capa de identidad y a la aplicación destino. Por eso este ADR es transversal.

El alcance de la decisión es el perfil de administración del sistema. Este ADR resuelve, a nivel de diseño, la "autenticación del operador" que P-0023 dejó explícitamente diferida a implementación: no hay autenticación de un perfil porque no hay perfil; el control de acceso a las capacidades del operador es el control de acceso al sistema operativo y a la base de datos.

## Justificación

La decisión se sostiene sobre dos ejes.

El primero es la reducción de la superficie de ataque. Un perfil administrativo concentra, detrás de una credencial, la capacidad de configurar el sistema, alterar datos y modificar su comportamiento. Una credencial es un objeto que circula y puede robarse, adivinarse o phishearse, y un perfil con interfaz es una superficie autenticada y expuesta en red. Sin perfil administrativo, un atacante necesita una shell del sistema o acceso de lectura-escritura sobre las tablas críticas para ejercer esas capacidades. El control de acceso al sistema operativo y a la base de datos no es una vía nueva: es un control que el despliegue ya tiene que proteger de todos modos. La decisión no agrega una vía de acceso adicional; reutiliza la que ya existe.

El segundo es la coherencia con el modelo de confianza del sistema. P-0006 adopta un modelo de amenazas intermedio que no asume un operador benévolo y que asume que componentes individuales del sistema pueden ser comprometidos. El principio de integridad verificable establece que la arquitectura no debe depender de la confianza ciega en operadores o administradores. Un perfil administrativo es exactamente el tipo de punto único —una credencial que concentra capacidades sensibles— que ese modelo pide minimizar. La decisión no se funda en desconfiar del operador en abstracto: el operador tiene acceso al sistema operativo y a la base de datos de todos modos, y ese acceso es inherente a su rol. Se funda en no crear una vía de acceso *adicional* que amplíe lo que un atacante gana sin necesidad de comprometer la infraestructura.

La decisión no es trivialmente unívoca. Es coherente con el modelo de amenazas que el sistema ya adoptó en P-0006. Un sistema con un modelo de amenazas distinto, o un contexto de despliegue de baja amenaza con un operador poco técnico, podría razonablemente haber preferido la Opción A y aceptado su superficie de ataque a cambio de ergonomía. Este ADR elige la Opción B como consecuencia deliberada de los principios que el sistema ya tiene, no como la opción universalmente obvia.

## Consecuencias

- Las operaciones excepcionales del operador definidas en P-0023 —el retiro de propuestas y la configuración del catálogo de causales— se ejecutan por la vía decidida en este ADR: archivos de configuración o herramientas de línea de comando sobre la base de datos. No existe una interfaz de administración para ejecutarlas.

- Las capacidades del operador sobre logs y modo debug definidas en P-0020 se ejercen por esta misma vía. P-0020 ya establece el modo debug como un flag de configuración explícito y dedicado; esa caracterización es consistente con este ADR.

- Este ADR cierra el cabo suelto que P-0023 dejó declarado al diferir la "autenticación del operador" a implementación. La respuesta de diseño es que no hay un perfil que autenticar. Este ADR no contradice ni supersede a P-0023: P-0023 asumió que existía una pregunta de implementación, y este ADR provee la respuesta de diseño que la acota. Lo que queda para la etapa de implementación es genuinamente implementación: qué herramienta concreta, qué formato de archivo, qué comandos.

- El trade-off aceptado es una menor comodidad operativa. Operar por archivos de configuración y herramientas de línea de comando tiene más fricción que una interfaz, requiere acceso al servidor y exige conocimiento técnico. Es un costo real, no menor. Adicionalmente, no hay un registro de acciones administrativas a nivel de aplicación atado a identidades de perfiles; la trazabilidad de las acciones del operador queda en la capa del sistema operativo y de la base de datos. Esto es coherente con P-0023, que ya establece que la aplicación no almacena información sobre los actores que ejecutaron un retiro: la trazabilidad de las acciones del operador es responsabilidad de la infraestructura del despliegue.

- La decisión aplica de forma uniforme a la capa de identidad y a la aplicación destino. El conjunto concreto de operaciones difiere por componente —en el emisor, por ejemplo, la configuración del cool-down y la gestión de claves del verificador externo; en la aplicación de participación ciudadana, el retiro de propuestas, el catálogo de causales y los parámetros de ranking—, pero el mecanismo es el mismo en los dos casos. Esa diferencia es de inventario de operaciones, no de mecanismo, y no rompe la uniformidad de la decisión.

- El alcance de este ADR es el perfil de administración del sistema: la capacidad de configurar el sistema, alterar la base de datos, acceder a logs y metadatos operativos, y operar sobre la integridad de la capa de identidad. Este ADR niega que exista un perfil para esas capacidades en cualquier componente. No se pronuncia, en cambio, sobre los roles funcionales que el diseño de una aplicación destino pueda definir para actores no-ciudadanos como parte de su funcionalidad. Un rol funcional opera sobre los datos de dominio de su aplicación a través de operaciones que el diseño de esa aplicación define; no configura el sistema, no accede a datos operativos ni a infraestructura, y no opera sobre la integridad de la identidad. El criterio que separa ambos casos no es si la capacidad es privilegiada o si toca contenido, sino qué gana un atacante que comprometa esa credencial: administración del sistema, o una operación acotada al dominio funcional de una aplicación. Los roles funcionales, si una aplicación destino los definiera, son materia de los ADRs de esa aplicación, bajo su propio modelo de amenazas, y en ningún caso reciben capacidades de administración del sistema. A modo de ilustración: la aplicación de participación ciudadana se construye sobre la ausencia de moderación de contenido (P-0023) y no define ningún rol de ese tipo; una aplicación destino futura con otro principio funcional —por ejemplo, un sistema de denuncias que incorpore moderación— podría definir un rol de moderación, y esa sería una decisión de los ADRs de esa aplicación, no de este ADR.

- La guía de instalación y operación debe documentar la vía operativa concreta por la que el operador ejerce sus capacidades, así como los cuidados asociados al acceso al sistema operativo y a la base de datos.

- El detalle concreto de implementación —qué herramienta de línea de comando, qué formato de archivo de configuración, qué comandos— queda fuera del alcance de este ADR y corresponde a la etapa de implementación.

## Referencias

- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0020 — Política de logs y retención de metadatos
- P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino
- P-0023 — Moderación de contenido y retiro de propuestas
- `design/threat_model.md` — Modelo de amenazas
- `notas/propuesta_guia_de_instalacion.md` — Material en gestación para la guía de instalación y operación
