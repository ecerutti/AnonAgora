# P-0007 — Proveedores de verificación y niveles de garantía de unicidad

**Estado:** Activo

## Contexto

El sistema busca garantizar que cada persona real pueda participar mediante una única identidad anónima activa.

Para lograr esta propiedad es necesario verificar que cada identidad anónima emitida corresponde a una persona real.

Sin embargo, distintos entornos institucionales ofrecen distintos mecanismos de verificación de identidad.

Algunos proveedores entregan credenciales firmadas asociadas a identificadores únicos estables, mientras que otros solo ofrecen autenticación básica sin garantías fuertes de unicidad.

Esto plantea la pregunta de diseño:

¿Cómo debe manejar el sistema la verificación de identidad cuando los proveedores disponibles ofrecen distintos niveles de garantías?

## Opciones consideradas

### Opción 1 — Requerir un proveedor fuerte de identidad

En esta opción la plataforma solo podría desplegarse en entornos donde exista un proveedor de identidad que ofrezca:

- credenciales firmadas
- identificadores únicos estables
- garantías institucionales fuertes

Ventajas

- garantía fuerte de unicidad
- modelo de seguridad claro

Desventajas

- limita severamente la reutilización del sistema
- impide despliegues en contextos donde no exista un proveedor de identidad fuerte

### Opción 2 — No depender de proveedores externos

En esta opción la plataforma intentaría garantizar la unicidad internamente.

Ventajas

- independencia de sistemas externos

Desventajas

- extremadamente difícil de implementar
- mayor riesgo de identidades duplicadas
- mayor exposición a manipulación interna

### Opción 3 — Plataforma agnóstica del proveedor pero no de las garantías

En esta opción la plataforma puede operar con distintos proveedores de verificación.

El nivel de garantía de unicidad dependerá de las propiedades que ofrezca el proveedor.

Ventajas

- arquitectura flexible
- reutilización en distintos países o instituciones
- posibilidad de obtener garantías fuertes cuando el proveedor lo permite

Desventajas

- distintos despliegues pueden ofrecer distintos niveles de garantía

## Decisión

Se adopta la **opción 3**.

La plataforma se diseña para operar con proveedores externos de verificación de identidad o unicidad.

Cuando el proveedor entrega credenciales firmadas y asociadas a identificadores únicos estables, el sistema puede sostener una garantía fuerte de unicidad entre personas reales e identidades anónimas activas.

Cuando el proveedor no ofrece estas propiedades, el sistema puede operar con garantías de unicidad más débiles.

## Justificación

Esta decisión permite mantener un diseño genérico y reutilizable sin depender de un sistema de identidad específico.

Al mismo tiempo permite aprovechar proveedores fuertes cuando están disponibles.

## Consecuencias

- el sistema debe definir una interfaz clara para proveedores de verificación
- los despliegues deben declarar explícitamente el nivel de garantía de unicidad que ofrecen
- las credenciales emitidas por proveedores externos deben ser verificables
- los identificadores sensibles utilizados para generar identificadores derivados no deben almacenarse bajo ningún concepto
