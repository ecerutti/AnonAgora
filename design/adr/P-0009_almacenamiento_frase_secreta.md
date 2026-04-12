# P-0009 — Algoritmo de almacenamiento de la frase secreta

## Contexto

La frase secreta definida por el ciudadano durante el registro es el único
mecanismo de acceso y recuperación de su identidad anónima. Por esta razón,
su almacenamiento requiere protección especial.

Almacenar la frase en texto plano es inaceptable: cualquier acceso no
autorizado a la base de datos expondría todas las frases secretas del sistema.
Es necesario elegir un algoritmo de derivación de claves que haga
computacionalmente inviable recuperar la frase original a partir del valor
almacenado, incluso para un atacante con acceso directo a la base de datos.

## Opciones consideradas

### Opción 1 — bcrypt

Algoritmo ampliamente adoptado, diseñado específicamente para el hash de
contraseñas. Incorpora salt automático y un factor de costo ajustable.

**Ventajas**

- Muy maduro y ampliamente soportado en todas las plataformas y lenguajes.
- Probado en producción durante décadas.
- Salt automático integrado.

**Desventajas**

- Limita la entrada a 72 bytes, lo que puede truncar passphrases largas sin
  advertir al usuario.
- Diseñado para hardware de hace décadas; los ataques con GPU modernos son
  más eficientes contra bcrypt que contra algoritmos más recientes.
- No permite configurar uso de memoria, lo que lo hace más vulnerable a
  ataques con hardware especializado (ASICs).

### Opción 2 — PBKDF2

Estándar recomendado por NIST, basado en aplicar repetidamente una función
hash (generalmente SHA-256 o SHA-512).

**Ventajas**

- Estándar reconocido por NIST y requerido en algunos contextos regulatorios.
- Amplio soporte en bibliotecas estándar.
- Sin límite práctico de longitud de entrada.

**Desventajas**

- Más eficiente en GPU que bcrypt o Argon2, lo que facilita ataques de fuerza
  bruta con hardware moderno.
- No tiene componente de uso de memoria, haciéndolo más vulnerable a hardware
  especializado.

### Opción 3 — Argon2id

Ganador del Password Hashing Competition (2015). Combina resistencia a
ataques de canal lateral (variante Argon2i) con resistencia a ataques de
fuerza bruta con GPU y hardware especializado (variante Argon2d).

**Ventajas**

- Recomendación actual de OWASP para almacenamiento de contraseñas.
- Costoso en CPU y en memoria de forma configurable, dificultando ataques
  con GPU y ASICs.
- Sin límite de longitud de entrada.
- Salt único por registro generado automáticamente.
- Parámetros de costo (memoria, iteraciones, paralelismo) configurables y
  ajustables a futuro sin romper hashes existentes.

**Desventajas**

- Más reciente que bcrypt o PBKDF2, aunque ya cuenta con soporte maduro
  en las principales bibliotecas de los lenguajes más usados.

## Decisión

Se adopta la **Opción 3 — Argon2id**.

Cada frase secreta se almacena como un hash Argon2id con salt único generado
aleatoriamente por registro. La frase en texto plano se descarta
inmediatamente después de calcular el hash y nunca se persiste.

Los parámetros de costo (memoria, iteraciones, paralelismo) son configurables
por el operador y deben revisarse periódicamente a medida que el hardware
disponible evolucione.

## Justificación

Argon2id es la recomendación actual de OWASP para almacenamiento de
contraseñas y el resultado del proceso de selección más riguroso realizado
por la comunidad de seguridad (PHC). Su diseño híbrido lo hace resistente
tanto a ataques de canal lateral como a ataques de fuerza bruta con hardware
especializado, que son los vectores más relevantes en un sistema donde el
anonimato del ciudadano depende de que su frase secreta no pueda recuperarse
aunque un atacante obtenga acceso a la base de datos.

## Consecuencias

- La base de datos almacena únicamente el hash Argon2id y el salt, nunca la
  frase en texto plano.
- Los parámetros de costo son configurables y deben documentarse en la guía
  de instalación y operación de la plataforma.
- La verificación de la frase en el ingreso consiste en recalcular el hash
  con el salt almacenado y comparar, sin necesidad de conocer la frase
  original.
