# P-0004 — Autenticación de identidad anónima

**Estado:** Activo

## Contexto

Después del registro inicial mediante verificación de identidad estatal, el ciudadano obtiene una identidad anónima persistente que usa dentro de la aplicación.

A partir de ese momento, el uso cotidiano de la aplicación debe quedar completamente desacoplado de cualquier sistema de identidad real.

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

El ciudadano debería autenticarse nuevamente mediante AUTENTICAR (ANSES, ARCA/AFIP, MiArgentina, ReNaPer) u otro proveedor estatal cada vez que quisiera ingresar.

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

# Implicancias para el reingreso

Dado que la identidad anónima funciona como identificador de acceso, la aplicación normaliza el pseudónimo ingresado por el ciudadano al momento del login para tolerar variaciones tipográficas. La aplicación ignora mayúsculas/minúsculas, acentos, espacios adicionales y guiones (medios y bajos): el ciudadano puede escribir variantes equivalentes y todas son aceptadas.

Ejemplo:

```
Tejón Versátil 91
Tejon Versátil 91
Tejon Versatil 91
tejon versatil 91
tejonversatil91
tejon_versatil-91
```

La estructura del pseudónimo (`animal + color/adjetivo + número`) está decidida en P-0002. La selección y curaduría de las listas de palabras está documentada en `design/capa_de_identidad/identity_wordlists.md`.

Esta decisión simplifica la experiencia de acceso a la aplicación y refuerza la coherencia conceptual entre identidad anónima y autenticación.
