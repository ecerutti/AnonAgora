# Mapa de contexto

Este mapa permite que una conversación nueva cargue solo lo que su tarea necesita. Los índices resumen cada documento en una línea para decidir qué abrir **sin abrirlo**. Ante contradicciones entre este mapa y un documento, prevalece el documento; entre documentos, prevalecen los ADRs (regla en `AGENTS.md`).

## El proyecto en pocas líneas

El sistema se compone de una **capa de identidad anónima verificada** reutilizable y una **aplicación destino** construida sobre ella, una por despliegue (P-0021). La primera aplicación diseñada es una **plataforma de participación ciudadana**: un termómetro social no vinculante donde ciudadanos reales proponen y apoyan ideas sin exponer su identidad. La capa verifica que cada participante es una persona real (AUTENTICAR, solo ARCA/ANSES) y emite una identidad anónima persistente; la separación entre identidad real y actividad es criptográficamente forzada (`anon_seed` solo en el emisor, `anon_id` solo en la aplicación, nonce descartado, prueba ZK de legitimidad). No hay código todavía: el repositorio es documental. Las decisiones de diseño de plataforma están **cerradas** (P-0001 a P-0025); la etapa actual es el diseño específico de la demo.

## Ruteo por tipo de tarea

| Tipo de tarea | Leer primero | Profundizar solo si hace falta |
|---|---|---|
| Cualquier modificación del repo | `AGENTS.md` + `notas/estado_del_trabajo.md` | — |
| Diseño/desarrollo de la **demo** (etapa actual) | `notas/estado_del_trabajo.md`, `notas/recordatorios.md`, `demo/README.md`, DP-0001 | `contexto/sintesis_del_proyecto.md`, `demo/design/capa_de_identidad/simulacion_autenticar.md`, notas sueltas en `demo/` |
| Crear o modificar un **ADR** | `design/adr/README.md` (convención y criterios) + índice de ADRs de abajo → abrir solo los ADRs relacionados | `design/glosario.md` |
| **Capa de identidad** / criptografía / AUTENTICAR | Síntesis §3 → P-0013, P-0014, P-0015, P-0025 + `design/capa_de_identidad/README.md` (contrato) | `identity_model.md`, `docs/autenticar.md` (referencia OIDC completa), `docs/zk_jwt_investigacion.md` (solo para ZK profundo) |
| **Plataforma participativa** (reglas funcionales) | Síntesis §4 → los ADRs del tema según índice de abajo | `vinculacion_de_propuestas.md`, `docs/propuesta/02` |
| **UX / flujos / interfaces** | `notas/inventario_de_flujos_ui_ux.md` (10 flujos + huecos H-1..H-12) | P-0005, P-0010, P-0020 (modo debug) |
| **Seguridad / amenazas / operador** | `design/threat_model.md` + `design/modelo_operativo.md` | P-0006, P-0020, P-0024 |
| **Portal público / GitHub Pages** | `docs/index.md` + `docs/_config.yml` (tema jekyll-theme-slate; Pages sirve `docs/` y el sitio público es solo `index.md` + `propuesta/`). **Regla:** todo nuevo `.md` técnico en `docs/` (fuera de `propuesta/`) debe agregarse al `exclude:` del `_config.yml` — la exclusión es explícita por archivo, no genérica (el `include`/`exclude` con globs del Jekyll de Pages resultó no confiable). | `docs/propuesta/README.md` |
| Pregunta **conceptual** sobre la propuesta | `contexto/sintesis_del_proyecto.md` | `docs/propuesta/00` y `01` (narrativos, para público no técnico) |
| Duda de **vocabulario** | `design/glosario.md` | — |

## Índice de ADRs (una línea por decisión)

Estados: todos **Activos** salvo donde se indica. La carpeta indica el ámbito; el número nunca se reutiliza.

### Transversales — `design/adr/`

