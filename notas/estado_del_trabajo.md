# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

No hay decisiones de plataforma pendientes.

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma. Incluye una decisión pendiente surgida durante el diseño de P-0014:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Manejo de la identidad anónima en la demo (simulación de la verificación de unicidad). La demo no usa AUTENTICAR real ni implementa ZK; ambas omisiones deben quedar documentadas en un ADR de demo (DP-XXXX) que explique por qué no aplican en ese contexto.
6. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
7. Autenticación administrativa para gestión técnica de la demo.
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **ADR P-0020 — Política de logs y retención de metadatos.** Cerrado. Granularidad mínima por tipo de evento en modo operativo. Campos prohibidos diferenciados por componente. Modo debug activable explícitamente con visibilidad obligatoria. Gestión interna de retención por cada componente (default 7 días operativo, 1 día debug). Limitaciones conocidas documentadas: backups a nivel de sistema, copia manual de logs, logs de infraestructura fuera del alcance de la plataforma.

- **ADR P-0019 — Búsqueda y filtrado de propuestas.** Cerrado. Búsqueda full-text con morfología y stopwords en español sobre título y cuerpo, ponderada a favor del título. Filtros: emergente, tendencia, cantidad de vínculos, vínculos a propuestas específicas (AND), rango de fechas, rango de apoyos, apoyadas/no apoyadas por el ciudadano (requiere sesión).

- **ADR P-0018 — Modelo de datos de propuestas.** Cerrado. Campos: id, titulo (texto plano, default 200 caracteres configurable), cuerpo (Markdown, default 20.000 caracteres configurable), fecha_publicacion, conteo_apoyos, score, vinculos. Sin autoría almacenada en la propuesta. Sin imágenes. Links externos permitidos como texto plano no clickeable.

- **ADR P-0017 — Límite anual de propuestas por ciudadano.** Cerrado. Límite configurable con default 2, valor 0 válido (sin límite). Año móvil de 365 días por slot. Las propuestas derivadas consumen cupo.

- **Documento de diseño — Vinculación entre propuestas.** Cerrado. Vínculos genéricos sin tipo, múltiples por propuesta (default 10, configurable), inmutables, sin aceptación del autor referenciado. Propuesta derivada es una propuesta común con vínculo, sin entidad ni nomenclatura especial.

- **ADR P-0016 — Invalidación de identidades anónimas en la plataforma participativa.** Cerrado. La plataforma no implementa ningún mecanismo de invalidación de `anon_ids`. Las identidades no tienen estado de inactivación. Una identidad robada no puede ser revocada, pero el daño está acotado por los límites del sistema. El mecanismo de invalidación fue descartado porque introduce un vector de ataque de denegación de servicio sin contramedida técnica posible.

- **ADR P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas.** Cerrado. Decisiones formalizadas:
  - Cool-down de 6 meses configurable por el operador como mecanismo de control de abuso. Contado desde `fecha_emision` de la identidad activa. Sin límite de renovaciones de por vida.
  - El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena `anon_id`, pseudónimo amigable, nonce, ni ningún dato relacionado con la frase secreta del ciudadano.
  - El `anon_id` se deriva como `HASH(anon_seed + nonce)`, donde `nonce` es un valor aleatorio generado por el emisor en cada emisión y descartado inmediatamente sin almacenarse. La separación entre `anon_seed` y `anon_id` es criptográficamente forzada porque el nonce no existe fuera del momento de emisión.
  - El flujo de emisión ocurre en una única interacción. El emisor verifica el JWT de AUTENTICAR, calcula el `anon_seed`, verifica unicidad y cool-down, y si procede genera el nonce, calcula el `anon_id`, genera la prueba ZK, y descarta el nonce.
  - La frase secreta del ciudadano no interviene en el flujo de emisión. Es asunto exclusivo de la plataforma.
  - La plataforma recibe del emisor `{pseudónimo amigable, anon_id, prueba ZK}` y opera de forma completamente independiente a partir de ese momento.
  - ZK cubre únicamente el `anon_id`. La certificación del `anon_seed` mediante ZK queda descartada. P-0015 superseda parcialmente P-0014 en el alcance del circuito ZK, que pasa a certificar la cadena completa hasta el `anon_id`.
  - La infraestructura de JWKS histórico y revocación por `kid` comprometido de P-0014 se mantienen vigentes.
  - La pérdida de credenciales (pseudónimo, frase secreta o ambos) hace la identidad irrecuperable. El ciudadano espera el cool-down y solicita una nueva identidad completa. Este comportamiento refuerza la percepción de anonimato.
  - La plataforma no puede invalidar `anon_ids`. No existe revocación técnica. Esa decisión queda fuera del alcance del emisor y corresponde al modelo de datos de la plataforma participativa.
  - El historial de identidades inactivas permanece visible y sigue contando. El sistema no distingue identidades abandonadas de inactivas.

