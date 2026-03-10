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

Los números deben asignarse en orden incremental.

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

Los ADR deben utilizarse para documentar decisiones que afecten aspectos importantes del diseño de la plataforma, por ejemplo:

- arquitectura del sistema
- modelo de identidad anónima
- mecanismos de seguridad o privacidad
- convenciones estructurales del sistema
- decisiones que podrían ser cuestionadas o revisadas en el futuro

No es necesario utilizar ADR para decisiones menores de implementación o detalles de interfaz.

# Evolución de las decisiones

En el futuro una decisión puede ser revisada o reemplazada por otra.

Cuando esto ocurra:

- se debe crear un **nuevo ADR**
- el ADR anterior no se elimina
- el nuevo ADR debe mencionar cuál decisión reemplaza

Esto permite mantener un historial claro de la evolución del diseño del sistema.
