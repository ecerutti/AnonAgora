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

| Carpeta | Contenido | Qué NO va acá |
|---|---|---|
| `docs/propuesta/` | Documentación conceptual de la idea, orientada a lectores no técnicos (responsables institucionales, personas interesadas en comprender el objetivo del proyecto). | Decisiones técnicas, especificaciones de implementación. |
| `docs/` | Documentación técnica del desarrollo: arquitectura general, overviews, especificaciones, material de apoyo para el desarrollo. | Decisiones de diseño con alternativas evaluadas. |
| `design/` | Documentos de diseño de la plataforma que no requieren ADR: modelos conceptuales, glosarios, especificaciones que no surgen de elegir entre alternativas. | ADRs, decisiones entre alternativas. |
| `design/adr/` | ADRs de la plataforma general, con prefijo `P-XXXX`. | Documentos descriptivos sin decisión entre alternativas, ADRs específicos de la demo. |
| `design/demo/` | Documentos de diseño específicos de la versión demo. La demo puede utilizar soluciones simplificadas que no representen la arquitectura final del sistema. | Decisiones de la plataforma general. |
| `design/demo/adr/` | ADRs específicos de la demo, con prefijo `D-XXXX`. | ADRs de la plataforma general. |
| `demo/` | Código fuente de la implementación demo. | Documentación. |

La convención de nombres, estructura y formato de los ADR está definida en `design/adr/README.md`, incluyendo el criterio para determinar cuándo una decisión requiere un ADR y cuándo corresponde un documento de diseño.

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
