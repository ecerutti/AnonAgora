# AGENTS.md

## Propósito del proyecto

AnonAgora explora el diseño de una **Plataforma de Participación Ciudadana con Identidad Verificada y Anonimato Persistente**.

El objetivo del sistema es permitir que ciudadanos reales participen en propuestas y debates públicos sin exponer su identidad real ni crear perfiles públicos rastreables.

La plataforma funciona como un **termómetro social**, no como un sistema de decisión política vinculante.

Este repositorio contiene:

- la propuesta conceptual del sistema
- decisiones de diseño de la plataforma
- una futura implementación demo

---

# Fuente de verdad

Este repositorio es la **fuente de verdad del proyecto**.

Antes de realizar cualquier tarea, el agente debe reconstruir el contexto leyendo la documentación existente en el repositorio.

Las conversaciones no deben considerarse fuente de verdad permanente.

---

# Principios fundamentales del sistema

Las decisiones técnicas deben respetar los siguientes principios.

## Identidad verificada

Cada participante debe corresponder a una persona real.

El sistema debe evitar identidades múltiples.

## Anonimato persistente

Después de la verificación inicial, la identidad real del ciudadano no debe participar en el funcionamiento cotidiano del sistema.

Las acciones dentro de la plataforma no deben poder vincularse con la identidad real.

## Separación de funciones

El sistema debe mantener separadas tres funciones:

1. verificación de identidad  
2. emisión de identidad anónima  
3. plataforma participativa  

## Ausencia de reputación pública

La plataforma evita construir identidades públicas persistentes.

Las propuestas deben evaluarse por su contenido y no por la identidad del autor.

## Minimización de correlaciones

El diseño debe minimizar:

- correlaciones temporales
- metadatos sensibles
- identificadores persistentes visibles

---

# Estructura del repositorio

## Documentación de la propuesta

```
docs/propuesta/
```

Contiene la documentación que describe la idea y la lógica conceptual del sistema.

Estos documentos están orientados principalmente a explicar la propuesta a lectores no técnicos, como responsables institucionales o personas interesadas en comprender el objetivo del proyecto.

---

## Documentación del desarrollo

```
docs/
```

Contiene documentación relacionada con el desarrollo técnico del sistema.

Aquí pueden encontrarse documentos de arquitectura, especificaciones técnicas, análisis de implementación y material de apoyo para el desarrollo de la plataforma.

---

## Diseño del sistema

```
design/
design/adr/
```

Contiene decisiones de diseño que aplican a la plataforma general.

Las decisiones arquitectónicas importantes deben registrarse mediante **Architecture Decision Records (ADR)**.

La convención de nombres, estructura y formato de los ADR está definida en:

```
design/adr/README.md
```

---

## Diseño específico de la demo

```
design/demo/
design/demo/adr/
```

Contiene decisiones exclusivas de la versión demo.

La demo puede utilizar soluciones simplificadas que no representen la arquitectura final del sistema.

---

## Código de la demo

```
demo/
```

Contiene únicamente el código fuente de la implementación demo.

---

# Decisiones de arquitectura

Las decisiones de arquitectura deben registrarse mediante
Architecture Decision Records (ADR).

La convención de nombres, estructura y formato de los ADR
está definida en:

design/adr/README.md

---

# Reglas de escritura

El repositorio debe mantenerse limpio.

Solo deben guardarse en el repositorio:

- documentación del proyecto
- decisiones de diseño
- ADR
- código fuente

No deben guardarse archivos de trabajo internos del agente, incluyendo:

- planes de trabajo
- notas internas
- scratchpads
- logs
- documentos de razonamiento
- archivos temporales
- documentación generada únicamente para ayudar al agente a pensar

Estos elementos deben permanecer en el workspace interno del agente.

---

# Comportamiento esperado del agente

Antes de modificar el repositorio, el agente debe:

1. leer la documentación relevante del repositorio
2. identificar decisiones de diseño existentes
3. evitar duplicar decisiones ya registradas
4. proponer nuevos ADR cuando corresponda
5. distinguir claramente entre diseño conceptual y simplificaciones de la demo
6. mantener cambios mínimos, coherentes y trazables

---

# Prioridad de interpretación

En caso de contradicción entre documentos:

1. Los ADR prevalecen sobre otros documentos.
2. Las decisiones más recientes prevalecen sobre las anteriores.
3. La documentación conceptual en `docs/propuesta/` describe la intención del sistema, pero las decisiones técnicas se definen en `design/`.

---

# Promesas de anonimato

La documentación pública del proyecto no debe prometer anonimato absoluto.

El sistema está diseñado para minimizar la posibilidad de vincular identidad real con actividad dentro de la plataforma, pero debe evitar afirmaciones del tipo:

- "nadie puede saber"
- "imposible identificar"
- "anonimato total garantizado"
