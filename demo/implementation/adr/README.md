# Decisiones de implementación de la demo

Esta carpeta contiene los Architecture Decision Records (ADR) de implementación **transversales** específicos de la demo (`DI-XXXX`). Los ADR de implementación específicos de la capa de identidad y de las aplicaciones destino de la demo viven en sus respectivas subcarpetas (`demo/implementation/capa_de_identidad/adr/` y `demo/implementation/aplicaciones/<nombre>/adr/`).

Una decisión de implementación responde a "con qué piezas concretas se construye", a diferencia de las decisiones de diseño en `demo/design/adr/` que responden a "qué hace la demo y por qué".

La demo puede adoptar piezas concretas distintas a las del sistema general (documentadas en `implementation/adr/`) cuando su alcance conceptual y demostrativo lo justifique. Las simplificaciones adoptadas por la demo no deben trasladarse automáticamente a un despliegue productivo.

Ejemplos de decisiones que corresponden aquí:

- stack tecnológico específico para la demo cuando difiere del del sistema general
- persistencia y despliegue de la demo
- datos de prueba o seed inicial
- mock vs API real para servicios externos en el contexto demo
- autenticación administrativa para gestión técnica de la demo

Ejemplos de decisiones que **no** corresponden aquí:

- decisiones de implementación del sistema general → van a `implementation/adr/`
- decisiones de diseño específicas de la demo → van a `demo/design/adr/`
- decisiones de diseño del sistema general → van a `design/adr/`

## Convención de nombres

Los archivos siguen la convención:

`DI-XXXX_titulo_descriptivo.md`

Donde `D` indica *demo*, `I` indica *implementación*, y `XXXX` es un número secuencial de cuatro dígitos.

La estructura interna de cada ADR sigue la misma convención definida en `design/adr/README.md`.