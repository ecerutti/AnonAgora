# Identity Wordlists — Generación de Identidades Anónimas

## Propósito

Las listas de palabras utilizadas para generar identidades anónimas tienen un doble objetivo:

* producir identificadores **fáciles de recordar y escribir**
* evitar combinaciones ofensivas, ambiguas o desagradables

Por esta razón las listas han sido **curadas** siguiendo criterios lingüísticos y de usabilidad.

Estas listas deben considerarse **conceptuales**, no exhaustivas.

## Estructura de la identidad anónima

Las identidades anónimas se generan automáticamente utilizando el siguiente formato:

```
animal + color/adjetivo + número
```

Ejemplos:

```
Lobo Azul 714
Puma Sereno 84
```

El número se genera en el rango:

```
1..999
```

No se utilizan ceros a la izquierda para facilitar lectura y memorización.

## Normalización en el ingreso

Aunque las identidades se muestran respetando la ortografía correcta (incluyendo acentos), el sistema normaliza el texto ingresado durante el login.

Se ignoran:

* mayúsculas/minúsculas
* acentos
* espacios adicionales
* guiones medios
* guiones bajos

Por ejemplo, todas las siguientes entradas deben ser consideradas equivalentes:

```
Lobo Azul 714
lobo azul 714
loboazul714
lobo_azul-714
```

## Criterios de curación del vocabulario

Las listas de palabras han sido seleccionadas aplicando los siguientes criterios.

### Neutralidad semántica

Se excluyen palabras que puedan resultar:

* ofensivas
* descalificatorias
* cultural o políticamente cargadas
* potencialmente humillantes para el usuario

Ejemplos de términos deliberadamente excluidos:

```
Vibora
Rata
Serpiente
Gorila
Gato
```

### Longitud limitada

Las palabras tienen un máximo de **7 letras**.

Este límite busca mantener identidades:

* cortas
* fáciles de escribir
* fáciles de recordar
* visualmente simples

### Legibilidad

Se priorizan palabras:

* conocidas
* fáciles de escribir
* fáciles de pronunciar
* sin caracteres especiales

Se permiten acentos, pero el sistema los ignora durante la autenticación.

### Combinaciones agradables

Se evitan combinaciones potencialmente desagradables como:

```
Rata Sucia
Vibora Rastrera
Cerdo Asqueroso
```

Esto se logra mediante curación cuidadosa de las listas de animales y adjetivos.

### Concordancia gramatical

Para evitar discordancias gramaticales con los animales utilizados, los adjetivos deben ser **neutros en género.**

Esto significa que no deben tener terminaciones masculinas o femeninas marcadas (por ejemplo -o o -a).

Ejemplo correcto:

```
Hormiga Alegre
Abeja Sagaz
Pulpo Feliz
```

Ejemplo incorrecto (evitado):

```
Hormiga Mimoso
Abeja Sereno
```

## Tamaño del espacio de identidades

Con las listas actuales:

```
animales: 45
colores: 14
adjetivos: 15
```

El número total de combinaciones posibles es:

```
45 × (14 + 15) × 999
```

Resultado:

```
1.303.695 identidades únicas
```

Este tamaño se considera **suficiente para una implementación conceptual o piloto**.

## Escalabilidad futura

Si la plataforma necesitara soportar un número mayor de ciudadanos, existen dos estrategias simples.

### 1. Ampliar las listas de palabras

Las listas ubicadas en:

```
design/wordlists/
```

pueden ampliarse agregando nuevas palabras que cumplan los criterios de curación.

### 2. Aumentar el rango numérico

El rango podría extenderse fácilmente a:

```
1..9999
```

lo que multiplicaría el espacio de identidades por 10.

## Naturaleza conceptual de las listas

Las listas actuales deben entenderse como **una base conceptual cuidadosamente curada**.

Su objetivo es:

* validar el modelo de identidad anónima
* demostrar su funcionamiento
* garantizar identidades amigables para el usuario

En una implementación a escala nacional, es esperable que estas listas se amplíen para aumentar el espacio total de identidades.
