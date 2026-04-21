# P-0008 — Mecanismo de credencial de acceso: passphrase vs password

## Contexto

Una vez creada la identidad anónima, el ciudadano necesita un mecanismo para volver a acceder a la plataforma en visitas posteriores. Según lo definido en P-0004, ese mecanismo es la combinación de identidad anónima (pseudónimo) más una credencial secreta definida por el propio ciudadano durante el registro.

La credencial secreta es el único mecanismo de acceso y recuperación de la identidad anónima. No existe recuperación alternativa por correo electrónico, teléfono ni intervención de administradores. Si el ciudadano la pierde, su identidad anónima queda irrecuperable y deberá esperar un período de cool-down antes de poder registrar una nueva (por defecto seis meses, configurable por el operador).

Este contexto hace que la elección del tipo de credencial tenga consecuencias directas sobre la memorabilidad, la seguridad y el modelo de anonimato del sistema.

## Opciones consideradas

### Opción 1 — Password tradicional

El ciudadano define una contraseña siguiendo reglas típicas: longitud mínima, combinación de mayúsculas, números y símbolos.

**Ventajas**

- Formato familiar para la mayoría de los usuarios.
- Fácil de implementar con bibliotecas estándar.

**Desventajas**

- Alta probabilidad de reutilización de contraseñas existentes (correo, redes sociales). Si esa contraseña se filtra en otro sistema, la identidad anónima queda expuesta.
- Las reglas de complejidad típicas generan contraseñas difíciles de recordar, lo que empuja al usuario a anotarlas. Dado que no existe recuperación alternativa, perder la anotación equivale a perder la identidad anónima.
- La reutilización introduce un riesgo de correlación: un atacante que conozca la contraseña de otra cuenta del ciudadano podría intentar usarla aquí.

### Opción 2 — Passphrase

El ciudadano define una frase de varias palabras, por ejemplo: "mi perro come pizza los jueves".

**Ventajas**

- Alta entropía con frases de cuatro palabras o más, superando en seguridad a la mayoría de los passwords tradicionales.
- Mucho más fácil de memorizar, reduciendo la necesidad de anotarla.
- Menor probabilidad de reutilización: las frases son personales e improbablemente coincidan con credenciales usadas en otros sistemas.
- Coherente con el modelo del sistema: si el ciudadano puede memorizar su identidad anónima (pseudónimo), también puede memorizar una frase propia.

**Desventajas**

- Formato menos familiar para usuarios acostumbrados a contraseñas cortas.
- Requiere comunicar claramente el concepto al ciudadano durante el registro.

## Decisión

Se adopta la **Opción 2 — Passphrase**.

El ciudadano define libremente su frase secreta, sujeta a los siguientes criterios mínimos configurables:

- Mínimo de 4 palabras.
- Mínimo de 20 caracteres en total.

El sistema muestra durante el registro un ejemplo o sugerencia para que el ciudadano comprenda el concepto, dado que muchos usuarios no están familiarizados con el término "frase secreta" y solo conocen el concepto de contraseña o clave.

El sistema normaliza la frase al momento de validarla, ignorando diferencias de mayúsculas, acentos y espacios adicionales, para reducir errores de tipeo.

## Justificación

La passphrase resuelve el problema más crítico del sistema: dado que no existe recuperación alternativa de la identidad anónima, la credencial debe ser memorable sin necesidad de anotarla. Un password tradicional con reglas de complejidad típicas es difícil de recordar y fácil de reutilizar. Una frase personal es lo opuesto: fácil de recordar y muy improbable de coincidir con credenciales usadas en otros sistemas.

La reutilización de passwords es además un riesgo de correlación: si una contraseña usada en otra plataforma se filtra, podría usarse para acceder a la identidad anónima del ciudadano. Una passphrase personal y única elimina ese vector.

## Consecuencias

- El sistema debe comunicar claramente el concepto de passphrase durante el registro, con un ejemplo o sugerencia.
- Los criterios mínimos (cantidad de palabras y longitud total) son configurables por el operador de la plataforma.
- El sistema aplica normalización al validar la frase: ignora mayúsculas, acentos y espacios adicionales.
- No existe control de calidad adicional más allá de los criterios mínimos: el ciudadano es libre de elegir la frase que considere memorable.
