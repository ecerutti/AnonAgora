# P-0004 — Autenticación de identidad anónima

## Contexto

Después del registro inicial mediante verificación de identidad estatal, el ciudadano obtiene una identidad anónima persistente dentro de la plataforma.

A partir de ese momento, el uso cotidiano de la plataforma debe quedar completamente desacoplado de cualquier sistema de identidad real.

El problema de diseño a resolver es:

**Cómo vuelve a ingresar el ciudadano a la plataforma en visitas posteriores?**

El mecanismo de acceso debe cumplir las siguientes condiciones:

- no depender nuevamente de autenticación estatal
- funcionar desde cualquier dispositivo
- no requerir identificadores técnicos difíciles de recordar
- mantener una experiencia de uso comprensible para ciudadanos
- reforzar la percepción de anonimato del sistema
- evitar mecanismos que sugieran que la plataforma "recuerda" la identidad real del usuario

## Opciones consideradas

### 1. Reautenticación estatal en cada ingreso

El ciudadano debería autenticarse nuevamente mediante la plataforma AUTENTICAR (ANSES, ARCA/AFIP, MiArgentina, ReNaPer) u otro proveedor estatal cada vez que quisiera ingresar.

**Problemas**

- rompe el desacople entre identidad real y uso cotidiano de la plataforma
- refuerza la percepción de vigilancia o seguimiento estatal
- introduce fricción innecesaria en el uso diario

**Resultado**

Descartado.

### 2. Frase secreta como único medio de acceso

El ciudadano ingresaría únicamente introduciendo su frase secreta.

**Problemas**

- posibles colisiones entre frases secretas
- la frase funcionaría simultáneamente como identificador y secreto
- experiencia de uso poco familiar para los usuarios

**Resultado**

Descartado.

### 3. Identificador técnico corto + frase secreta

El sistema entregaría al ciudadano un identificador técnico adicional que debería usar junto con su frase secreta.

Ejemplo:

```
ID: AN-4K7P-92LM
```

**Problemas**

- introduce un elemento difícil de recordar
- empuja al usuario a anotarlo en algún lugar
- aumenta la complejidad innecesariamente

**Resultado**

Descartado.

### 4. Uso del identidad anónima como identificador de acceso

El ciudadano utiliza:

- su identidad anónima
- su frase secreta

para volver a ingresar a la plataforma.

Ejemplo:

```
Identidad anónima: Pato Naranja 72
Frase secreta: ...
```

**Ventajas**

- no introduce identificadores adicionales
- mantiene coherencia conceptual con la identidad anónima
- es comprensible para el usuario
- funciona desde cualquier dispositivo

**Resultado**

Seleccionado.

## Decisión

El reingreso a la plataforma se realiza mediante:

- identidad anónima
- frase secreta

La frase secreta funciona como prueba de posesión de dicha identidad.

# Diseño de la identidad anónima

Dado que la identidad anónima también funciona como identificador de acceso, su diseño debe cumplir ciertas condiciones.

## Estructura

La identidad anónima se generan automáticamente mediante una combinación de:

```
animal + color/adjetivo + número
```

Ejemplos:

```
Pato Naranja 72
Tejón Versátil 91
```

El segundo término puede ser:

- un color
- un adjetivo corto

Esto permite aumentar el espacio de combinaciones disponibles.

## Restricciones del vocabulario

Las listas de animales, colores y adjetivos deben cumplir los siguientes criterios:

- palabras cortas
- fáciles de escribir
- sin caracteres especiales
- preferentemente sin tildes ni acentos

El objetivo es evitar problemas de escritura y facilitar el uso de la identidad anónima como identificador de acceso.

## Curaduría del vocabulario

Las listas deben ser cuidadosamente curadas para evitar combinaciones potencialmente ofensivas o desagradables.

Ejemplos de combinaciones a evitar:

```
Vibora Rastrera
Rata Asquerosa
```

## Normalización de entrada

Al momento de autenticarse, el sistema debe aceptar la identidad anónima en forma normalizada.

Esto implica ignorar:

- mayúsculas o minúsculas
- acentos y tildes
- espacios adicionales
- guiones

Ejemplo:

```
Tejón Versátil 91
Tejon Versátil 91
Tejon Versatil 91
tejon versatil 91
tejonversatil91
tejon_versatil-91
```

Todas estas variantes deben considerarse equivalentes.

## Impacto

Esta decisión simplifica la experiencia de acceso al sistema y refuerza la coherencia conceptual entre identidad anónima y autenticación.
