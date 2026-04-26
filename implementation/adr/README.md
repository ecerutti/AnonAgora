# Decisiones de implementación

Esta carpeta contiene los Architecture Decision Records (ADR) de implementación **transversales** del sistema general (`I-XXXX`). Los ADR de implementación específicos de la capa de identidad y de las aplicaciones destino viven en sus respectivas subcarpetas (`implementation/capa_de_identidad/adr/` e `implementation/aplicaciones/<nombre>/adr/`).

Una decisión de implementación responde a "con qué piezas concretas se construye", a diferencia de las decisiones de diseño en `design/adr/` que responden a "qué hace el sistema y por qué".

Ejemplos de decisiones que corresponden aquí (cuando son transversales) o en las subcarpetas correspondientes (cuando son específicas):

- lenguaje de programación y framework de cada componente
- motor de base de datos
- estrategia de despliegue e infraestructura
- bibliotecas concretas para algoritmos ya decididos en el diseño (por ejemplo, qué biblioteca de Argon2id se usa)
- herramientas de observabilidad, logging o monitoreo
- convenciones de desarrollo (testing, CI/CD, gestión de secretos)

Ejemplos de decisiones que **no** corresponden aquí sino en `design/adr/` (o en sus subcarpetas equivalentes):

- qué algoritmo de derivación de claves usa el sistema
- qué mecanismo de autenticación ofrece la aplicación al ciudadano
- qué propiedades de anonimato garantiza el sistema

## Convención de nombres

Los archivos siguen la convención:

`I-XXXX_titulo_descriptivo.md`

Donde `I` indica que la decisión corresponde a implementación y `XXXX` es un número secuencial de cuatro dígitos.

La estructura interna de cada ADR sigue la misma convención definida en `design/adr/README.md`.