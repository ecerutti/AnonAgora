# P-0026 — Contrato de entrega entre emisor y aplicación destino

**Estado:** Activo

## Contexto

El contrato capa↔aplicación destino (`design/capa_de_identidad/README.md`) establece qué entrega el emisor —la tupla `{pseudónimo, anon_id, prueba ZK}`, una sola vez— y P-0025 agrega dos operaciones previas: consulta-con-reserva y liberación de pseudónimos, invocadas por el emisor sobre la aplicación destino por un canal servidor a servidor.

P-0022 Decisión 3 fija el principio de atomicidad de la emisión: el emisor persiste `{anon_seed, fecha_emision}`, y con eso consume el cool-down, solo después de recibir confirmación de la aplicación destino de que la entrega fue recibida y aceptada. P-0022 difiere a un ADR posterior la mecánica concreta de esa confirmación, la política de reintentos de la entrega y el formato del flujo de entrega. Este es ese ADR.

El inventario de flujos (`notas/inventario_de_flujos_ui_ux.md`, F-CI-01 y F-AP-01) ubica la confirmación después de que el ciudadano define su frase secreta en la aplicación destino y esta persiste su registro. Entre la entrega y la confirmación pueden pasar minutos, o la confirmación puede no llegar nunca si el ciudadano abandona. Durante ese intervalo, ambos componentes sostienen estado provisional.

Las preguntas que este ADR responde son:

- ¿Por qué canal viaja la tupla y cómo pasa el ciudadano del emisor a la aplicación destino?
- ¿Qué estado provisional sostiene cada componente entre la entrega y la confirmación, dónde y por cuánto tiempo?
- ¿Cómo se confirma, qué pasa si la confirmación falla y cuál es el fallo residual aceptado?
- ¿Cómo se autentican los componentes entre sí?

Un requisito atraviesa todas las respuestas: la separación entre `anon_seed` y `anon_id` (P-0015 Decisión 3) debe sobrevivir también al estado provisional. P-0025 establece que la aplicación destino guarda sus reservas, con el identificador efímero del flujo de emisión, en una tabla de su base. Si el emisor guardara en su base el mismo identificador junto al `anon_seed`, cualquier copia que contenga ambas bases —un backup, por ejemplo— vincularía `anon_seed` con `anon_id` a través del identificador de flujo.

## Opciones consideradas

### Decisión 1 — Canal de la entrega

#### Opción A — Servidor a servidor, con ticket opaco para el navegador

El emisor envía la tupla a la aplicación destino por el canal servidor a servidor. La aplicación destino responde con un ticket opaco de un solo uso. El navegador del ciudadano pasa a la aplicación destino llevando solo ese ticket.

Ventajas

- El `anon_id` nunca pasa por el navegador, ni por URLs, historiales o registros intermedios.
- Usa el canal servidor a servidor que P-0025 ya exige; no agrega un canal nuevo.
- La aplicación destino recibe la tupla directamente del emisor autenticado; el navegador no puede alterarla.

Desventajas

- Requiere conectividad de red directa entre emisor y aplicación destino (ya requerida por P-0025).

#### Opción B — A través del navegador, firmada

El emisor entrega la tupla firmada al navegador, que la presenta a la aplicación destino (estilo OIDC o SAML).

Ventajas

- Patrón conocido de federación de identidad.

Desventajas

- El `anon_id` queda expuesto en el navegador y potencialmente en historiales o registros.
- No elimina el canal servidor a servidor, que P-0025 exige igual.
- Requiere gestión de claves de firma y verificación de vigencia y repetición.

### Decisión 2 — Mecánica de confirmación y fallo residual

P-0022 ya decidió el orden: la aplicación destino persiste primero y el emisor consume el cool-down después. La decisión abierta es qué ocurre si la confirmación no llega.

#### Opción A — Confirmación con reintentos; se acepta la posible segunda identidad

La aplicación destino persiste el registro y confirma al emisor, reintentando con backoff dentro de una ventana configurable. Si la confirmación no llega dentro del plazo en que el emisor conserva el estado pendiente, el ciudadano queda registrado pero el cool-down no se consume.

Ventajas

- El ciudadano nunca queda sin identidad y con cool-down consumido, que es el resultado que P-0022 descartó.
- Sin coordinación adicional.

Desventajas

