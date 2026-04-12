# Architecture Decision Records (ADR)

Esta carpeta contiene los **Architecture Decision Records (ADR)** del proyecto.

Un ADR es un documento breve que registra **una decisión de diseño importante** tomada durante el desarrollo de la arquitectura de la plataforma.

El objetivo de estos documentos es preservar el razonamiento detrás de cada decisión para que pueda ser comprendido en el futuro, incluso si las personas que tomaron la decisión original ya no participan del proyecto.

Los ADR permiten responder preguntas como:

- ¿Por qué el sistema funciona de esta manera?
- ¿Qué alternativas se evaluaron?
- ¿Qué problemas se intentaban resolver?
- ¿Cuáles son las consecuencias de esta decisión?

Cada ADR describe **una sola decisión**.

# Convención de nombres

Los archivos ADR siguen la siguiente convención:

```
P-XXXX_titulo_descriptivo.md
```

Ejemplos:

```
P-0001_identidad_anonima_visible_o_no.md
P-0002_pseudonimos_anonimos_amigables.md
P-0003_seleccion_de_pseudonimo_anonimo.md
```

Donde:

- `P` indica que la decisión corresponde al **diseño de la plataforma**.
- `XXXX` es un número secuencial de cuatro dígitos.
- El resto del nombre es una descripción corta del tema de la decisión.

Los números deben asignarse en orden incremental y no deben reordenarse a posteriori.

# Estructura de un ADR

Cada ADR debe seguir la siguiente estructura.

## Título

La primera línea del documento debe contener el identificador y una descripción breve de la decisión.

Ejemplo:

```
# P-0003 — Selección del pseudónimo de identidad anónima
```

## Contexto

Describe el problema o situación que motivó la decisión.

Debe explicar:

- qué parte del sistema está involucrada
- qué preguntas surgieron durante el diseño
- qué aspectos del sistema se ven afectados

Este apartado debe proporcionar suficiente contexto para que alguien que no participó en la discusión original pueda entender el problema.

## Opciones consideradas

Describe las alternativas evaluadas antes de tomar la decisión.

Cada opción debe presentarse con:

- un título
- una breve descripción
- una lista de ventajas
- una lista de desventajas

Ejemplo:

```

### Opción 1 — Identificador técnico

En esta opción, la identidad anónima se mostraría mediante un identificador técnico o pseudónimo numérico, por ejemplo:

Anon#1729
User-8F3A91
ID-A17K92

Ventajas

- Implementación simple
- Garantiza unicidad

Desventajas

- Difícil de recordar
- Poco amigable para los usuarios
```

## Decisión

Describe claramente qué opción fue elegida.

Debe indicar:

- qué solución se adopta
- cómo se aplicará en el sistema

Este apartado debe ser explícito y no ambiguo.

## Justificación

Explica **por qué** se eligió esta opción y no las otras.

Puede incluir consideraciones como:

- experiencia de usuario
- seguridad o privacidad
- simplicidad de implementación
- coherencia con los principios del sistema

Este apartado registra el razonamiento detrás de la decisión.

## Consecuencias

Describe los efectos de la decisión adoptada.

Puede incluir:

- cambios en el diseño del sistema
- limitaciones introducidas
- implicaciones futuras

Este apartado ayuda a comprender qué impacto tiene la decisión en el sistema.

# Reglas de formato

Para mantener consistencia entre los ADR del proyecto:

- Los documentos deben escribirse en **Markdown**.
- No utilizar separadores horizontales (`---`).
- Cada ADR debe documentar **una sola decisión**.
- El lenguaje debe ser **claro y explicativo**.
- Cuando sea posible, incluir **ejemplos concretos**.

# Alcance

## Cuándo corresponde un ADR

No toda decisión de diseño requiere un ADR.

Un ADR es el registro de una decisión **que pudo haber sido distinta**. Es decir, una decisión donde existían varias opciones razonables y se eligió una en base a criterios explícitos.

Un ADR responde preguntas como:

- ¿Por qué se eligió esto y no aquello?
- ¿Qué alternativas se evaluaron?
- ¿Qué consecuencias tiene la decisión?

Las decisiones de diseño que no cumplen esta condición —por ejemplo, la descripción de un modelo conceptual, un glosario, o una especificación que no surge de elegir entre alternativas— **no requieren ADR** y se documentan directamente en `design/` como documentos de diseño.

### Criterio práctico

Una decisión merece un ADR si cumple **al menos una** de las siguientes condiciones:

**A. Hay alternativas reales.**
Existían varias opciones razonables y se eligió una.
Ejemplo: ¿el pseudónimo lo elige el usuario o lo genera el sistema? Tres opciones evaluadas, una elegida. Eso es un ADR (ver `P-0003`).

**B. La decisión puede ser cuestionada en el futuro.**
Alguien razonable podría proponer lo contrario más adelante, y conviene dejar registrado por qué no.
Ejemplo: las propuestas publicadas son inmutables. Un desarrollador futuro podría preguntar *"¿y si permitiéramos editarlas?"*. El ADR explica por qué no.

**C. La decisión afecta la arquitectura de forma profunda.**
Cambiarla más adelante tendría impacto estructural sobre el sistema.
Ejemplo: dónde vive la identidad anónima (cliente, servidor, híbrido).

**D. Hay trade-offs importantes.**
La decisión tiene ventajas y desventajas claras y fue tomada deliberadamente aceptando las contras.
Ejemplo: no existe reputación pública de usuarios (ver `P-0001`).

Si una decisión de diseño no cumple ninguna de estas condiciones, probablemente no necesita ADR y basta con documentarla como parte del diseño general en `design/`.

### Regla rápida

Si el documento puede escribirse sin una sección "Opciones consideradas", probablemente sea un documento de diseño, no un ADR.

Si no puede escribirse sin explicar qué alternativas se evaluaron y por qué se eligió una, es un ADR.

# Evolución de las decisiones

En el futuro una decisión puede ser revisada o reemplazada por otra.

Cuando esto ocurra:

- se debe crear un **nuevo ADR**
- el ADR anterior no se elimina
- el nuevo ADR debe mencionar cuál decisión reemplaza

Esto permite mantener un historial claro de la evolución del diseño del sistema.
