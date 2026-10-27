# Protocolo de la frase secreta

## Propósito

Este documento especifica el algoritmo exacto de normalización y hash de la frase secreta que el cliente aplica antes de enviarla a la plataforma. P-0008 establece que la frase se normaliza en el cliente, de forma determinista e idéntica en registro y login, ignorando mayúsculas, acentos y espacios adicionales. P-0009 establece que el cliente aplica una función hash estándar sobre la forma normalizada y envía solo ese hash, sobre el cual la plataforma aplica Argon2id. Ambos ADRs exigen que el algoritmo quede especificado como protocolo entre cliente y plataforma, para que cualquier implementación de cliente sea compatible.

Cualquier cambio a este protocolo invalida las credenciales existentes: la frase de un ciudadano registrado produciría un hash distinto. Por eso el protocolo lleva versión, y un cambio requiere una versión nueva y una estrategia de migración.

## Normalización (versión 1)

Dada la frase tal como la escribió el ciudadano, el cliente aplica en orden:

1. **Minúsculas.** Conversión a minúsculas con el mapeo Unicode por defecto, independiente del idioma configurado (en JavaScript, `toLowerCase()`, no `toLocaleLowerCase()`).
2. **Descomposición.** Normalización Unicode NFKD. Separa cada letra de sus marcas diacríticas ("á" pasa a "a" más el acento) y unifica variantes tipográficas de compatibilidad (ligaduras, caracteres de ancho completo).
3. **Eliminación de marcas diacríticas.** Se eliminan todos los caracteres de la categoría Unicode `M` (marcas), con una única excepción: la virgulilla combinante (U+0303) que sigue inmediatamente a una "n" se conserva.
4. **Recomposición.** Normalización Unicode NFC. Recompone "n" más virgulilla en "ñ".
5. **Solo letras y números.** Todo carácter que no pertenezca a las categorías Unicode `L` (letras) o `N` (números) se reemplaza por un espacio. Esto elimina puntuación, guiones, símbolos y emojis.
6. **Espacios.** Se eliminan los espacios al principio y al final, y toda secuencia de espacios se reduce a uno solo.

Decisiones incorporadas:

- **Acentos y diéresis se ignoran** ("pingüino" = "pinguino", "Ñandú" = "ñandu"), según P-0008.
- **La ñ se conserva.** En castellano es una letra distinta, no una "n" acentuada: "año" y "ano" son frases distintas.
- **La puntuación se ignora**, igual que los símbolos: "hola, mundo" = "hola mundo". Extiende la tolerancia a errores de tipeo que busca P-0008.

## Validación de mínimos

Los criterios mínimos configurables de P-0008 (default: 4 palabras y 20 caracteres) se evalúan sobre la **forma normalizada**:

- **Palabras:** cantidad de segmentos separados por espacio.
- **Caracteres:** cantidad de puntos de código Unicode, incluidos los espacios.

La validación ocurre en el cliente, para dar respuesta inmediata al ciudadano, y no puede repetirse en la plataforma, que nunca ve la frase. Un cliente que omita la validación solo debilita la frase de su propio usuario.

## Hash (versión 1)

```
hash_cliente = hex( SHA-256( UTF-8( "frase-v1:" + forma_normalizada ) ) )
```

- El prefijo `frase-v1:` identifica la versión del protocolo y separa este uso de SHA-256 de cualquier otro.
- El resultado se codifica en hexadecimal en minúsculas (64 caracteres).
- El cliente envía únicamente `hash_cliente`. La plataforma le aplica Argon2id según P-0009.

## Vectores de prueba

Toda implementación del protocolo debe reproducir exactamente estos resultados.

| Frase ingresada | Forma normalizada | Palabras / caracteres | `hash_cliente` |
|---|---|---|---|
| `El Año pasado fuimos a Mar del Plata` | `el año pasado fuimos a mar del plata` | 8 / 36 | `aa22c4566651186e83a1bce4f54b76cdf844ce8d766cd38eb81893e0cab9ead3` |
| `  el año, pasado... FUIMOS a mar-del-plata!! ` | `el año pasado fuimos a mar del plata` | 8 / 36 | `aa22c4566651186e83a1bce4f54b76cdf844ce8d766cd38eb81893e0cab9ead3` |
| `Mi pingüino Ñandú come 3 manzanas 🍎` | `mi pinguino ñandu come 3 manzanas` | 6 / 33 | `8c823a8bb07954d9e75110f144f1dac8071e0478dfa8132954cfd434934b7210` |
| `mi PINGUINO ñandu come 3 manzanas` | `mi pinguino ñandu come 3 manzanas` | 6 / 33 | `8c823a8bb07954d9e75110f144f1dac8071e0478dfa8132954cfd434934b7210` |

## Referencias

- P-0008 — Credencial basada en passphrase
- P-0009 — Algoritmo de almacenamiento de la frase secreta
