# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Todas las pendientes son consecuencia del checkpoint realizado sobre el repositorio. Deben abordarse en conversaciones dedicadas en este orden:

1. **Refactorización por arquitectura modular (Gap 5 en curso).** El ADR P-0021 está cerrado y formaliza la estructura del sistema en dos capas: capa de identidad (infraestructura de identidad anónima verificada) y aplicación destino. Está pendiente la ejecución del refactor que aplica esa decisión al repositorio:
   - Reestructuración física: crear carpetas `design/capa_de_identidad/`, `design/aplicaciones/participacion_ciudadana/`, simétricas en `implementation/` y en las secciones de demo. Mover los ADRs y documentos descriptivos según el mapeo cerrado.
   - Refactor de vocabulario en ADRs cerrados: aplicar la regla de "vocabulario y marco sí, decisiones no" para distinguir capa, aplicación y sistema. Casos especiales: el principio estructural de P-0005 ("no sugerir memoria entre sesiones") se absorbe en `identity_model.md`; P-0020 se clarifica por capa.
   - Refactor de documentos descriptivos: `AGENTS.md` (Propósito y tabla de estructura del repo), `docs/architecture_overview.md` (reescritura), `design/README.md`, `design/adr/README.md`, READMEs de las carpetas nuevas. Documentar el contrato capa↔aplicación en `design/capa_de_identidad/README.md`.
   - Ajustes menores en `docs/propuesta/` para enmarcar la propuesta como descripción de una aplicación destino específica.
   - Cierre: grep de vocabulario residual, validación de coherencia.

2. **P-0022 — Comportamiento ante fallos de servicios externos.** ADR que defina el comportamiento del sistema ante fallos o indisponibilidad de servicios externos: verificador de identidad caído, token vencido en tránsito, componente de proving ZK con demoras o error, API del revisor de lenguaje caída o con error. Debe cubrir principios generales de degradación, reintentos y comunicación al ciudadano.

3. **P-0023 — Moderación de contenido y retiro de propuestas.** ADR que formalice la ausencia de moderación de contenido más allá del revisor de lenguaje (P-0011), y que defina cómo se maneja el caso de propuestas que deben retirarse por orden judicial, contenido ilegal, o solicitudes de ciudadanos con implicancias legales (derecho al olvido). Debe dejar explícitas las limitaciones derivadas del diseño actual y las implicancias legales para el operador que despliega el sistema.

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma. Incluye una decisión pendiente surgida durante el diseño de P-0014:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Manejo de la identidad anónima en la demo (simulación de la verificación de unicidad). La demo no usa AUTENTICAR real ni implementa ZK; ambas omisiones deben quedar documentadas en un ADR de demo (DP-XXXX) que explique por qué no aplican en ese contexto.
6. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
7. Autenticación administrativa para gestión técnica de la demo.
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **P-0021 — Arquitectura modular: capa de identidad y aplicaciones destino.** Cierra Gap 5 a nivel decisión. Reconoce que el sistema está compuesto por una capa de identidad reutilizable y una aplicación destino, con el modelo "una capa ↔ una aplicación por despliegue". La aplicación de participación ciudadana queda como una aplicación destino entre otras posibles. Las consecuencias estructurales (refactor del repositorio, refactor de vocabulario en ADRs y documentos) están pendientes de ejecución (ver decisiones pendientes).

## Mapeo del refactor en curso (P-0021)

Esta sección es insumo de la refactorización derivada de P-0021. Una vez completada la reestructuración física (Bloque 4 del trabajo de Gap 5), esta sección se elimina porque la información queda reflejada en la estructura de carpetas del repositorio.

### Estructura de carpetas resultante

