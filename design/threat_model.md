# Modelo de amenazas

## Propósito

Este documento define el modelo de amenazas asumido por el sistema.

Su objetivo es explicitar:

- qué actores podrían intentar inferir o reconstruir la relación entre la identidad real de un ciudadano y su actividad dentro de la plataforma
- qué capacidades técnicas se considera razonable asumir para esos actores
- qué riesgos el sistema intenta mitigar
- qué riesgos quedan fuera del alcance del diseño

El modelo de amenazas establece los supuestos de seguridad que guían el diseño del sistema, en particular en lo relacionado con verificación de identidad, participación anónima, manejo de metadatos y separación de componentes.

Este documento no describe toda la arquitectura de seguridad, sino el **entorno de amenazas y supuestos de confianza** dentro del cual la arquitectura debe operar.

## Objetivos de seguridad

La arquitectura busca cumplir los siguientes objetivos:

- **Una persona real por participación:** Cada participante activo debe corresponder a una persona real que haya sido verificada mediante un proceso de verificación de identidad.

- **Separación entre identidad real y participación:** El sistema debe permitir que la interacción cotidiana con la plataforma ocurra sin requerir ni exponer la identidad real del ciudadano.

- **Minimización del riesgo de correlación:** La arquitectura debe reducir la posibilidad de que la actividad dentro de la plataforma pueda correlacionarse con la identidad real a partir de los datos almacenados por el sistema.

### Auditabilidad operativa

El diseño debe permitir la revisión externa del funcionamiento del sistema con el fin de detectar manipulaciones, fallos de diseño o desviaciones respecto de los principios declarados.

## No objetivos

El sistema **no está diseñado para garantizar anonimato absoluto**.

En particular, la arquitectura no intenta resistir escenarios como:

- compromiso completo de toda la infraestructura
- vigilancia de red de alcance global
- colusión total entre todos los operadores del sistema
- revelación voluntaria de identidad por parte de los usuarios
- correlaciones sociales externas realizadas fuera de la plataforma

El objetivo del sistema es **reducir la posibilidad práctica de correlación entre identidad real y actividad dentro de la plataforma.**

## Actores considerados

El modelo de amenazas considera los siguientes tipos de actores.

### Observadores públicos

Personas que acceden a la plataforma y pueden ver su contenido público.

Sus capacidades incluyen:

- leer propuestas publicadas
- observar conteos de apoyo
- notar momentos aproximados de actividad
- intentar inferencias sociales a partir de información externa

El sistema no puede impedir inferencias basadas en información que existe fuera de la plataforma.

### Participantes maliciosos

Usuarios que interactúan con la plataforma pero intentan abusar del sistema.

Sus posibles capacidades incluyen:

- intentar ataques de fuerza bruta sobre credenciales débiles
- automatizar interacciones con la plataforma
- analizar patrones de actividad pública
- intentar obtener múltiples identidades

La arquitectura busca limitar este tipo de abuso mediante verificación de ciudadanía, límites operativos y mecanismos básicos de seguridad.

### Atacantes técnicos externos

Individuos o grupos que intentan comprometer la infraestructura del sistema.

Entre sus posibles capacidades se consideran:

- explotación de vulnerabilidades de software
- acceso no autorizado a bases de datos
- abuso de APIs o endpoints
- interceptación de tráfico si la infraestructura está mal configurada

El diseño asume que **un componente aislado podría eventualmente ser comprometido**, por lo que intenta limitar la información disponible en cada componente.

### Operadores de la plataforma de participación

Administradores responsables del funcionamiento del sistema donde viven las propuestas, apoyos y sesiones.

Sus posibles accesos incluyen:

- bases de datos de participación
- registros operativos
- configuración de infraestructura
- sistemas de monitoreo

La arquitectura debe asegurar que este componente **no requiera acceso a la identidad real de los participantes**.

### Operadores del sistema de verificación de identidad

Entidades responsables de verificar que un participante corresponde a una persona real.

Sus capacidades incluyen:

- acceder a la identidad real durante el proceso de verificación
- registrar eventos de verificación
- habilitar la emisión de una identidad anónima

Este componente no debería tener visibilidad sobre la actividad del ciudadano dentro de la plataforma participativa.

### Servicio de emisión o gestión de identidad anónima

Componente encargado de administrar la existencia de identidades anónimas y aplicar reglas operativas como emisión, revocación o reemisión.

Sus capacidades pueden incluir:

- confirmar la existencia de una identidad anónima válida
- aplicar reglas operativas
- almacenar identificadores mínimos necesarios para esas reglas

Este componente no debería tener acceso directo a la actividad dentro de la plataforma participativa.

### Colusión entre componentes

Un escenario más exigente ocurre cuando operadores de distintos componentes cooperan o son obligados a cooperar.

Por ejemplo, combinando información entre:

- sistemas de verificación de identidad
- servicios de emisión de identidad anónima
- plataformas de participación

La arquitectura busca **reducir la información almacenada en cada sistema** para dificultar este tipo de correlaciones.

Sin embargo, en un sistema centralizado no es posible garantizar protección completa frente a colusión total entre todos los componentes.

### Requerimientos legales

Autoridades judiciales o regulatorias pueden exigir acceso a información disponible en los sistemas operativos o solicitar la preservación de registros.

El sistema no puede impedir la entrega de información que efectivamente exista en su infraestructura.

Por esta razón, el diseño enfatiza:

- minimización de datos almacenados
- retención limitada de registros operativos
- separación entre identidad real y actividad dentro de la plataforma

## Capacidades asumidas

El modelo de amenazas asume que atacantes o incidentes operativos podrían permitir acceso a:

- bases de datos de participación
- registros operativos
- copias de respaldo
- datos de sesión
- marcas de tiempo de eventos
- sistemas de monitoreo o telemetría

El diseño del sistema debe minimizar la posibilidad de que esta información permita reconstruir la relación entre identidad real y participación.

## Principios de arquitectura derivados del modelo de amenazas

A partir de este modelo de amenazas se derivan varios principios de diseño.

### Separación de funciones

Las distintas responsabilidades del sistema deben manejarse mediante componentes diferenciados.

En particular:

- verificación de identidad
- gestión de identidad anónima
- operación de la plataforma de participación

deben poder operar de forma separada.

### Minimización de datos

Cada componente del sistema debe almacenar únicamente la información estrictamente necesaria para cumplir su función.

Siempre que sea posible, deben evitarse datos que puedan facilitar la correlación entre identidad real y actividad dentro de la plataforma.

### Retención limitada de metadatos

Los metadatos operativos deben conservarse únicamente durante el tiempo necesario para garantizar la estabilidad y seguridad del sistema.

### Auditabilidad

El diseño del sistema debe permitir la revisión independiente de sus principios y comportamiento.

Esto incluye documentación abierta y la posibilidad de inspección técnica por terceros.

## Evolución del modelo de amenazas

Este documento describe el modelo de amenazas actualmente asumido por el sistema.
A medida que la arquitectura evolucione, podrán identificarse nuevos escenarios de riesgo o introducirse nuevas mitigaciones.

Los cambios relevantes en el modelo de amenazas deberán documentarse y, cuando corresponda, registrarse mediante un Architecture Decision Record (ADR)
