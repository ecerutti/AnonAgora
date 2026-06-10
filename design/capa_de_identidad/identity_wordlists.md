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
animal + color/adjetivo + número [+ letra]
```

Ejemplos:

```
Lobo Azul 714
Puma Sereno 84
Tigre Audaz 23A
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
Tigre Audaz 23A
tigre audaz 23a
tigreaudaz23a
tigre_audaz-23a
```

La unicidad de pseudónimos (P-0025) se evalúa sobre esta misma forma normalizada. Por eso las listas no deben contener pares de palabras que colisionen al normalizar (por ejemplo, una misma palabra con y sin tilde).

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

El mismo criterio aplica a los colores: las listas usan únicamente colores invariantes en género (Azul, Gris, Verde, etc.), evitando combinaciones discordantes como *Abeja Rojo* o *Jirafa Blanco*.

## Tamaño del espacio de identidades

Con las listas actuales:

```
animales: 59
colores: 9
adjetivos: 17
```

El número total de combinaciones posibles es:

```
59 × (9 + 17) × 999
```

Resultado:

```
1.532.466 identidades únicas
```

Este tamaño se considera **suficiente para una implementación conceptual o piloto**.

## Sufijo opcional de letra mayúscula

Cuando el espacio sin sufijo de letra se satura, el emisor activa empíricamente un sufijo de una letra mayúscula al final del número (`Lobo Azul 714H`, `Tigre Audaz 23A`). La activación es consecuencia del mecanismo de generación: si tras un umbral de intentos consecutivos sin sufijo (configurable, default 10) todos los candidatos resultan ocupados según la consulta-con-reserva a la aplicación destino (P-0025), el emisor genera candidatos con sufijo de letra.

Las letras admitidas son 21:

```
A C D E F G H J K L M N P Q R T U V W X Y
```

Se excluyen O, I, S, Z, B por confusión visual con dígitos (0, 1, 5, 2, 8 respectivamente) y la Ñ por no integrar el conjunto base ASCII.

El sufijo agrega:

```
59 × 26 × 999 × 21 = 32.181.786 combinaciones
```

que, sumadas a las combinaciones sin sufijo, llevan el espacio total a **33.714.252** identidades posibles.

El sistema no mantiene estado explícito de agotamiento del espacio sin sufijo. La activación es natural y gradual: con el espacio sin letra mayormente disponible, los ciudadanos reciben pseudónimos sin sufijo; con el espacio sin letra saturado, los reciben con sufijo.

## Naturaleza conceptual de las listas

Las listas actuales deben entenderse como **una base conceptual cuidadosamente curada**.

Su objetivo es:

* validar el modelo de identidad anónima
* demostrar su funcionamiento
* garantizar identidades amigables para el usuario

En una implementación a escala nacional, es esperable que estas listas se amplíen para aumentar el espacio total de identidades.
