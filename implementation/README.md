# Implementación

Esta carpeta documenta la implementación técnica del sistema general: con qué piezas concretas se construye lo que el diseño define.

La estructura interna refleja el modelo definido en P-0021: el sistema se compone de una **capa de identidad** y una o más **aplicaciones destino** construidas sobre ella.

## Organización

- **Documentos transversales** (en la raíz de `implementation/` y `implementation/adr/`): documentación e decisiones de implementación que aplican al sistema completo o a la relación entre capa y aplicación.
- **`implementation/capa_de_identidad/`**: implementación de la capa de identidad.
- **`implementation/aplicaciones/`**: implementación de las aplicaciones destino. Cada aplicación vive en su propia subcarpeta (actualmente solo `participacion_ciudadana/`).

Cada uno de estos ámbitos tiene su propia subcarpeta `adr/` con los Architecture Decision Records de implementación correspondientes (prefijo `I-XXXX`).

Las decisiones de diseño sobre qué hace el sistema y por qué viven en `design/` y deben leerse antes de tomar o entender cualquier decisión de implementación.

La implementación específica de la versión demo no vive en esta carpeta sino bajo `demo/implementation/`.

La convención de nombres y formato de los ADR vive en `design/adr/README.md`.