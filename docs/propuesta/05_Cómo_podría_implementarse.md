# 05 — Cómo podría implementarse  
### Por qué la idea es técnicamente viable con herramientas actuales

## Introducción

La propuesta presentada en este documento no depende de tecnologías futuristas ni de avances científicos aún inexistentes.

En la actualidad ya existen numerosas herramientas tecnológicas utilizadas en ámbitos como la identidad digital, la criptografía aplicada, las plataformas de participación ciudadana y el análisis automático de lenguaje. Estas tecnologías permiten construir sistemas seguros, escalables y auditables capaces de operar a gran escala.

La innovación principal de esta propuesta no reside en una tecnología individual, sino en la **arquitectura que combina estas herramientas para habilitar un sistema de participación ciudadana con identidad anónima persistente**.

Este documento describe, a nivel conceptual, cómo las distintas partes del sistema podrían implementarse utilizando tecnologías disponibles actualmente.

## Identidad digital y verificación de ciudadanos

Uno de los requisitos fundamentales del sistema es garantizar que cada participante sea una persona real y única. Esto permite evitar manipulaciones mediante cuentas múltiples y mantener la integridad del proceso participativo.

Muchos países ya cuentan con infraestructura de identidad digital utilizada en distintos servicios públicos. En el caso de Argentina, existen diversos sistemas que cumplen funciones de verificación de identidad, como:

- ANSES  
- AFIP / ARCA  
- MiArgentina  
- ReNaPer  

En términos tecnológicos, mecanismos de autenticación ampliamente utilizados como **OAuth** u **OpenID Connect** permiten integrar servicios de verificación de identidad sin que la plataforma de destino tenga que almacenar directamente credenciales o datos sensibles del usuario.

La plataforma propuesta podría utilizar estos sistemas únicamente para la **verificación inicial del ciudadano**, evitando que la identidad real de la persona circule dentro del sistema participativo.

### Uso de la plataforma estatal de autenticación (AUTENTICAR)

En el caso argentino, una implementación real de este sistema podría integrarse con la **Plataforma de Autenticación Electrónica Central del Estado (AUTENTICAR)**, gestionada por la Jefatura de Gabinete de Ministros.

AUTENTICAR funciona como un intermediario que permite a las aplicaciones validar la identidad de una persona utilizando distintos proveedores de identidad del Estado, como ANSES, AFIP/ARCA, MiArgentina o el Registro Nacional de las Personas (ReNaPer).

De esta forma, la plataforma de participación no necesitaría manejar directamente credenciales de los ciudadanos ni integrarse individualmente con cada organismo, sino que delegaría la autenticación a esta infraestructura estatal ya existente.

## Emisión de identidades anónimas persistentes

Una vez completada la verificación de identidad, el sistema genera una **identidad anónima persistente** que será utilizada para todas las acciones dentro de la plataforma.

Esta identidad no contiene información personal y está diseñada para no incluir datos que identifiquen directamente al ciudadano. Su función es simplemente representar a un participante único dentro del sistema.

Existen diversas tecnologías desarrolladas en el ámbito de la identidad digital y la criptografía que permiten separar la identidad real de la identidad operativa utilizada en un sistema. Algunos ejemplos incluyen:

- **W3C Verifiable Credentials**, un estándar para credenciales digitales verificables  
- **Identidades descentralizadas (DID)**  
- Protocolos basados en **blind signatures** (firmas ciegas)  
- Técnicas de **pruebas de conocimiento cero** (zero-knowledge proofs)

Estas tecnologías permiten demostrar que una persona está habilitada para participar en un sistema sin revelar su identidad real.

La plataforma podría utilizar enfoques inspirados en estas herramientas para emitir identidades anónimas persistentes después de la verificación inicial.

El control de acceso a esa identidad anónima se mantiene exclusivamente mediante la **frase secreta de recuperación** definida por el propio ciudadano durante el registro.

## Arquitectura con separación de funciones

Un principio técnico clave del sistema es la **separación entre verificación de identidad y participación ciudadana**.

En lugar de concentrar todas las funciones en un único sistema, la arquitectura puede dividirse en distintos componentes con responsabilidades diferenciadas. Por ejemplo:

1. Sistema de verificación de identidad  
2. Emisor de identidad anónima  
3. Plataforma participativa

El sistema de verificación confirma que una persona es un ciudadano habilitado.  
El emisor de identidad anónima genera una identidad pseudónima persistente.  
La plataforma participativa gestiona propuestas, votos e interacciones entre ciudadanos.

La separación entre estos componentes reduce significativamente la posibilidad de correlacionar identidades reales con identidades anónimas dentro del sistema.

