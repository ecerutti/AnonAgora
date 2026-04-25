# Decisiones de implementación de la demo

Esta carpeta contiene los Architecture Decision Records (ADR) de implementación específicos de la demo.

Una decisión de implementación responde a "con qué piezas concretas se construye", a diferencia de las decisiones de diseño en `design/demo/adr/` que responden a "qué hace la demo y por qué".

La demo puede adoptar piezas concretas distintas a las de la plataforma productiva (documentadas en `implementation/adr/`) cuando su alcance conceptual y demostrativo lo justifique. Las simplificaciones adoptadas por la demo no deben trasladarse automáticamente a un despliegue productivo.

Ejemplos de decisiones que corresponden aquí:

- stack tecnológico específico para la demo cuando difiere del de la plataforma productiva
- persistencia y despliegue de la demo
- datos de prueba o seed inicial
- mock vs API real para servicios externos en el contexto demo
- autenticación administrativa para gestión técnica de la demo

Ejemplos de decisiones que **no** corresponden aquí:

- decisiones de implementación de la plataforma productiva → van a `implementation/adr/`
- decisiones de diseño específicas de la demo → van a `design/demo/adr/`
- decisiones de diseño de la plataforma general → van a `design/adr/`

## Convención de nombres

Los archivos siguen la convención:

`DI-XXXX_titulo_descriptivo.md`

Donde `D` indica *demo*, `I` indica *implementación*, y `XXXX` es un número secuencial de cuatro dígitos.

La estructura interna de cada ADR sigue la misma convención definida en `design/adr/README.md`.
