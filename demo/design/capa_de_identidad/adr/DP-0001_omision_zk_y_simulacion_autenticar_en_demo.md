# DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo

**Estado:** Activo

## Contexto

El sistema adopta en P-0014 pruebas de conocimiento cero (ZK) como mecanismo de auditoría criptográfica de legitimidad del emisor, y en P-0013 la integración con AUTENTICAR como proveedor de verificación de identidad. Ambas decisiones aplican al sistema en producción.

La implementación demo tiene como objetivo demostrar el funcionamiento de la plataforma participativa en un entorno controlado. Su alcance es funcional y conceptual, no de seguridad criptográfica. Este ADR documenta las simplificaciones que aplican en ese contexto respecto de P-0013 y P-0014.

## Opciones consideradas

### Decisión 1 — Integración con AUTENTICAR en la demo

#### Opción A — Integrar AUTENTICAR real en la demo

Implementar el flujo OAuth 2.0 / OIDC completo contra el ambiente de testing de AUTENTICAR.

Ventajas

- La demo reproduce fielmente el flujo de verificación de identidad del sistema real.

Desventajas

- Requiere credenciales de acceso al ambiente de testing de AUTENTICAR, que se entregan bajo solicitud formal y no están disponibles de forma inmediata.
- Introduce una dependencia externa que puede estar caída o inaccesible durante una demostración.
- El foco de la demo es la plataforma participativa, no el flujo de verificación de identidad, que ya está documentado en P-0013.

#### Opción B — Simular AUTENTICAR mediante un mecanismo interno

La demo implementa un mecanismo propio que simula la verificación de identidad sin integrarse con AUTENTICAR real. El emisor acepta una identidad de prueba sin validar un token JWT real.

Ventajas

- Sin dependencias externas. La demo funciona de forma completamente autónoma.
- Permite demostrar el flujo completo de la plataforma participativa sin fricción operativa.

Desventajas

- El flujo de verificación de identidad no es representativo del sistema real.

### Decisión 2 — Implementación de ZK en la demo

#### Opción A — Implementar ZK en la demo

Incluir el componente de proving ZK en el emisor de la demo, tal como define P-0014.

Ventajas

- La demo reproduce el comportamiento de seguridad del sistema real.

Desventajas

- ZK resuelve un problema de auditoría de legitimidad del emisor que solo tiene sentido cuando existe un token JWT real de AUTENTICAR que probar. Sin AUTENTICAR real (Decisión 1, Opción B), no hay token que demostrar y la prueba ZK no certifica nada significativo.
- Agrega complejidad de implementación y dependencias de librerías ZK sin aportar valor demostrativo.
- Requiere trusted setup y circuito compilado, lo cual es desproporcionado para el alcance de una demo.

#### Opción B — Omitir ZK en la demo

La demo no implementa el componente de proving ZK. El emisor simula la emisión de identidades anónimas sin generar pruebas criptográficas.

Ventajas

- Sin complejidad adicional de implementación.
- Coherente con la omisión de AUTENTICAR real: si no hay JWT real, no tiene sentido generar una prueba ZK sobre él.

Desventajas

- La garantía de auditoría criptográfica de legitimidad del emisor no está presente en la demo. Esta limitación es conocida y explícita.

## Decisiones

**Decisión 1:** La demo simula AUTENTICAR mediante un mecanismo interno (Opción B). No se integra con el ambiente de testing de AUTENTICAR real.

**Decisión 2:** La demo omite la implementación de ZK (Opción B). La ausencia de ZK en la demo es una consecuencia directa de la ausencia de tokens JWT reales de AUTENTICAR: sin token real no existe el problema de auditoría de legitimidad que ZK resuelve.

## Justificación

El objetivo de la demo es demostrar el funcionamiento de la plataforma participativa —propuestas, apoyos, identidades anónimas, límites operativos— no el mecanismo de auditoría criptográfica del emisor. Las simplificaciones adoptadas son coherentes entre sí: sin AUTENTICAR real no hay JWT real, y sin JWT real ZK no aporta valor demostrativo. Ambas omisiones están documentadas explícitamente y no representan decisiones de diseño que deban trasladarse al sistema en producción.

## Consecuencias

- El emisor de la demo no implementa el flujo OAuth 2.0 / OIDC con AUTENTICAR ni el componente de proving ZK.
- La demo debe dejar claro en su documentación interna que estas omisiones son simplificaciones del entorno demo y no del diseño del sistema.
- Quien implemente el sistema en producción debe referirse a P-0013 y P-0014 para las decisiones correspondientes, no a la implementación de la demo.

## Referencias

- P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad
- P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero
