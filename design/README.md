# Diseño

Esta carpeta documenta el diseño técnico del sistema general: cómo funciona, qué decisiones se tomaron y por qué.

La estructura interna refleja el modelo definido en P-0021: el sistema se compone de una **capa de identidad** y una o más **aplicaciones destino** construidas sobre ella.

## Organización

- **Documentos transversales** (en la raíz de `design/` y `design/adr/`): modelos y decisiones que aplican al sistema completo o a la relación entre capa y aplicación. Ejemplo: el modelo de amenazas (`threat_model.md`) y los ADRs P-0006, P-0020, P-0021.
- **`design/capa_de_identidad/`**: diseño de la capa de identidad (infraestructura de identidad anónima verificada). Incluye el contrato capa↔aplicación en su `README.md` y el modelo de identidad en `identity_model.md`.
- **`design/aplicaciones/`**: diseño de las aplicaciones destino. Cada aplicación vive en su propia subcarpeta (actualmente solo `participacion_ciudadana/`).

Cada uno de estos ámbitos tiene su propia subcarpeta `adr/` con los Architecture Decision Records correspondientes.

El diseño específico de la versión demo no vive en esta carpeta sino bajo `demo/design/`.

## Cómo encontrar lo que buscás

| Si querés saber... | Mirá en... |
|---|---|
| ¿Cómo funciona el sistema en general? | `design/` (esta carpeta) y sus subcarpetas. |
| ¿Por qué se tomó tal decisión? | El ADR correspondiente en alguna de las subcarpetas `adr/`. |
| ¿Qué garantiza la capa de identidad a las aplicaciones? | `design/capa_de_identidad/README.md` (contrato capa↔aplicación). |
| ¿Cómo funciona la aplicación de participación ciudadana? | `design/aplicaciones/participacion_ciudadana/`. |
| ¿Qué simplificaciones adopta la demo? | `demo/design/` (fuera de esta carpeta). |

La convención de nombres y formato de los ADR vive en `design/adr/README.md`.
