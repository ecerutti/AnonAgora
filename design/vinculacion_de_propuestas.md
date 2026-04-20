# Vinculación entre propuestas

## Propósito

Este documento describe el mecanismo de vinculación entre propuestas dentro de la plataforma.

## Mecanismo

Una propuesta puede declarar vínculos a otras propuestas existentes en el momento de su publicación. Un vínculo es una referencia genérica sin tipo: expresa que existe una relación entre dos propuestas, sin calificar la naturaleza de esa relación. La interfaz puede presentar los vínculos con etiquetas descriptivas, pero esa es una decisión de presentación, no del modelo de datos.

El número máximo de vínculos por propuesta es configurable por el operador, con un valor por defecto de 10.

## Propiedades derivadas de P-0018

**Los vínculos son inmutables.** Al ser parte de la propuesta, quedan fijados en el momento de publicación. No pueden agregarse ni eliminarse posteriormente, por la misma razón que el texto de la propuesta es inmutable.

**Los vínculos no requieren aceptación.** La plataforma no almacena la autoría de las propuestas, por lo que no existe mecanismo para identificar ni notificar al autor de una propuesta referenciada. Cualquier ciudadano puede vincular su propuesta a cualquier otra sin intervención de terceros.

## Propuestas derivadas

Una propuesta derivada es una propuesta que declara un vínculo a otra propuesta de la cual toma como base. No es una entidad distinta en el modelo de datos: es una propuesta común con uno o más vínculos genéricos. La interfaz puede ofrecer un flujo de creación específico para propuestas derivadas, pero el modelo subyacente es idéntico al de cualquier propuesta con vínculos.

La nomenclatura jerárquica fue descartada porque las derivaciones pueden formar un grafo, no un árbol: múltiples propuestas pueden derivar de una misma propuesta, y una propuesta puede integrar ideas de varias propuestas simultáneamente. Cualquier esquema de identificadores que intente reflejar esa estructura se vuelve ambiguo o caótico cuando el grafo se expande.

## Modelo de datos

Los vínculos se almacenan como referencias desde la propuesta que los declara hacia las propuestas referenciadas. Una propuesta no tiene conocimiento de cuántas otras propuestas la referencian ni quiénes son.