```
design/
  README.md
  adr/                                      ADRs transversales
    README.md
    P-0006_*.md
    P-0020_*.md
    P-0021_*.md
  threat_model.md                           Documento descriptivo transversal
  capa_de_identidad/
    README.md
    identity_model.md
    adr/
      P-0002_*.md, P-0003_*.md, P-0007_*.md,
      P-0013_*.md, P-0014_*.md, P-0015_*.md
  aplicaciones/
    participacion_ciudadana/
      README.md
      vinculacion_de_propuestas.md
      adr/
        P-0001_*.md, P-0004_*.md, P-0005_*.md, P-0008_*.md,
        P-0009_*.md, P-0010_*.md, P-0011_*.md, P-0012_*.md,
        P-0016_*.md, P-0017_*.md, P-0018_*.md, P-0019_*.md
  demo/
    capa_de_identidad/
      adr/
        DP-0001_*.md
    aplicaciones/
      participacion_ciudadana/
        adr/
          (vacío hasta que existan ADRs DP de la aplicación)
```

La estructura de `implementation/` e `implementation/demo/` es simétrica a la de `design/` y `design/demo/`. Hoy esas carpetas no tienen ADRs ni documentos descriptivos, por lo que el trabajo del Bloque 4 sobre ellas se limita a crear las carpetas vacías con sus README según corresponda.

### Mapeo de ADRs por capa

| ADR     | Capa                              |
|---------|-----------------------------------|
| P-0001  | Aplicación participación ciudadana |
| P-0002  | Capa de identidad                 |
| P-0003  | Capa de identidad                 |
| P-0004  | Aplicación participación ciudadana |
| P-0005  | Aplicación participación ciudadana |
| P-0006  | Transversal                       |
| P-0007  | Capa de identidad                 |
| P-0008  | Aplicación participación ciudadana |
| P-0009  | Aplicación participación ciudadana |
| P-0010  | Aplicación participación ciudadana |
| P-0011  | Aplicación participación ciudadana |
| P-0012  | Aplicación participación ciudadana |
| P-0013  | Capa de identidad                 |
| P-0014  | Capa de identidad                 |
| P-0015  | Capa de identidad                 |
| P-0016  | Aplicación participación ciudadana |
| P-0017  | Aplicación participación ciudadana |
| P-0018  | Aplicación participación ciudadana |
| P-0019  | Aplicación participación ciudadana |
| P-0020  | Transversal                       |
| P-0021  | Transversal                       |
| DP-0001 | Capa de identidad (demo)          |

### Mapeo de documentos descriptivos

| Documento                          | Ubicación destino                         |
|------------------------------------|-------------------------------------------|
| `design/threat_model.md`           | Permanece en `design/` (transversal)      |
| `design/identity_model.md`         | Mueve a `design/capa_de_identidad/`       |
| `design/vinculacion_de_propuestas.md` | Mueve a `design/aplicaciones/participacion_ciudadana/` |

### Notas sobre el alcance del Bloque 4

- El Bloque 4 incluye únicamente movimientos físicos: crear carpetas, mover archivos, sin renombrar (Camino A: identificadores `P-XXXX` se conservan).
- Cada carpeta nueva se crea con un `README.md` mínimo en este bloque para evitar dejar el repositorio en estado intermedio. El contenido definitivo de cada README es responsabilidad del Bloque 6 (refactor de documentos descriptivos).
- El refactor de vocabulario dentro de los ADRs es tarea del Bloque 5, no del 4.
- Los documentos descriptivos transversales o que vivan en otras capas requerirán refactor de vocabulario en el Bloque 5 o 6 según corresponda; el Bloque 4 solo los reubica.

### Casos especiales del refactor de vocabulario (referencia para Bloque 5)

- **P-0005:** el principio estructural "no sugerir memoria entre sesiones" se absorbe en `identity_model.md` como propiedad del modelo de identidad (no decisión con alternativas). El ADR queda como decisión específica de la aplicación de participación ciudadana sobre su política concreta de sesiones.
- **P-0020:** el ADR clarifica qué reglas son universales (válidas para cualquier componente de cualquier despliegue) y cuáles son específicas de los componentes actuales (emisor de la capa, plataforma de la aplicación participativa). La tabla de eventos incluye los eventos actuales y queda implícito que toda aplicación destino futura agrega sus propios eventos siguiendo los criterios generales.
- **Contrato capa↔aplicación:** se documenta en `design/capa_de_identidad/README.md` (Bloque 6). La capa entrega `{pseudónimo, anon_id, prueba ZK}` al momento del registro; la aplicación destino es responsable de cualquier credencial recurrente que use para autenticar al ciudadano en visitas posteriores.