- **P-0006** — Modelo de amenazas intermedio con separación de funciones; no-objetivos declarados: colusión total, compromiso completo de infraestructura.
- **P-0020** — Logs: granularidad temporal mínima por tipo de evento, campos prohibidos por componente (`anon_id` nunca en emisor, `anon_seed` nunca en plataforma, IP en ninguno), modo debug visible con autodestrucción, retención 7 días gestionada por cada componente.
- **P-0021** — Arquitectura modular: capa de identidad + exactamente una aplicación destino por despliegue; aplicaciones futuras = despliegues independientes.
- **P-0022** — Fallos de servicios externos: reintentos server-to-server con backoff, mensajes al ciudadano por categoría, **emisión atómica respecto del cool-down** (se consume solo tras confirmación de entrega), fail-closed del revisor de lenguaje.
- **P-0024** — Sin perfil administrativo: el operador actúa solo por archivos de configuración y CLI sobre la infraestructura.
- **P-0025** — Unicidad de pseudónimos: consulta-con-reserva (TTL 10 min) del emisor a la aplicación destino + verificación atómica en el commit, ambas sobre la **forma normalizada** del pseudónimo; sufijo de letra se activa empíricamente al saturarse el espacio sin sufijo.

### Capa de identidad — `design/capa_de_identidad/adr/`

- **P-0002** — Pseudónimos amigables `Animal + Color/Adjetivo + Número [+ Letra]`, representación visible de la identidad anónima.
- **P-0003** — Pseudónimo generado por el sistema, regenerable antes de aceptar, permanente después; nunca de escritura libre.
- **P-0007** — Capa agnóstica del proveedor de verificación; el nivel de garantía de unicidad depende del proveedor disponible.
- **P-0013** — *(parcialmente supersedido por P-0014)* AUTENTICAR restringido a ARCA+ANSES (comparten espacio CUIT/CUIL); `anon_seed = HASH(salt_del_sistema + CUIT/CUIL)`; sin retención de token ni metadatos.
- **P-0014** — *(parcialmente supersedido por P-0015)* Auditoría ZK de legitimidad del emisor: Groth16/circom/snarkjs sobre el circuito RSA de zk-email-verify, proving en servidor; requiere JWKS histórico, trusted setup Phase 2 y auditoría del circuito.
- **P-0015** — El emisor almacena solo `{anon_seed, fecha_emision}` (fecha a granularidad de día); `anon_id = HASH(anon_seed + nonce)` con nonce descartado (separación criptográficamente irreversible); cool-down 6 meses configurable; ZK cubre solo el `anon_id`. Incluye la limitación aceptada de doble identidad post cool-down.

### Aplicación de participación ciudadana — `design/aplicaciones/participacion_ciudadana/adr/`

- **P-0001** — El pseudónimo es visible solo para el propio ciudadano; las propuestas no muestran autor (sin reputación pública).
- **P-0004** — Login = identidad anónima + frase secreta, con normalización del pseudónimo (mayúsculas, acentos, espacios, guiones).
- **P-0005** — Sesiones temporales con expiración total por inactividad (1 h); la interfaz nunca sugiere memoria entre sesiones.
- **P-0008** — Credencial = passphrase (mín. 4 palabras / 20 caracteres, configurable), sin recuperación alternativa.
- **P-0009** — El cliente normaliza la frase y envía `HASH(frase)` (nunca la frase); el servidor almacena Argon2id con salt único y parámetros configurables.
- **P-0010** — Ranking: `score = apoyos / (edad_días+1)^G` × multiplicadores 🔥 tendencia (MT=2.0) y 🌱 emergente (ME=1.5); "relevancia" normalizada 0-100; conteo real siempre visible.
- **P-0011** — Revisor de lenguaje con API de moderación de OpenAI (`omni-moderation-latest`): modera la forma, nunca el contenido ideológico; categorías sexual/self-harm desactivadas; normalización anti-ofuscación.
- **P-0012** — Apoyo binario retractable, sin voto negativo ni escalas; el autor nace como primer apoyo; conteo público.
- **P-0016** — Sin mecanismo de invalidación de identidades (ni a pedido ni administrativo): un mecanismo de invalidación sería un vector de DoS inauditables.
- **P-0017** — Cupo anual de propuestas configurable (default 2, 0 = sin límite), conteo por año móvil de 365 días; las derivadas consumen cupo.
- **P-0018** — Modelo de datos de propuestas: **sin autoría almacenada** (evento `{anon_id, fecha}` a granularidad de día en tabla separada, solo para el cupo), cuerpo Markdown (20.000 caracteres), sin imágenes, links como texto plano no clickeable.
- **P-0019** — Búsqueda full-text en español (morfología + stopwords) sobre título y cuerpo, más filtros estructurados combinables.
- **P-0023** — Sin moderación editorial humana; contenido inmutable tras publicar; retiro excepcional solo por causales legales (catálogo configurable) → tombstone que conserva solo el `id`; sin retiro a pedido del autor.

### Demo — `demo/design/capa_de_identidad/adr/`

