# P-0016 — Invalidación de identidades anónimas en la plataforma participativa

## Contexto

Durante el diseño del ciclo de vida de identidades anónimas (P-0015) quedó pendiente una decisión que corresponde exclusivamente al modelo de datos de la plataforma participativa: si un ciudadano sospecha que sus credenciales fueron robadas y su identidad anónima está siendo usada por un tercero, ¿puede solicitar la invalidación de ese `anon_id`?

La pregunta es relevante porque la plataforma es el único componente que puede actuar sobre un `anon_id` activo: el emisor no almacena `anon_ids` y no tiene mecanismo de revocación sobre ellos (P-0015).

El diseño debe evaluar si el beneficio de permitir la invalidación justifica los riesgos que introduce.

## Opciones consideradas

### Opción A — Sin mecanismo de invalidación

La plataforma no provee ningún mecanismo para desactivar un `anon_id`, ni a solicitud del ciudadano ni por acción administrativa. Las identidades permanecen activas hasta que el ciudadano renueva su identidad al vencimiento del cool-down.

Ventajas

- No introduce ningún vector de ataque nuevo sobre la plataforma.
- Elimina la posibilidad de que un administrador malicioso o un atacante con acceso a la plataforma invalide identidades de forma masiva o selectiva.
- Simplifica el modelo de datos: no existen estados adicionales en las identidades.

Desventajas

- Un ciudadano con credenciales robadas no puede detener el uso ilegítimo de su identidad. Vencido el cool-down puede obtener una nueva identidad, pero la robada seguirá activa indefinidamente junto a ella.

### Opción B — Invalidación a solicitud del ciudadano

El ciudadano puede solicitar la invalidación de su `anon_id` activo autenticándose con sus credenciales. La plataforma marca el `anon_id` como inactivo.

Ventajas

- El ciudadano puede limitar el daño de un robo de credenciales sin esperar el cool-down.

Desventajas

- Para invalidar, el ciudadano debe autenticarse con las mismas credenciales que sospecha comprometidas. Si el atacante ya las tiene, también puede invalidar.
- La invalidación por parte del ciudadano y por parte del atacante son indistinguibles para el sistema.
- Un atacante con acceso a credenciales de múltiples ciudadanos puede invalidar un gran número de identidades de forma coordinada, efectuando un ataque de denegación de servicio sobre la participación.
- Un administrador malicioso con acceso a la base de datos puede invalidar identidades sin necesidad de credenciales, sin dejar rastro distinguible de invalidaciones legítimas.
- Una vez que existe el mecanismo de invalidación, es técnicamente imposible determinar qué invalidaciones fueron solicitadas por el ciudadano real y cuáles son resultado de un ataque.

### Opción C — Invalidación administrativa

Los operadores de la plataforma pueden marcar `anon_ids` como inactivos por decisión administrativa, por ejemplo ante comportamiento abusivo detectado.

Ventajas

- Habilita una capacidad de moderación a nivel de identidad.

Desventajas

- Introduce la posibilidad de silenciamiento arbitrario de ciudadanos por parte de los operadores, lo que contradice los principios del sistema.
- Comparte los riesgos de auditoría de la Opción B: es imposible distinguir invalidaciones legítimas de abusivas una vez que el mecanismo existe.

## Decisión

Se adopta la **Opción A**. La plataforma no implementa ningún mecanismo de invalidación de `anon_ids`. Las identidades anónimas no tienen estado de inactivación dentro de la plataforma.

## Justificación

El daño que puede causar una identidad robada está acotado por el propio diseño del sistema: el atacante opera con los mismos límites que el ciudadano legítimo. Puede emitir apoyos, modificar apoyos propios o crear propuestas si el cupo anual lo permite. No puede actuar fuera de esos límites ni obtener información sobre la identidad real del ciudadano.

La Opción B introduce un riesgo desproporcionado: cualquier mecanismo de invalidación es un vector de ataque de denegación de servicio. Un atacante con credenciales robadas o un administrador malicioso con acceso a la base de datos puede invalidar identidades de forma masiva. Lo crítico no es solo el daño directo del ataque, sino que el sistema no tiene forma de distinguir invalidaciones legítimas de maliciosas, haciendo el daño imposible de auditar y la recuperación incierta.

La asimetría de costos es clara: el costo de no tener invalidación es bajo y acotado; el costo de tenerla es un vector de ataque sin contramedida técnica.

La moderación de contenido abusivo, si se considera necesaria, es un problema separado que debe resolverse sobre propuestas y apoyos individuales,