- **ADR P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero.** Cerrado. Decisiones formalizadas:
  - Se adopta ZK como mecanismo de auditoría de legitimidad del emisor. Superseda P-0013 Decisión 4 (que era temporal y procedimental).
  - La generación de la prueba se realiza en el servidor (emisor), no en el cliente. La generación en cliente fue descartada por tiempos de proving de 60-180 segundos en dispositivos de gama media-baja y requisitos de RAM incompatibles con smartphones del segmento objetivo.
  - Stack adoptado: circom + snarkjs con el circuito RSA de `zkemail/zk-email-verify` (auditado) como base. Esquema Groth16 sobre BN254. Pruebas de ~256 bytes, verificación en 1-5 ms en servidor.
  - El circuito certifica la cadena completa hasta el `anon_id`: existe un JWT válido de AUTENTICAR cuyo CUIT produce el `anon_seed`, y existe un nonce generado por el emisor que combinado con el `anon_seed` produce el `anon_id`. Witnesses privados: CUIT, `anon_seed`, nonce. Outputs públicos: `anon_id`, `publicKeyHash`, `kid`.
  - La adaptación del circuito (extracción del claim `cuit`, cálculo de `anon_seed`, derivación de `anon_id`, separación de dominios en hash Poseidon) no está auditada y requiere auditoría especializada en ZK antes de cualquier despliegue en producción. Costo estimado: USD 30.000–150.000.
  - `zkemail/zk-jwt` no debe usarse como dependencia directa (autodeclarado no apto para producción, sin auditoría). Puede consultarse como referencia de implementación.
  - El emisor debe implementar un servicio de JWKS histórico para mantener verificabilidad de pruebas tras rotaciones de clave de AUTENTICAR.
  - El emisor debe implementar un mecanismo de revocación por `kid` comprometido.
  - Requiere ceremonia de trusted setup Phase 2 antes del despliegue en producción. Phase 1 puede reutilizarse de la ceremonia pública de Hermez/Polygon.

- **ADR P-0013 — Integración con AUTENTICAR.** Cerrado. Decisiones formalizadas:
  - Se usa AUTENTICAR como proveedor de verificación de identidad.
  - Se aceptan únicamente ARCA y ANSES como proveedores de identidad. La razón es que ambos entregan CUIT/CUIL, que es el mismo espacio de identificadores: la misma persona tiene el mismo número en ambos proveedores, lo que permite construir un `anon_seed` estable e independiente del proveedor usado en cada autenticación. Cualquier otro proveedor disponible en AUTENTICAR (ReNaPer, Mi Argentina, NIC.ar) usa identificadores en espacios distintos y rompe la unicidad.
  - Fórmula del `anon_seed`: `HASH(salt_del_sistema + CUIT/CUIL)`. El claim a extraer es `cuit`. No se usa `sub` (varía entre reinos) ni `preferred_username` (inestable entre proveedores).
  - Nivel mínimo requerido: nivel 2. El criterio es el nivel mínimo que garantice verificación de persona real con credenciales estatales activas.
  - La Decisión 4 de P-0013 (auditoría procedimental) fue temporal y queda supersedada por P-0014.
  - La auditoría de legitimidad en la plataforma participativa se resuelve en P-0014 mediante ZK sobre la cadena completa hasta el `anon_id`, según P-0015.

- **ADR P-0009 — Recepción y almacenamiento de la frase secreta en la plataforma.** Cerrado. Decisiones formalizadas:
  - El cliente calcula `HASH(frase_secreta)` localmente antes de enviar cualquier valor a la plataforma, tanto en el registro inicial del ciudadano en la plataforma como en cada login posterior. La plataforma nunca recibe la frase en texto plano.
  - La función hash del cliente debe ser una función criptográfica estándar (por ejemplo SHA-256), fija para todo el sistema, y forma parte de la especificación del protocolo entre cliente y plataforma.
  - La plataforma aplica Argon2id al `HASH(frase_secreta)` recibido, con salt único generado aleatoriamente por registro. Almacena el hash Argon2id, el salt y los parámetros de costo. Descarta el `HASH(frase_secreta)` inmediatamente.
  - Los parámetros de costo de Argon2id (memoria, iteraciones, paralelismo) son configurables por el operador y deben documentarse en la guía de instalación y operación.
  - La decisión de hashear en el cliente responde al mismo principio que P-0015 aplicó entre ciudadano y emisor: ningún componente servidor ve la frase en claro. Las dos decisiones son independientes pero coinciden por enfrentar la misma amenaza (exposición ante administradores locales, logs, infraestructura compartida).