- **DP-0001** — La demo simula AUTENTICAR (pantallas con banner "SIMULACIÓN") y omite ZK; ambas omisiones son del entorno demo, no del diseño.

## Índice de documentos no-ADR

### `docs/` — técnico y propuesta pública

- `docs/index.md` — portada del sitio GitHub Pages (único lugar, junto a `AGENTS.md`, donde aparece el nombre provisorio del repo).
- `docs/propuesta/00..06_*.md` — propuesta conceptual narrativa para no técnicos (00 resumen ejecutivo, 01 idea, 02 fundamentos, 03 historia de uso "María", 04 objeciones, 05 viabilidad, 06 arquitectura y desafíos). Licencia narrativa: ante conflicto prevalece `design/`, y las divergencias (p. ej. qué proveedores de verificación se mencionan) son deliberadas y **no deben reportarse como inconsistencias** (ver `docs/propuesta/README.md`).
- `docs/architecture_overview.md` — overview técnico del sistema: componentes, contrato, principios.
- `docs/autenticar.md` — referencia técnica completa de AUTENTICAR: realms, endpoints, JWT, claims, JWKS, flujos OIDC, errores.
- `docs/zk_jwt_investigacion.md` — investigación de ZK sobre JWT RS256: proyectos, librerías, trusted setup, riesgos, alternativas no-ZK descartadas.

### `design/` — diseño del sistema general

- `design/glosario.md` — vocabulario congelado del proyecto, por categorías, con referencias a ADRs.
- `design/threat_model.md` — actores, capacidades asumidas, objetivos y no-objetivos de seguridad.
- `design/modelo_operativo.md` — el rol del operador consolidado: capacidades, límites (imposibles por diseño vs. restricciones auditables), modelo de confianza.
- `design/capa_de_identidad/README.md` — **contrato capa↔aplicación destino**: qué se entrega, qué se garantiza, qué se prohíbe.
- `design/capa_de_identidad/identity_model.md` — modelo de identidad: real vs. anónima, unicidad, identificadores derivados, no-memoria entre sesiones.
- `design/capa_de_identidad/identity_wordlists.md` + `wordlists/*.txt` — generación de pseudónimos: curación, normalización, espacio de ≈1,5M → ≈33,7M con sufijo.
- `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md` — vínculos genéricos sin tipo, inmutables, grafo dirigido; derivadas = propuestas comunes con vínculos.

### `notas/` — material de trabajo (mantenido por el humano)

- `notas/estado_del_trabajo.md` — **leer antes de modificar el repo**: etapa actual, pendientes, últimas decisiones cerradas.
- `notas/recordatorios.md` — tareas diferidas por momento de activación (pre-demo, post-plataforma, siempre).
- `notas/anotaciones_e_ideas_de_trabajo.md` — ideas sin decisión asociada (ej.: exigir apoyo previo a crear propuesta; apoyo desde el listado).
- `notas/inventario_de_flujos_ui_ux.md` — 10 flujos UI/UX con bifurcaciones y errores + huecos H-1..H-12 (abiertos: H-1, H-3..H-7, H-12).
- `notas/propuesta_guia_de_instalacion.md` — material en gestación para la futura guía de instalación/operación; incluye semántica de cambio de parámetros en caliente.

### `demo/` e `implementation/`

- `demo/README.md` — regla de herencia: la demo hereda todo el diseño general; solo registra sus simplificaciones.
- `demo/design/capa_de_identidad/simulacion_autenticar.md` — pantallas simuladas de ANSES/ARCA con banner de simulación obligatorio.
- `demo/design/aplicaciones/participacion_ciudadana/i18n_y_a11y.md` — demo monolingüe es-AR, accesibilidad WCAG nivel A.
- `demo/design/aplicaciones/participacion_ciudadana/telemetria.md` — la demo no tiene analytics ni telemetría.
- `demo/implementation/aplicaciones/participacion_ciudadana/identificador_propuestas.md` — IDs de propuesta: Crockford Base32, 4 caracteres, prefijo `#` (ej. `#8KF2`).
- `implementation/` — vacía de contenido por ahora (solo READMEs de estructura; ADRs futuros `I-XXXX`).

## Mantenimiento de este mapa

Ver reglas completas en `contexto/README.md`. En corto: cada ADR nuevo o supersedido actualiza su línea acá; cada documento agregado/movido actualiza el índice; la tabla de ruteo se ajusta cuando aparece un tipo de tarea nuevo (p. ej., cuando exista código de la demo).
