# Síntesis del proyecto

**Última actualización: 2026-06-09.** Este documento destila la comprensión profunda del proyecto a partir de la lectura completa del repositorio. Condensa y conecta; no reemplaza a las fuentes. Ante cualquier contradicción, prevalecen los ADRs y documentos de `design/`. El estado del trabajo vigente vive en `notas/estado_del_trabajo.md`.

## 1. Propósito e idea central

Las democracias tienen pocas herramientas para escuchar de forma continua qué piensa la sociedad entre elecciones. Las redes sociales exponen y polarizan; las encuestas son fotos puntuales. El proyecto explora una solución técnica al dilema estructural de la participación digital: con anonimato total el sistema se degrada (trolls, bots, cuentas múltiples); con identidad real la gente se autocensura por miedo a represalias.

La respuesta es combinar **verificación de identidad + anonimato persistente**: el ciudadano demuestra una sola vez que es una persona real (analogía del guardia que pide el DNI en la puerta) y a partir de ahí actúa con una identidad anónima estable que nadie —ni el gobierno, ni los administradores, ni los desarrolladores— puede vincular con su identidad real. El resultado buscado es un **termómetro social**: observar qué propuestas reúnen apoyo genuino y cómo evolucionan las opiniones, sin tomar decisiones vinculantes ni reemplazar a las instituciones. Las ideas se evalúan por su contenido, no por quién las enuncia.

El objetivo final del proyecto es doble: (a) una **demo funcional** que muestre la experiencia (en construcción conceptual; seguimiento en anonagora.cloud), y (b) un diseño completo, auditable y publicado bajo Apache 2.0 que cualquier institución pueda desplegar. El proyecto deliberadamente **no tiene nombre de producto**; el nombre del repositorio es provisorio y no se usa en la documentación.

## 2. Arquitectura: capa de identidad + aplicación destino

P-0021 estructura el sistema en dos partes, **una capa de identidad y exactamente una aplicación destino por despliegue**:

- La **capa de identidad** (reutilizable) verifica personas reales y emite identidades anónimas. Componentes: emisor, integración con el verificador externo, proving ZK, JWKS histórico.
- La **aplicación destino** define qué hace el ciudadano con su identidad. La única diseñada es la **plataforma de participación ciudadana**; otras (p. ej. denuncias anónimas) serían despliegues independientes, con registro separado e identidades no vinculables entre sí — esto es deliberado: respeta la elección del ciudadano y evita correlación cruzada.

El **contrato capa↔aplicación** (`design/capa_de_identidad/README.md`): en la emisión —una única vez— el emisor entrega `{pseudónimo, anon_id, prueba ZK}`; después la aplicación opera con total independencia. Durante la emisión existen además dos operaciones de protocolo (consulta-con-reserva y liberación de pseudónimos, P-0025) y una confirmación de entrega que es la que consume el cool-down (P-0022).

## 3. La capa de identidad: mecanismos y porqués

La cadena de identificadores es el corazón del diseño:

- `anon_seed = HASH(salt_del_sistema + CUIT/CUIL)` — determinista, vive **solo en el emisor**, sirve para unicidad y cool-down. El salt protege contra diccionario sobre el espacio finito de CUITs.
- `anon_id = HASH(anon_seed + nonce)` — el nonce se genera por emisión y **se descarta de inmediato**. Esto convierte la separación entre emisor y aplicación en una propiedad criptográfica, no organizacional: ni con acceso total al emisor se puede recalcular qué `anon_id` corresponde a qué ciudadano. Ningún componente tiene ambos identificadores; el emisor persiste únicamente `{anon_seed, fecha_emision}` (P-0015).
- La **prueba ZK** (P-0014: Groth16, circom/snarkjs, circuito RSA auditado de zk-email-verify, proving en servidor ~15-45 s) certifica "existe un JWT válido de AUTENTICAR cuyo CUIT deriva en este `anon_id`" sin revelar nada más. Su función es la **integridad verificable**: un operador malicioso no puede fabricar identidades sin tokens reales, y cualquier auditor lo verifica con el JWKS público. Requisitos pendientes para producción: JWKS histórico, ceremonia de trusted setup Phase 2, auditoría del circuito adaptado (USD 30-150k).
- **Verificación**: AUTENTICAR (OIDC/Keycloak estatal), restringido a **ARCA y ANSES** porque CUIT y CUIL son el mismo número para personas físicas — único modo de que el `anon_seed` sea estable entre proveedores (P-0013). Nivel mínimo 2. El token se descarta sin retener metadatos.
- **Pseudónimo** `Animal + Color/Adjetivo + Número [+ Letra]` (P-0002/P-0003): generado por el sistema, regenerable antes de aceptar, permanente después, visible solo para el propio ciudadano. Espacio: 1,86M sin sufijo → 39M con sufijo de letra, que se activa empíricamente al saturarse el espacio (P-0025). La unicidad importa porque el pseudónimo es identificador de login.
- **Cool-down** de 6 meses entre emisiones (P-0015): es EL mecanismo anti-abuso. Limitación aceptada: quien conserva sus credenciales y simula pérdida puede operar dos identidades tras el cool-down; el costo temporal lo desincentiva pero no lo elimina (documentado en P-0015).

