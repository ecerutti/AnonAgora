# DI-0002 — Persistencia y despliegue de la demo

**Estado:** Activo

## Contexto

Con el stack definido (DI-0001), falta decidir cómo se despliega la demo y dónde persiste cada tipo de dato.

El diseño separa el emisor de la aplicación destino (P-0006, P-0021): el emisor conoce solo `{anon_seed, fecha_emision}` (P-0015) y la aplicación destino solo el `anon_id`. Ningún componente tiene ambos. Los dos componentes se comunican únicamente durante la emisión: consulta-con-reserva y liberación de pseudónimos (P-0025), entrega de la identidad y confirmación de recepción (P-0022). El modelo operativo (`design/modelo_operativo.md`) admite que ambos componentes estén a cargo del mismo operador.

El entorno de despliegue actual de la demo condiciona la decisión de tres formas:

- Provee una única base PostgreSQL.
- Hace backup diario de la base, con retención de 30 días.
- Conserva la salida estándar de los contenedores con rotación por tamaño, no por tiempo.

P-0020 exige que los logs operativos se eliminen a los 7 días y los de modo debug a las 24 horas o al desactivar el modo, y asigna esa responsabilidad a cada componente. P-0005 exige que las sesiones expiren a la hora de inactividad.

## Opciones consideradas

### Decisión 1 — Separación entre emisor y aplicación destino

#### Opción A — Dos servicios

Emisor y aplicación destino corren como dos procesos en contenedores separados, cada uno con su propio esquema en la base. Se comunican por HTTP.

Ventajas

- La separación es estructural: ningún proceso puede leer los datos del otro por un atajo del código.
- El contrato emisor↔aplicación destino se implementa como protocolo real, tal como debería funcionar en producción.

Desventajas

- Despliegue algo más complejo (dos contenedores, comunicación entre ellos).

#### Opción B — Un servicio con dos módulos

Ventajas

- Despliegue más simple.

Desventajas

- La separación queda como convención del código; un atajo puede romperla sin que se note.
- El contrato entre componentes no se ejercita como protocolo.

### Decisión 2 — Almacenamiento de logs

#### Opción A — Archivos dentro del contenedor

Cada componente escribe sus logs en archivos diarios dentro de su contenedor y los purga por sí mismo según P-0020.

Ventajas

- Los logs no entran en los backups de la base; la retención efectiva coincide con la de P-0020.
- Cada componente gestiona la eliminación de sus propios logs, como exige P-0020.

Desventajas

- Los logs se pierden al redesplegar el contenedor.
- Consultarlos requiere acceso al contenedor.

#### Opción B — Tabla en la base

Ventajas

- Fáciles de consultar.

Desventajas

- Los backups los conservan 30 días, lo que viola la retención de P-0020.

#### Opción C — Salida estándar del contenedor

Ventajas

- Convención habitual en contenedores; sin código adicional.

Desventajas

- La rotación por tamaño no garantiza ningún plazo: con poco tráfico, los logs pueden conservarse mucho más de 7 días. La aplicación no puede purgarlos.

### Decisión 3 — Almacenamiento de sesiones

#### Opción A — En memoria de la aplicación destino

Ventajas

- Las sesiones, que vinculan un token con un `anon_id`, nunca tocan disco ni backups.
- La expiración por inactividad es trivial de implementar.

Desventajas

- Un reinicio de la aplicación cierra todas las sesiones.
- No escala a más de una instancia de la aplicación.

#### Opción B — En la base

Ventajas

- Las sesiones sobreviven a reinicios.

Desventajas

- Las sesiones quedan en los backups durante 30 días.

## Decisiones

**Decisión 1:** dos servicios (Opción A). El emisor y la aplicación destino son dos procesos en contenedores separados, orquestados con Docker Compose, construidos desde el mismo repositorio (`demo/src/`). Cada uno usa su propio esquema en la base PostgreSQL. Si el entorno lo permite, cada uno accede con un usuario de base restringido a su esquema. Ambos exponen interfaz al ciudadano: el emisor sirve las pantallas de F-CI-01 (incluida la simulación de AUTENTICAR) y la aplicación destino el resto.

**Decisión 2:** logs en archivos diarios dentro de cada contenedor (Opción A), con purga a cargo de cada componente: 7 días para los operativos, 24 horas o desactivación del modo para los de debug. La salida estándar se limita a mensajes de arranque y detención que no contienen datos de eventos.

**Decisión 3:** sesiones en memoria de la aplicación destino (Opción A).

Los secretos se leen de variables de entorno y nunca se versionan: el salt del sistema (solo el emisor) y la clave de la API de moderación (solo la aplicación destino). Cada servicio recibe únicamente los secretos que usa.

## Justificación

La demo existe para mostrar el diseño, y la separación de funciones es su núcleo. Dos servicios la convierten en una propiedad estructural y obligan a ejercitar el contrato entre componentes. Los backups de 30 días hacen que todo lo que se guarde en la base persista al menos ese tiempo; por eso, los datos sujetos a retención corta (logs) o sensibles y efímeros (sesiones) quedan fuera de la base. Los datos de la base de la demo son ficticios (DP-0002), así que su permanencia en backups es una limitación aceptada.

## Consecuencias

- El contrato concreto emisor↔aplicación destino (formato, endpoints, autenticación entre servicios, política de reintentos de la entrega) debe definirse antes del desarrollo. Es el ADR diferido que referencia P-0022.
- El despliegue requiere configurar dos servicios y la forma en que el ciudadano accede a cada uno (dominios o rutas).
- Un reinicio de la aplicación destino cierra todas las sesiones; un redespliegue elimina los logs.
- La aplicación destino corre en una sola instancia.
- Los datos de la base (incluidos `{anon_seed, fecha_emision}` en el emisor) permanecen en los backups 30 días. Esto es aceptable porque los CUILs de la demo son ficticios (DP-0002) y no sería aceptable en producción sin una política de backups acorde.

## Referencias

- DI-0001 — Stack tecnológico de la demo
- DP-0002 — Alcance funcional del MVP de la demo
- P-0005 — Sesiones temporales
- P-0006 — Modelo de amenazas y confianza
- P-0015 — Qué almacena el emisor
- P-0020 — Política de logs y retención
- P-0021 — Arquitectura modular
- P-0022 — Comportamiento ante fallos y atomicidad de la emisión
- P-0025 — Unicidad de pseudónimos
- `design/modelo_operativo.md`
