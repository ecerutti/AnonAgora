# AGENTS.md

# AGENTS.md

## Propósito del proyecto

Este proyecto explora el diseño de una **Plataforma de Participación Ciudadana con Identidad Verificada y Anonimato Persistente**.

El objetivo del sistema es permitir que ciudadanos reales participen en propuestas y debates públicos sin exponer su identidad real ni crear perfiles públicos rastreables.

La plataforma funciona como un **termómetro social**, no como un sistema de decisión política vinculante.

Este repositorio contiene:

- la propuesta conceptual del sistema
- decisiones de diseño de la plataforma
- una futura implementación demo

## Sobre el nombre del proyecto

El proyecto todavía no tiene un nombre definitivo. El repositorio se llama `AnonAgora` y el dominio del despliegue público es `anonagora.cloud`, pero ambos son circunstanciales: surgen de la necesidad de tener un identificador para alojar el repositorio en GitHub y de contar con un dominio para la futura demo.

Dentro de la documentación del proyecto debe usarse terminología genérica ("la plataforma", "el sistema", "el proyecto"). El nombre `AnonAgora` solo debe aparecer cuando sea inevitable hacer referencia al repositorio o al dominio como tales — principalmente en `docs/index.md`, que es la página de recepción del sitio público.

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
| `notas/` | Material de trabajo del proyecto: estado del trabajo en curso, recordatorios, ideas pendientes de evaluación, borradores conceptuales que todavía no son documentos oficiales. Mantenido por el humano con apoyo de los agentes. | Decisiones ya tomadas (van a `design/adr/`), descripciones de componentes (van a `design/`), scratchpads del agente durante una conversación. |

La convención de nombres, estructura y formato de los ADR está definida en `design/adr/README.md`, incluyendo el criterio para determinar cuándo una decisión requiere un ADR y cuándo corresponde un documento de diseño.

---

# Reglas de escritura

El repositorio debe mantenerse limpio.

Solo deben guardarse en el repositorio:

- documentación del proyecto
- decisiones de diseño
- ADR
- código fuente

No deben guardarse en el repositorio los archivos de trabajo internos que el agente genera para sí mismo durante una conversación:

- planes de trabajo propios
- notas de razonamiento
- scratchpads
- logs
- documentos temporales de análisis

Estos elementos deben permanecer en el workspace interno del agente y no pertenecen al repo.

Esto es distinto del material de trabajo del proyecto, que vive en `notas/` y sí forma parte del repo. La diferencia es quién lo mantiene y para qué sirve: `notas/` contiene material de gestión del proyecto mantenido por el humano (con apoyo del agente cuando corresponda), cuyo valor trasciende cualquier conversación individual. Los scratchpads del agente, en cambio, son insumo efímero de una sola conversación.

---

# Comportamiento esperado del agente

Antes de modificar el repositorio, el agente debe:

1. leer `notas/estado_del_trabajo.md` para conocer el estado actual del trabajo, las decisiones pendientes y los recordatorios activos 
2. leer la documentación relevante del repositorio
3. identificar decisiones de diseño existentes
4. evitar duplicar decisiones ya registradas
5. proponer nuevos ADR cuando corresponda
6. distinguir claramente entre diseño conceptual y simplificaciones de la demo
7. mantener cambios mínimos, coherentes y trazables

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
