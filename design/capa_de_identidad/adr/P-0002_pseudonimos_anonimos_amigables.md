# P-0002 — Representación de identidades anónimas mediante pseudónimos amigables

**Estado:** Activo

## Contexto

La plataforma asigna a cada ciudadano una **identidad anónima persistente** que le permite participar en el sistema sin revelar su identidad real.

Desde el punto de vista interno del sistema, cada identidad anónima puede representarse mediante un identificador técnico único (por ejemplo, un UUID u otro tipo de identificador interno). Sin embargo, surgió la necesidad de definir cómo representar esa identidad frente al propio usuario dentro de la interfaz.

El objetivo de esta representación es permitir que el ciudadano pueda reconocer su identidad anónima dentro del sistema y verificar que está participando siempre con la misma identidad.

Durante el diseño surgió la pregunta de cómo debía representarse esa identidad en la interfaz del usuario.

## Opciones consideradas

### Opción 1 — Identificador técnico o pseudónimo frío

En esta opción, la identidad anónima se mostraría mediante un identificador técnico o pseudónimo numérico, por ejemplo:

*Anon#1729*  
*User-8F3A91*  
*ID-A17K92*  

**Ventajas**

- Es simple de implementar.
- Garantiza unicidad fácilmente.
- Es un formato común en sistemas informáticos.

**Desventajas**

- Resulta impersonal y poco amigable para los usuarios.
- Es difícil de recordar o reconocer rápidamente.
- Refuerza la sensación de estar interactuando con un identificador técnico en lugar de una identidad dentro del sistema.

### Opción 2 — Pseudónimos amigables generados automáticamente

En esta opción, el sistema genera automáticamente pseudónimos compuestos por palabras fácilmente reconocibles, por ejemplo mediante la combinación:

*Animal + Color (o adjetivo) + Número*

Ejemplos posibles:

*Lobo Azul 714*  
*Puma Silencioso 981*   
*Tigre Dorado 442*  

El número garantiza la unicidad del pseudónimo, mientras que el resto del nombre aporta una forma más humana y fácilmente reconocible de representar la identidad.

**Ventajas**

- Resulta más amigable y fácil de recordar para los usuarios.
- Permite que el ciudadano reconozca rápidamente su identidad anónima dentro del sistema.
- Humaniza la interfaz de la plataforma.
- Mantiene la unicidad mediante el componente numérico.

**Desventajas**

- Requiere mantener listas de palabras (animales, colores o adjetivos).
- Introduce una pequeña complejidad adicional en la generación de pseudónimos.

## Decisión

Se adopta la **Opción 2 — Pseudónimos amigables generados automáticamente**.

Las identidades anónimas se representarán mediante pseudónimos compuestos por la combinación de:

*Animal + Color (o adjetivo) + Número*

El número actúa como elemento de diferenciación que garantiza la unicidad del pseudónimo.

Es importante destacar que este pseudónimo es **solo una representación visible para el propio usuario** y no constituye el identificador interno utilizado por el sistema.

## Justificación

El objetivo principal de esta decisión es mejorar la experiencia del usuario dentro de la plataforma.

Aunque la identidad anónima es técnicamente un identificador interno del sistema, mostrar únicamente un identificador técnico podría resultar impersonal y dificultar que los ciudadanos reconozcan su propia identidad dentro de la plataforma.

El uso de pseudónimos amigables:

- facilita que el ciudadano identifique su identidad anónima persistente
- aporta una interfaz más humana y accesible
- evita la exposición de identificadores técnicos internos

Al mismo tiempo, el componente numérico permite escalar el sistema a grandes volúmenes de usuarios manteniendo pseudónimos únicos.

## Consecuencias

- Cada identidad anónima tiene asociado un pseudónimo amigable generado automáticamente.
- El pseudónimo se muestra únicamente al propio usuario dentro de la interfaz.
- El sistema utiliza internamente identificadores técnicos independientes del pseudónimo visible.
- La convención de nombres permite escalar a grandes cantidades de usuarios manteniendo identificadores fáciles de reconocer.
