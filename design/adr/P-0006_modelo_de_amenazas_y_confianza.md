# P-0006 — Modelo de amenazas y supuestos de confianza

## Contexto

El sistema busca permitir participación ciudadana digital bajo identidades anónimas persistentes, manteniendo al mismo tiempo la garantía de que cada participación corresponde a una persona real verificada.

Este objetivo introduce un problema de seguridad fundamental: distintos actores podrían intentar reconstruir o inferir la relación entre la identidad real de un ciudadano y su actividad dentro de la plataforma.

Durante el diseño surgió la necesidad de definir explícitamente qué tipo de amenazas se asumen como plausibles y frente a cuáles se diseña el sistema.

La decisión es importante porque afecta múltiples aspectos de la arquitectura:

- separación entre servicios
- política de logs y metadatos
- manejo de identidades anónimas
- retención de información operativa
- capacidad de auditoría del sistema

Para evitar ambigüedades se decidió definir un modelo de amenazas explícito en el documento:

`design/threat_model.md`

La pregunta de diseño que motivó este ADR fue:

**¿Qué nivel de adversario debe asumir el sistema en su modelo de amenazas?**

## Opciones consideradas

### Opción 1 — Modelo de amenazas mínimo (usuarios externos)

En esta opción el sistema solo consideraría ataques provenientes de usuarios externos o atacantes técnicos.

Se asumiría que:

- los operadores del sistema son totalmente confiables
- la infraestructura no será comprometida
- no existe riesgo de correlación interna entre componentes

Ventajas

- Arquitectura más simple
- Menor complejidad operativa
- Implementación más rápida

Desventajas

- Riesgo elevado de correlación entre identidad real y participación
- Dependencia completa de la buena conducta de operadores
- Falta de protección frente a errores operativos o filtraciones internas
- Menor credibilidad del sistema desde el punto de vista de privacidad

### Opción 2 — Modelo de amenazas fuerte (adversario total)

En esta opción el sistema se diseñaría para resistir incluso escenarios extremos como:

- compromiso completo de la infraestructura
- vigilancia global de red
- colusión total entre operadores
- análisis avanzado de correlación temporal

Este tipo de modelo suele requerir tecnologías avanzadas como:

- redes de anonimato complejas
- criptografía avanzada (por ejemplo pruebas de conocimiento cero)
- sistemas completamente descentralizados

Ventajas

- Máximo nivel de protección teórica del anonimato
- Resistencia frente a adversarios extremadamente poderosos

Desventajas

- Complejidad técnica muy alta
- Coste operativo elevado
- dificultad de implementación y auditoría
- mayor dificultad para integración con sistemas de verificación de identidad del mundo real

### Opción 3 — Modelo de amenazas intermedio con separación de funciones

En esta opción el sistema asume que:

- pueden existir atacantes externos
- componentes individuales del sistema podrían ser comprometidos
- los operadores de distintos servicios podrían tener acceso a ciertos datos operativos

Sin embargo, la arquitectura se diseña para que:

- ningún componente individual tenga simultáneamente acceso a identidad real y participación
- la información almacenada en cada sistema sea mínima
- la correlación entre sistemas requiera acceso simultáneo a múltiples componentes

Este enfoque se basa en principios como:

- separación de funciones
- minimización de datos
- retención limitada de metadatos
- arquitectura auditable

Ventajas

- Buen equilibrio entre seguridad y complejidad operativa
- Reduce significativamente el riesgo de correlación
- Compatible con infraestructuras realistas
- Permite auditoría independiente del diseño

Desventajas

- No elimina completamente el riesgo frente a colusión total
- No protege frente a compromiso completo de la infraestructura
- Requiere disciplina estricta en el manejo de datos y logs

## Decisión

Se adopta la **Opción 3: modelo de amenazas intermedio con separación de funciones**.

El sistema se diseña asumiendo que distintos componentes pueden ser observados o incluso comprometidos de forma aislada.

Por este motivo:

- la verificación de identidad
- la gestión de identidades anónimas
- la plataforma de participación

deben operar como funciones separadas.

## Justificación

La opción elegida proporciona un equilibrio razonable entre protección de privacidad, viabilidad técnica y operatividad del sistema.

Un modelo de amenazas mínimo dejaría al sistema excesivamente dependiente de la confianza en los operadores y expondría a los participantes a riesgos innecesarios de correlación.

Por otro lado, un modelo de amenazas extremo requeriría tecnologías altamente complejas que dificultarían la implementación, auditoría y operación del sistema en entornos institucionales reales.

El modelo intermedio permite:

- reducir significativamente los riesgos de correlación
- mantener una arquitectura comprensible y auditable
- operar sobre infraestructuras realistas

## Consecuencias

La adopción de este modelo de amenazas introduce varios principios obligatorios para el diseño del sistema:

- separación entre verificación de identidad, gestión de identidad anónima y plataforma de participación
- minimización de datos almacenados en cada componente
- retención limitada de metadatos operativos
- diseño que permita auditoría externa

Decisiones futuras relacionadas con:

- política de logs
- retención de metadatos
- diseño de servicios de identidad
- mecanismos de reemisión de identidad anónima

deberán ser coherentes con este modelo de amenazas.
