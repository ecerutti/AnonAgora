# Demo

Esta carpeta contiene todo lo referente a la implementación demo del sistema: documentación, decisiones de diseño, decisiones de implementación y código fuente.

## Herencia respecto del sistema general

La demo hereda íntegramente las decisiones de diseño e implementación del sistema general (`design/` e `implementation/` en la raíz del repositorio). Los documentos y ADRs bajo `demo/design/` y `demo/implementation/` solo registran las decisiones específicas de la demo: simplificaciones, omisiones o sustituciones que se adoptan exclusivamente en este contexto.

Cuando un documento de la demo no contradice una decisión del sistema general, esa decisión se hereda íntegramente. Las simplificaciones de la demo no deben trasladarse automáticamente a un despliegue productivo.

## Estructura

- `demo/design/` — diseño específico de la demo. Replica la estructura interna de `design/` raíz: ADRs transversales (`demo/design/adr/`), capa de identidad (`demo/design/capa_de_identidad/`) y aplicaciones destino (`demo/design/aplicaciones/<nombre>/`). Prefijo de ADRs: `DP-XXXX`.
- `demo/implementation/` — implementación específica de la demo. Replica la estructura interna de `implementation/` raíz. Prefijo de ADRs: `DI-XXXX`.
- Código fuente de la demo (cuando exista): vivirá en una subcarpeta dedicada de esta misma raíz.

La convención general de ADRs del proyecto vive en `design/adr/README.md`.