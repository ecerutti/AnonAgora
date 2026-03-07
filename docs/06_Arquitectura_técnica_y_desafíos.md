# 06 — Arquitectura técnica y desafíos de implementación

## Introducción

El diseño de una plataforma de participación ciudadana con identidad anónima persistente plantea desafíos técnicos importantes. 

A diferencia de muchas plataformas digitales tradicionales, el objetivo no es únicamente gestionar cuentas de usuario, propuestas o votaciones, sino hacerlo preservando una separación robusta entre la identidad real del ciudadano y su identidad anónima dentro del sistema.

Este documento describe los principales desafíos técnicos asociados a ese objetivo y algunas de las estrategias que podrían utilizarse para abordarlos.

No pretende definir una implementación definitiva, sino demostrar que los problemas relevantes son conocidos y que existen enfoques técnicos plausibles para mitigarlos.

Un principio importante a tener en cuenta es que la privacidad de un sistema no depende únicamente de la criptografía utilizada, sino también de las decisiones de arquitectura y de los detalles de implementación.

## Separación entre identidad real e identidad anónima

El principio central de la arquitectura propuesta es la separación entre el proceso de verificación de identidad y el funcionamiento cotidiano de la plataforma participativa.

La verificación de identidad tiene como único objetivo confirmar que una persona es un ciudadano real y habilitado para participar. Una vez realizada esa verificación, el sistema genera una identidad anónima persistente que será utilizada dentro de la plataforma.

A partir de ese momento, la identidad real del ciudadano no debería formar parte del funcionamiento normal del sistema participativo.

Esta separación puede implementarse mediante distintos enfoques técnicos, incluyendo arquitecturas con componentes independientes o el uso de tecnologías diseñadas para desacoplar credenciales de identidad y credenciales de participación, como:

- W3C Verifiable Credentials  
- Identidades descentralizadas (DID)  
- Blind signatures  
- Pruebas de conocimiento cero (Zero-Knowledge Proofs)

Estas tecnologías permiten demostrar que un usuario está habilitado para participar sin revelar su identidad real.

## Riesgo de correlación entre sistemas

Incluso cuando la verificación de identidad y la plataforma participativa se implementan como componentes separados, existe el riesgo de correlación indirecta entre identidades.

Este tipo de problema se conoce comúnmente como **correlation attack**.

Por ejemplo, si distintos sistemas registran eventos con marcas temporales precisas, un operador con acceso a múltiples registros podría inferir relaciones entre ellos.

Un caso simple sería:

Sistema de verificación de identidad:

`10:32:11 ciudadano verificado`

Sistema de emisión de identidad anónima:

`10:32:13 identidad anon_847223 creada`


Aunque los sistemas estén separados, la correlación temporal puede permitir inferir la relación entre ambos eventos.

Por esta razón, el diseño del sistema debe considerar cuidadosamente la generación de registros, logs y metadatos para evitar que la correlación indirecta se convierta en una fuente de desanonimización.

## Protección de metadatos

En muchos sistemas de privacidad, el principal riesgo no proviene del acceso directo a datos personales, sino del análisis de metadatos.

Entre los metadatos potencialmente sensibles se incluyen:

- marcas temporales exactas  
- direcciones IP  
- identificadores de sesión  
- patrones de conexión  
- telemetría del sistema  

Incluso cuando los datos principales están protegidos, estos elementos pueden permitir inferencias sobre el comportamiento de los usuarios.

El diseño de la plataforma debería minimizar la exposición de este tipo de información, evitando por ejemplo correlaciones directas entre eventos sensibles y reduciendo la cantidad de metadatos persistentes almacenados por el sistema.

## Prevención de identidades múltiples (Sybil attacks)

Uno de los problemas clásicos en sistemas participativos abiertos es el llamado **Sybil attack**, en el que una misma persona crea múltiples identidades para simular apoyo social inexistente.

La plataforma aborda este problema mediante la verificación inicial de identidad del ciudadano.

Este proceso permite sostener una regla básica del sistema:

**una persona, una única identidad dentro de la plataforma.**

Una vez emitida la identidad anónima persistente, el ciudadano puede participar libremente sin exponer su identidad real, pero sin posibilidad de multiplicar artificialmente su presencia dentro del sistema.

## Coerción y presión externa

En sistemas de participación política puede existir presión externa sobre los participantes para revelar o demostrar su comportamiento dentro del sistema.

Una de las características de esta plataforma es que las acciones de participación no están asociadas públicamente a la identidad real del ciudadano.

Además, el sistema permite modificar decisiones con el tiempo, lo que reduce la posibilidad de demostrar de forma permanente el comportamiento pasado de un usuario.

Aunque ningún sistema puede eliminar completamente el riesgo de coerción externa, estas características ayudan a reducir su impacto potencial.

## Auditoría y detección de manipulación

Como en cualquier sistema informático, existe la posibilidad de que operadores con acceso privilegiado intenten manipular datos internos, por ejemplo alterando contadores de votación o modificando registros.

El objetivo del diseño no es necesariamente hacer imposible cualquier manipulación —algo extremadamente difícil en sistemas centralizados— sino hacer que cualquier intento resulte detectable.

Entre los mecanismos que podrían utilizarse se encuentran:

- registros de auditoría verificables  
- detección de inconsistencias entre votos e identidades anónimas  
- verificación periódica de integridad de datos  
- identidades *canario* diseñadas para detectar alteraciones del sistema  

Estos mecanismos permiten aumentar la transparencia y facilitar la detección de intervenciones indebidas.

## Protección de la frase secreta de recuperación

La frase secreta definida por el ciudadano durante el registro constituye el único mecanismo de acceso y recuperación de su identidad anónima.

Para proteger su seguridad, el sistema no debería almacenar esta frase en texto plano. En su lugar, puede utilizar técnicas estándar de derivación segura de credenciales, como funciones de hashing con *salt* y algoritmos diseñados para proteger contraseñas.

El diseño deliberadamente evita mecanismos alternativos de recuperación basados en datos personales (como correo electrónico o número de teléfono), ya que estos podrían introducir nuevas superficies de correlación entre la identidad real del ciudadano y su identidad anónima.

Si un ciudadano pierde su frase secreta, su identidad anónima queda irrecuperable y deberá crear una nueva identidad siguiendo las reglas del sistema.

## Transparencia y software abierto

La transparencia es un componente fundamental para generar confianza en una plataforma de participación ciudadana.

El desarrollo del sistema como **software abierto** permite que cualquier persona o institución pueda examinar el código fuente, evaluar su arquitectura y detectar posibles debilidades.

El uso de una licencia permisiva como **Apache License 2.0** facilita la adopción del sistema por diferentes instituciones y permite su reutilización en otros países o contextos institucionales.

La auditabilidad pública del software no elimina todos los riesgos, pero contribuye significativamente a la confianza en el funcionamiento del sistema.

## Cierre

La construcción de una plataforma de participación ciudadana con identidad anónima persistente implica enfrentar desafíos técnicos relevantes relacionados con privacidad, seguridad y arquitectura de sistemas.

Muchos de estos desafíos ya han sido estudiados en ámbitos como la identidad digital, la criptografía aplicada y los sistemas de votación electrónica.

Si bien una implementación real requeriría un diseño cuidadoso y revisiones técnicas rigurosas, las herramientas conceptuales y tecnológicas necesarias para abordar estos problemas ya existen.

El desafío principal no es la ausencia de tecnología, sino el diseño de una arquitectura que combine estas herramientas de manera coherente para preservar la privacidad de los ciudadanos sin sacrificar la integridad del sistema.