Este tipo de arquitectura es común en sistemas modernos de privacidad digital, donde la minimización de información compartida entre componentes ayuda a reducir riesgos de exposición de datos.

## Gestión de propuestas y votaciones

Desde el punto de vista tecnológico, la gestión de propuestas y votaciones es una funcionalidad ampliamente conocida.

Existen numerosas plataformas digitales utilizadas en procesos de participación pública, presupuestos participativos o consultas ciudadanas. Estas plataformas permiten publicar propuestas, recolectar apoyo ciudadano y visualizar tendencias colectivas.

La innovación de la propuesta presentada en este documento no reside en el mecanismo de votación o almacenamiento de propuestas, sino en el **modelo de identidad anónima persistente** que permite que cada ciudadano participe de manera continua sin exponer su identidad real.

La plataforma también incorpora reglas de funcionamiento diseñadas para mantener la calidad del sistema, como el límite anual de propuestas por ciudadano y el incentivo a apoyar propuestas existentes antes de crear nuevas iniciativas.

### Vinculación entre propuestas

El sistema permite que las propuestas mantengan referencias explícitas a otras propuestas existentes.

Estas vinculaciones pueden ser declaradas por los ciudadanos al crear una nueva propuesta, indicando por ejemplo que una propuesta amplía, mejora o integra otras previamente publicadas.

Técnicamente, esto puede implementarse mediante referencias entre identificadores de propuestas dentro de la base de datos, permitiendo que la interfaz muestre las relaciones entre ideas sin necesidad de moderación editorial.

## Revisión automática del lenguaje

Las herramientas modernas de procesamiento de lenguaje natural permiten analizar automáticamente grandes volúmenes de texto y detectar patrones asociados a insultos, lenguaje ofensivo o expresiones discriminatorias.

Sistemas de este tipo ya se utilizan ampliamente en plataformas digitales para mejorar la calidad de las conversaciones públicas.

En la plataforma propuesta, el revisor automático de lenguaje actúa únicamente sobre la **forma del texto**, solicitando al autor que revise su redacción cuando se detecta lenguaje inapropiado.

El sistema no evalúa el contenido ideológico de las propuestas ni interviene sobre las ideas que los ciudadanos desean expresar. Su función se limita a promover un estándar mínimo de claridad y respeto en la comunicación.

## Auditoría, transparencia y software abierto

La confianza pública en un sistema de participación ciudadana depende en gran medida de su transparencia.

Una forma de fortalecer esa confianza es desarrollar la plataforma como **software abierto**, permitiendo que cualquier persona pueda examinar el código fuente y comprender cómo funciona el sistema.

El proyecto podría publicarse bajo una licencia permisiva como **Apache License 2.0**, que facilita su adopción institucional y su reutilización por otras organizaciones o países.

El carácter abierto del software permite auditorías independientes y reduce el riesgo de manipulaciones ocultas dentro del sistema.

A nivel de datos, los mecanismos de auditoría también pueden diseñarse para detectar inconsistencias. Por ejemplo, podrían identificarse votos que no estén asociados a identidades anónimas válidas o patrones de comportamiento que sugieran manipulaciones.

Estos mecanismos no eliminan completamente el riesgo de intervención indebida, pero ayudan a que cualquier intento de manipulación resulte detectable.

## Escalabilidad

Desde el punto de vista técnico, la escala del sistema no representa un obstáculo significativo.

Las tecnologías web actuales permiten construir plataformas capaces de manejar millones de usuarios y grandes volúmenes de interacción simultánea.

Existen numerosos ejemplos de servicios digitales que operan a escalas similares o mayores, incluyendo redes sociales, plataformas de comercio electrónico y sistemas de gobierno digital.

Esto demuestra que la infraestructura necesaria para sostener una plataforma de participación ciudadana de gran escala ya está ampliamente disponible.

## Cierre

La propuesta presentada en este documento no requiere inventar tecnologías completamente nuevas.

Las herramientas necesarias para construir una plataforma de participación ciudadana con identidad anónima persistente ya existen en diferentes ámbitos: identidad digital, criptografía aplicada, sistemas participativos y análisis automatizado de lenguaje.

La innovación principal de la propuesta reside en **la arquitectura que integra estas tecnologías** para crear un espacio donde los ciudadanos puedan participar de forma continua sin exponer su identidad pública.

El siguiente documento profundiza en los aspectos técnicos más delicados de esta arquitectura, incluyendo los riesgos de correlación de identidades, los desafíos de auditoría y las consideraciones de seguridad necesarias para implementar un sistema de estas características.
