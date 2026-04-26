# AGENTS.md

## Propósito del proyecto

Este proyecto explora el diseño de un sistema con dos componentes:

- una **capa de identidad anónima verificada** reutilizable, que se encarga de verificar que cada participante es una persona real y de emitir identidades anónimas persistentes; y
- una **aplicación destino** construida sobre esa capa, que define qué pueden hacer los ciudadanos con sus identidades anónimas.

La modularidad y el modelo "una capa de identidad por aplicación destino" están definidos en P-0021.

La primera aplicación destino diseñada es una **plataforma de participación ciudadana**: permite a ciudadanos reales participar en propuestas y debates públicos sin exponer su identidad real ni crear perfiles públicos rastreables, y funciona como un **termómetro social**, no como un sistema de decisión política vinculante. El modelo permite construir otras aplicaciones destino sobre la misma capa (por ejemplo, un sistema de denuncias anónimas), cada una como despliegue independiente.

Este repositorio contiene:

- la propuesta conceptual de la aplicación de participación ciudadana
- decisiones de diseño de la capa de identidad y de la aplicación
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
3. aplicación destino  

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

El repositorio se organiza en dos árboles paralelos: el del **sistema general** (en la raíz) y el de la **demo** (bajo `demo/`). Cada uno tiene la misma estructura interna: documentación conceptual, diseño, implementación y código fuente. La demo hereda íntegramente las decisiones del sistema general; los documentos bajo `demo/` solo registran las simplificaciones, omisiones o sustituciones específicas de la demo. Esta regla de herencia se documenta en `demo/README.md`.

Dentro de `design/` y `implementation/` (tanto en el sistema general como en la demo), la estructura refleja el modelo de P-0021: una **capa de identidad** y una o más **aplicaciones destino**. Para cada uno, las decisiones se registran en ADRs (en una subcarpeta `adr/`) y los documentos descriptivos viven directamente en la carpeta correspondiente. Las decisiones y documentos **transversales** —los que aplican al sistema completo o a la relación entre capa y aplicación— viven en la raíz de `design/` y `design/adr/` (o sus equivalentes en `implementation/`).

## Árboles principales

| Carpeta | Contenido | Qué NO va acá |
|---|---|---|
| `docs/propuesta/` | Documentación conceptual de la aplicación de participación ciudadana, orientada a lectores no técnicos (responsables institucionales, personas interesadas en comprender el objetivo del proyecto). | Decisiones técnicas, especificaciones de implementación. |
| `docs/` | Documentación técnica del desarrollo: arquitectura general, overviews, especificaciones, material de apoyo para el desarrollo. | Decisiones de diseño con alternativas evaluadas. |
| `design/` | Documentos de diseño del sistema general que no requieren ADR. La estructura interna se describe abajo. | ADRs, decisiones entre alternativas, decisiones de implementación, material específico de la demo. |
| `implementation/` | Documentos de implementación del sistema general que no requieren ADR. La estructura interna replica la de `design/`. | ADRs, decisiones de implementación entre alternativas, diseño conceptual, material específico de la demo. |
| `demo/` | Todo lo referente a la versión demo: documentación, decisiones de diseño, decisiones de implementación y código fuente. La estructura interna replica la del sistema general (`demo/design/`, `demo/implementation/` y eventualmente `demo/src/`). La demo hereda las decisiones del sistema general; aquí solo se registran las simplificaciones específicas. | Decisiones del sistema general, documentación conceptual general. |
| `notas/` | Material de trabajo del proyecto: estado del trabajo en curso, recordatorios, ideas pendientes de evaluación, borradores conceptuales que todavía no son documentos oficiales. Mantenido por el humano con apoyo de los agentes. | Decisiones ya tomadas (van a las carpetas de ADR correspondientes), descripciones de componentes (van a `design/` o `implementation/`), scratchpads del agente durante una conversación. |

## Estructura interna de `design/` (y, simétricamente, de `implementation/` y de `demo/design/` y `demo/implementation/`)

| Carpeta | Contenido | Prefijo de ADRs |
|---|---|---|
| `design/` (raíz) | Documentos de diseño transversales al sistema completo: modelos conceptuales y especificaciones que aplican a la capa y a las aplicaciones destino por igual (ej.: `threat_model.md`). | — |
| `design/adr/` | ADRs transversales: decisiones sobre el sistema completo o sobre la relación entre capa y aplicación (ej.: P-0006, P-0020, P-0021). | `P-XXXX` |
| `design/capa_de_identidad/` | Documentos de diseño de la capa de identidad (ej.: `identity_model.md`, contrato capa↔aplicación en su `README.md`). | — |
| `design/capa_de_identidad/adr/` | ADRs de diseño específicos de la capa de identidad. | `P-XXXX` |
| `design/aplicaciones/<nombre>/` | Documentos de diseño de una aplicación destino concreta (actualmente solo `participacion_ciudadana/`). | — |
| `design/aplicaciones/<nombre>/adr/` | ADRs de diseño específicos de esa aplicación destino. | `P-XXXX` |

`implementation/` replica la misma estructura. Los ADRs de implementación del sistema general usan prefijo `I-XXXX`.

`demo/design/` y `demo/implementation/` replican la misma estructura interna que sus contrapartes del sistema general. Los ADRs de diseño específicos de la demo usan prefijo `DP-XXXX` y los de implementación de la demo usan prefijo `DI-XXXX`.

## Notas sobre la organización

- El **prefijo del ADR** (`P`, `I`, `DP`, `DI`) indica si es de diseño o implementación, y si aplica al sistema general o solo a la demo. La **carpeta** indica si pertenece a la capa, a una aplicación destino o es transversal. Ambos ejes son independientes (P-0021).
- Los identificadores de ADRs son únicos y secuenciales **dentro de cada prefijo**, sin repartirse por carpeta. Un `P-XXXX` puede vivir en `design/adr/`, en `design/capa_de_identidad/adr/` o en `design/aplicaciones/<nombre>/adr/` según corresponda, pero su número no se reutiliza.
- Cuando se diseñe una nueva aplicación destino, se crea una subcarpeta hermana de `participacion_ciudadana/` bajo `design/aplicaciones/` (y simétricamente en `implementation/` y en las contrapartes de la demo).

La convención de nombres, estructura y formato de los ADR está definida en `design/adr/README.md`. El criterio aplica a los cuatro tipos de ADR del proyecto: `P-XXXX` (diseño del sistema general), `I-XXXX` (implementación del sistema general), `DP-XXXX` (diseño específico de la demo) y `DI-XXXX` (implementación específica de la demo).

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
