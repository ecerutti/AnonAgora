# Implementación de la demo

Esta carpeta contiene la implementación técnica específica de la demo.

Las decisiones de implementación específicas de la demo se documentan en los ADRs `DI-XXXX` distribuidos en esta carpeta y sus subcarpetas (transversales en `demo/implementation/adr/`, específicos de capa en `demo/implementation/capa_de_identidad/adr/`, específicos de aplicación en `demo/implementation/aplicaciones/<nombre>/adr/`).

Las decisiones sobre qué hace la demo y por qué viven en `demo/design/` y deben leerse antes de tomar o entender cualquier decisión de implementación de la demo.

La implementación del sistema general (fuera del contexto demo) vive en `implementation/`. La demo hereda íntegramente esas decisiones excepto donde las contradiga explícitamente; las simplificaciones de la demo no deben trasladarse automáticamente a un despliegue productivo. Ver `demo/README.md` para la regla de herencia completa.