- En el caso residual, el ciudadano podría obtener una segunda identidad antes de que venza el cool-down que debió consumirse. El ciudadano no puede provocar el caso: depende de un fallo entre servidores.

#### Opción B — Compromiso en dos fases con coordinador

Un protocolo de compromiso en dos fases garantiza que ambos componentes persisten o ninguno lo hace.

Ventajas

- Elimina, en teoría, el caso residual.

Desventajas

- Requiere estado persistente de coordinación en ambos componentes, que choca con el requisito de no persistir el identificador de flujo en el emisor.
- Complejidad desproporcionada; el bloqueo del coordinador introduce sus propios modos de fallo.

### Decisión 3 — Dónde sostiene el emisor el estado pendiente

#### Opción A — Solo en memoria

Entre la entrega y la confirmación, el emisor conserva `{anon_seed, identificador de flujo, vencimiento}` únicamente en memoria.

Ventajas

- El identificador de flujo y el `anon_seed` nunca coexisten en almacenamiento persistente. Ninguna copia de las bases de datos, ni siquiera una que contenga las de ambos componentes, permite vincular `anon_seed` con `anon_id`.

Desventajas

- Si el emisor se reinicia con emisiones pendientes, pierde ese estado. Las confirmaciones correspondientes llegan a un emisor que no las reconoce, y caen en el caso residual de la Decisión 2.

#### Opción B — En su base de datos

Ventajas

- El estado pendiente sobrevive a reinicios.

Desventajas

- El identificador de flujo y el `anon_seed` quedan juntos en almacenamiento persistente y en sus copias. Combinados con la tabla de reservas de la aplicación destino, vinculan `anon_seed` con `anon_id`, y rompen la separación de P-0015 frente al atacante con acceso a ambos componentes que contempla P-0006.

### Decisión 4 — Autenticación entre componentes

#### Opción A — TLS mutuo (mTLS)

Cada componente presenta un certificado que el otro verifica, en ambas direcciones.

Ventajas

- Autenticación fuerte en ambas direcciones, sin secretos compartidos que viajen en cada petición.
- Adecuado para componentes en infraestructuras y bajo operadores distintos.

Desventajas

- Requiere gestión de certificados (emisión, rotación, revocación).

#### Opción B — Secreto compartido

Ventajas

- Simple de configurar.

Desventajas

- Un mismo secreto en ambos componentes: su filtración desde cualquiera de los dos permite suplantar a ambos.
- Inadecuado cuando los componentes están bajo operadores distintos.

## Decisiones

**Decisión 1:** Opción A. La entrega viaja servidor a servidor y el navegador lleva solo un ticket opaco de un solo uso.

**Decisión 2:** Opción A. Confirmación con reintentos; el caso residual de posible segunda identidad se acepta.

**Decisión 3:** Opción A. El emisor sostiene el estado pendiente solo en memoria.

**Decisión 4:** Opción A. Emisor y aplicación destino se autentican mutuamente con mTLS en todas las operaciones del protocolo de emisión.

### Protocolo resultante

Operaciones del emisor sobre la aplicación destino:

- **Reservar** (P-0025): `{id_flujo, pseudónimo}` → `libre` u `ocupado`.
- **Liberar** (P-0025): `{id_flujo}` → `ok`.
- **Entregar**: `{id_flujo, pseudónimo, anon_id, prueba ZK}` → `{ticket}`, `colisión` (P-0025, colisión post-TTL) o error. Antes de responder, la aplicación destino verifica la prueba ZK (P-0014), comprueba que el pseudónimo coincide con la reserva vigente del flujo y convierte la reserva en un registro pendiente con plazo de vigencia.

Operación de la aplicación destino sobre el emisor:

- **Confirmar**: `{id_flujo}` → `ok` o `desconocido`.

Secuencia:

1. El ciudadano acepta el pseudónimo en el emisor (F-CI-01).
2. El emisor invoca **Entregar**. Si recibe el ticket, descarta de inmediato el `anon_id` y la prueba ZK y conserva en memoria solo `{anon_seed, id_flujo, vencimiento}`.
3. El emisor muestra al ciudadano un paso de continuación cuyo envío lleva el ticket a la aplicación destino en el cuerpo de una petición POST, nunca en la URL.
4. La aplicación destino canjea el ticket (que queda invalidado), asocia el registro pendiente a la sesión de onboarding y presenta la definición de la frase secreta (F-AP-01).
5. Al persistir el registro del ciudadano, con la verificación atómica de unicidad de P-0025, la aplicación destino elimina el `id_flujo` de todo lo persistido y lo conserva solo para la confirmación.
6. La aplicación destino invoca **Confirmar**, con reintentos y backoff exponencial dentro de una ventana configurable.
7. Al recibir **Confirmar** con un `id_flujo` pendiente, el emisor persiste `{anon_seed, fecha_emision}` (P-0015) y descarta el estado pendiente. Con un `id_flujo` desconocido responde `desconocido`; la aplicación destino deja de reintentar y conserva el registro del ciudadano (caso residual de la Decisión 2).

Plazos (configurables por el operador):

- **Plazo del registro pendiente** en la aplicación destino: tiempo máximo entre la entrega y la persistencia del registro. Default: 30 minutos. Vencido, la aplicación destino descarta el registro pendiente y libera el pseudónimo; el ciudadano debe reiniciar F-CI-01, sin consumo de cool-down.
- **Vencimiento del estado pendiente** en el emisor: debe superar al plazo del registro pendiente más la ventana de reintentos de la confirmación, para que una confirmación legítima no llegue a un emisor que ya descartó el estado. Default: 60 minutos.

## Justificación

El canal servidor a servidor ya existe por P-0025, y usarlo para la entrega mantiene el `anon_id` fuera del navegador sin costo adicional. La entrega a través del navegador solo tendría sentido si eliminara el canal interno, y no lo hace.

La confirmación con reintentos aplica el criterio de asimetría de costos de P-0022: entre dejar a un ciudadano sin identidad y bloqueado por el cool-down, y permitir en un caso residual infrecuente una segunda identidad que el ciudadano no puede provocar, el segundo error es preferible. Un compromiso en dos fases eliminaría el caso residual a cambio de estado persistente de coordinación, justo lo que la Decisión 3 descarta.

Mantener el estado pendiente del emisor solo en memoria preserva la propiedad central de P-0015 en el único momento en que el emisor conoce simultáneamente el `anon_seed` y un identificador que también existe en la aplicación destino. La pérdida de estado ante un reinicio se resuelve en el mismo caso residual ya aceptado, sin introducir uno nuevo.

mTLS es el mecanismo adecuado para componentes que el modelo operativo admite bajo infraestructuras y operadores distintos (`design/modelo_operativo.md`). Un secreto compartido daría a cada operador la capacidad de suplantar al otro.

## Consecuencias

- El contrato capa↔aplicación destino se completa con la operación **Entregar**, la operación **Confirmar** en sentido inverso y la secuencia descrita. Se actualiza `design/capa_de_identidad/README.md`.
- El emisor nunca persiste el identificador de flujo. La aplicación destino lo persiste solo en sus reservas y registros pendientes, y lo elimina al persistir el registro definitivo del ciudadano.
- Un reinicio del emisor con emisiones pendientes, o un fallo prolongado de la confirmación, puede dejar ciudadanos registrados sin cool-down consumido. El caso no expone datos ni perjudica al ciudadano. La aplicación destino lo registra en sus logs como error de integración, sin identificadores (P-0020).
- El estado pendiente del emisor se acota a una sola instancia del emisor o a un mecanismo de memoria compartida no persistente; replicar el emisor exige preservar esa propiedad.
- Los despliegues requieren una infraestructura de certificados para mTLS entre emisor y aplicación destino.
- El ticket debe ser aleatorio (al menos 128 bits), de un solo uso, vencer con el plazo del registro pendiente y no registrarse en logs.
- Este ADR completa lo que P-0022 Decisión 3 difirió y no modifica P-0015 ni P-0025.

## Referencias

- P-0006 — Modelo de amenazas y confianza
- P-0014 — Auditoría ZK de legitimidad del emisor
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0020 — Política de logs y retención de metadatos
- P-0021 — Arquitectura modular
- P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos
- P-0025 — Unicidad y desambiguación de pseudónimos
- `design/capa_de_identidad/README.md` — Contrato capa↔aplicación destino
- `design/modelo_operativo.md` — Modelo operativo
- `notas/inventario_de_flujos_ui_ux.md` — F-CI-01 y F-AP-01