## 4. La plataforma participativa: reglas funcionales y porqués

- **Acceso**: login = pseudónimo (normalizado: mayúsculas/acentos/espacios/guiones) + frase secreta (P-0004). La credencial es una **passphrase** memorable sin recuperación posible (P-0008); el cliente envía solo su hash y el servidor almacena Argon2id (P-0009). Sesiones que expiran por completo (1 h) sin dejar rastro visible: la interfaz jamás sugiere que recuerda al ciudadano (P-0005).
- **Apoyo binario retractable, sin voto negativo** (P-0012): se mide qué ideas atraen interés, no qué ideas generan rechazo; elimina voto bronca y campañas de descalificación. El autor nace como primer apoyo.
- **Cupo anual de propuestas** (default 2, año móvil de 365 días, P-0017): costo de oportunidad contra spam y ruido político; las derivadas consumen cupo.
- **Inmutabilidad + propuestas derivadas** (P-0023, P-0018): el contenido publicado no se edita jamás (los apoyos siempre refieren a un texto fijo); las ideas evolucionan creando derivadas con **vínculos** genéricos, inmutables, en grafo dirigido.
- **Sin autoría almacenada** (P-0018): la propuesta no guarda referencia al autor; solo existe un evento `{anon_id, fecha}` en tabla separada para el cupo. Nadie —ni el operador— puede listar las propuestas de un ciudadano.
- **Ranking con decaimiento temporal** (P-0010): termómetro del presente, no del pasado. Señales 🔥 tendencia y 🌱 emergente con umbrales percentiles adaptativos.
- **Revisor de lenguaje** (P-0011): API de moderación de OpenAI; modera la **forma** (agresión, odio, violencia, ilegalidad), nunca el contenido ideológico. Fail-closed si el servicio cae (P-0022).
- **Sin moderación editorial ni invalidación de identidades** (P-0023, P-0016): no hay perfiles con poder sobre el contenido ni mecanismo para desactivar identidades (sería un vector de DoS inauditables). Única excepción: **retiro por causales legales** → tombstone que conserva solo el `id`.
- **Sin perfil administrativo** (P-0024): el operador configura por archivos y CLI; no se agrega ninguna credencial privilegiada expuesta en red.

## 5. Modelo de amenazas y confianza

P-0006 adopta un **modelo intermedio**: componentes individuales pueden ser comprometidos y los operadores no se asumen benévolos, pero no se pretende resistir colusión total ni compromiso completo de infraestructura (no-objetivos declarados; nunca prometer "anonimato absoluto" en documentación pública). Principios derivados: separación de funciones, minimización de datos por componente, retención limitada de metadatos (P-0020: timestamps recortados por tipo de evento, campos prohibidos por componente, logs a 7 días), auditabilidad.

El modelo de confianza hacia el operador (`design/modelo_operativo.md`) distingue dos clases de límite: lo **imposible por diseño** (identificar autores, revocar identidades, vincular `anon_seed`↔`anon_id` — falta el dato o el mecanismo, vale incluso contra un operador malicioso) y lo **prohibido pero detectable** (moderación encubierta, manipulación de datos — protegido por auditabilidad, no por imposibilidad). La privacidad se fortalece cuanto más separados están el operador de la capa y el de la aplicación; el diseño soporta todo el continuo.

## 6. Hoja de ruta y estado

Fases del proyecto: propuesta conceptual (✅ publicada en `docs/propuesta/` + GitHub Pages) → diseño de plataforma (✅ P-0001..P-0025 cerrados) → **diseño específico de la demo (etapa actual)** → desarrollo de la demo (código vivirá en `demo/src/`) → eventualmente guía de instalación/operación y camino a despliegues reales.

Pendientes de la etapa actual (detalle vigente en `notas/estado_del_trabajo.md` y `notas/recordatorios.md`): alcance funcional del MVP, stack tecnológico, persistencia/deployment, seed de datos, look & feel. La demo simulará AUTENTICAR y omitirá ZK (DP-0001), sin i18n ni telemetría. Para la fase de interfaces existe el inventario de 10 flujos UI/UX con sus huecos abiertos (H-1, H-3..H-7, H-12) en `notas/inventario_de_flujos_ui_ux.md`. Quedan también ADRs diferidos identificados: contrato concreto de entrega emisor↔aplicación, manejo de sesión en redacciones largas, términos y condiciones, formalización del onboarding (H-2).

## 7. Convenciones operativas del repositorio

Resumen mínimo (las reglas completas y la estructura de carpetas viven en `AGENTS.md`): el repo es la fuente de verdad; los ADRs prevalecen y son inmutables salvo su campo "Estado"; cuatro prefijos de ADR (`P` diseño general, `I` implementación general, `DP` diseño demo, `DI` implementación demo) con numeración única por prefijo; la demo hereda todo y solo registra simplificaciones; `notas/` es material de trabajo del humano; los documentos se escriben en castellano con el vocabulario de `design/glosario.md`.
