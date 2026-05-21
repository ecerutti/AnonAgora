# Capa de identidad

Diseño de la capa de identidad: la infraestructura de identidad anónima verificada definida en P-0021. La capa encapsula la integración con el verificador externo, la emisión de identidades anónimas y la provisión de credenciales que el ciudadano usa para autenticarse en la aplicación destino.

## Componentes

La capa de identidad está compuesta por:

- **Emisor.** Verifica que el ciudadano sea una persona real mediante el verificador externo, deriva el `anon_seed`, genera el `anon_id` y la prueba ZK de legitimidad, y entrega el resultado a la aplicación destino. Su modelo de datos y ciclo de vida están definidos en P-0015.
- **Integración con verificador externo.** El emisor opera con un proveedor externo de verificación de identidad. P-0007 establece la propiedad genérica de la integración; P-0013 concreta la decisión para el contexto argentino (AUTENTICAR, restringido a ARCA y ANSES).
- **Componente de proving ZK.** Genera, en cada emisión, una prueba criptográfica de que el `anon_id` proviene de un ciudadano real verificado, sin revelar la identidad real. P-0014 define el mecanismo, el stack y la arquitectura de generación.
- **Infraestructura de JWKS histórico.** Mantiene el registro de claves públicas del verificador externo necesario para que las pruebas ZK puedan verificarse aún después de rotaciones de clave. P-0014 lo establece como requisito.

La política transversal de logs y retención que aplica al emisor está definida en P-0020.

## Contrato capa ↔ aplicación destino

El contrato describe qué se intercambia entre la capa y la aplicación destino y qué condiciones asume cada parte. Es la consecuencia consolidada de P-0015, P-0014 y P-0021, y aplica a cualquier aplicación destino construida sobre la capa.

### Qué entrega la capa al momento de la emisión

Por cada ciudadano verificado, el emisor entrega a la aplicación destino:

- el **pseudónimo amigable** asociado a la identidad emitida (P-0002, P-0003);
- el **`anon_id`** correspondiente (P-0015 Decisión 3);
- la **prueba ZK** que certifica la legitimidad del `anon_id` (P-0014).

La entrega ocurre una sola vez, en el momento de la emisión. A partir de ese momento la aplicación destino opera de forma independiente del emisor.

### Operaciones del protocolo de emisión

Durante el flujo de emisión, antes de la entrega final, el emisor consume dos operaciones adicionales del protocolo de emisión que la aplicación destino expone (P-0025):

- **Consulta-con-reserva.** El emisor envía un pseudónimo candidato y un identificador efímero del flujo de emisión. La aplicación destino, atómicamente, verifica disponibilidad y registra la reserva con TTL si el pseudónimo está libre. Responde "libre" u "ocupado".
- **Liberación.** El emisor envía el identificador efímero del flujo de emisión. La aplicación destino libera la reserva asociada. Se invoca cuando el ciudadano regenera el pseudónimo.

La verificación atómica final de unicidad ocurre en el commit de la entrega (P-0022 Dec 3).

### Qué garantiza la capa

- **Una identidad anónima activa por ciudadano real, por aplicación destino**, sujeta al cool-down configurable (default 6 meses) entre emisiones sucesivas (P-0015 Decisión 1).
- **Auditoría criptográfica autónoma**: cualquier auditor externo puede verificar, con información pública, que cada `anon_id` emitido proviene de un ciudadano real verificado por el verificador externo, sin necesidad de cruzar datos con el emisor (P-0014).
- **Separación criptográfica entre `anon_seed` y `anon_id`**: el `anon_id` se deriva con un nonce aleatorio que se descarta inmediatamente, lo que hace que el vínculo entre ambos identificadores no pueda reconstruirse aunque se acceda al estado del emisor (P-0015 Decisión 3).
- **El emisor no almacena `anon_ids` ni asociaciones con ciudadanos**: solo retiene la tupla mínima `{anon_seed, fecha_emision}` (P-0015 Decisión 2).

### Qué no es responsabilidad de la capa

- El **mecanismo de credencial de acceso** del ciudadano (qué prueba la posesión de la identidad anónima durante el login). La aplicación destino lo decide. La aplicación de participación ciudadana adopta passphrase (P-0008, P-0009).
- La **gestión de sesiones** del ciudadano dentro de la aplicación destino.
- El **modelo de datos del comportamiento** del ciudadano (sus acciones dentro de la aplicación, su historial, sus interacciones con otros ciudadanos).
- Las **reglas funcionales de la aplicación**: qué puede hacer el ciudadano, con qué límites, bajo qué moderación.

### Qué restricciones impone la capa

- La aplicación destino **no debe persistir el `anon_seed`** ni recibirlo en ninguna comunicación. Solo recibe `anon_id`, pseudónimo y prueba ZK (P-0020 prohíbe explícitamente `anon_seed` en logs de la aplicación).
- La aplicación destino **no debe involucrar al emisor en el manejo de la credencial de acceso del ciudadano**. La credencial es asunto exclusivo de la aplicación; el emisor no la conoce, ni siquiera como hash (P-0015).
- La aplicación destino **no puede solicitar al emisor la invalidación o revocación de un `anon_id`** porque el emisor no almacena `anon_ids`. Cualquier mecanismo de ciclo de vida posterior a la emisión es responsabilidad de la aplicación destino (decisión que toma cada aplicación; en participación ciudadana, ver P-0016).
- La aplicación destino **debe verificar la prueba ZK** al recibirla y usar el JWKS histórico para validarla.

## Documentos relacionados

- `design/capa_de_identidad/identity_model.md` — modelo de identidad: pseudónimos, `anon_seed`, `anon_id`, separación de identificadores.
- `design/threat_model.md` — modelo de amenazas del sistema, transversal a capa y aplicación destino.
- `design/capa_de_identidad/adr/` — Architecture Decision Records de diseño de la capa de identidad.
