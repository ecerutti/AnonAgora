# Estado del trabajo

Este documento refleja el estado actual del trabajo de diseño y desarrollo del proyecto. Es la referencia de entrada para cualquier conversación nueva con agentes: permite reconstruir rápidamente en qué etapa estamos, qué decisiones están cerradas, qué queda pendiente y qué cosas deben revisarse más adelante.

Se mantiene con apoyo de los agentes pero la autoría final es del humano.

## Etapa actual

Diseño de la plataforma: cierre de decisiones de diseño previas al desarrollo de la demo con Claude Code. El objetivo de esta etapa es dejar cerradas todas las decisiones importantes para que el desarrollo posterior necesite preguntar lo menos posible.

## Decisiones de diseño pendientes

Orden sugerido de abordaje:

1. **Límite anual de propuestas por ciudadano.** Decidir la existencia del límite como propiedad del sistema (configurable, cómo se cuenta el año, qué pasa con propuestas derivadas, posibilidad de configurar 0 o desactivarlo). El valor concreto "2 por año" que aparece en la documentación conceptual es el default sugerido, no la decisión de fondo.

2. **Modelo de datos de propuestas.** Qué campos tiene una propuesta, longitud máxima, formato del cuerpo, si admite imágenes o links.

3. **Búsqueda de propuestas.** Tipo de búsqueda (texto plano, tags, categorías), si hay categorización temática.

4. **Vinculación entre propuestas.** Tipos de vínculo (deriva de, mejora a, integra a), direccionalidad, si requieren aceptación del autor original.

5. **Política de logs y retención de metadatos.** Concretar lo que P-0006 exige pero no fija: qué se registra, con qué granularidad temporal, por cuánto tiempo.

6. **Corrección de P-0009 por cambio de flujo de frase secreta.** P-0015 cambió el flujo: la plataforma nunca recibe la frase secreta en texto plano sino un verificador calculado por el emisor. P-0009 debe ser corregido formalmente mediante un nuevo ADR que documente este cambio y su justificación.

7. **Invalidación de anon_id en la plataforma participativa.** La decisión de si la plataforma puede marcar `anon_ids` como inactivos quedó fuera del alcance de P-0015. Corresponde resolverla en el ADR del modelo de datos de la plataforma participativa.

## Decisiones de diseño específicas de la demo

Estas se abordan después de cerrar las decisiones de plataforma. Incluye una decisión pendiente surgida durante el diseño de P-0014:

1. Stack tecnológico (lenguaje, framework, base de datos).
2. Alcance funcional del MVP (qué features entran y cuáles no).
3. Persistencia y deployment.
4. Datos de prueba o seed inicial.
5. Manejo de la identidad anónima en la demo (simulación de la verificación de unicidad). La demo no usa AUTENTICAR real ni implementa ZK; ambas omisiones deben quedar documentadas en un ADR de demo (D-XXXX) que explique por qué no aplican en ese contexto.
6. Revisor de lenguaje en la demo: ¿API real de OpenAI o mock local?
7. Autenticación administrativa para gestión técnica de la demo.
8. Look and feel, guía visual.

## Recordatorios activos

Los recordatorios accionables viven en `notas/recordatorios.md`.

## Material de trabajo en gestación

- `notas/propuesta_guia_de_instalacion.md` — borrador preliminar con ideas sobre qué debería contener una futura guía de instalación y operación para administradores.

## Últimas decisiones cerradas

- **ADR P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas.** Cerrado. Decisiones formalizadas:
  - Cool-down de 6 meses configurable por el operador como mecanismo de control de abuso. Contado desde `fecha_emision` de la identidad activa. Sin límite de renovaciones de por vida.
  - El emisor almacena únicamente `{anon_seed, fecha_emision}`. No almacena `anon_id` ni ninguna asociación entre ambos.
  - El `anon_id` se deriva como `HASH(anon_seed + HASH(frase_secreta))`. La separación entre `anon_seed` y `anon_id` es criptográficamente forzada por el secreto del ciudadano.
  - El flujo de emisión ocurre en dos fases: primero el emisor verifica unicidad y cool-down usando solo el `anon_seed`; recién entonces el ciudadano envía `HASH(frase_secreta)` para que el emisor calcule el `anon_id` y genere la prueba ZK.
  - ZK cubre únicamente el `anon_id`. La certificación del `anon_seed` mediante ZK queda descartada. P-0015 superseda parcialmente P-0014 en ese punto.
  - La infraestructura de JWKS histórico y revocación por `kid` comprometido de P-0014 se mantienen vigentes.
  - La pérdida de credenciales (pseudónimo, frase secreta o ambos) hace la identidad irrecuperable. El ciudadano espera el cool-down y solicita una nueva identidad completa. Este comportamiento refuerza la percepción de anonimato.
  - La plataforma no puede invalidar `anon_ids`. No existe revocación técnica. Esa decisión queda fuera del alcance del emisor y corresponde al modelo de datos de la plataforma participativa.
  - El historial de identidades inactivas permanece visible y sigue contando. El sistema no distingue identidades abandonadas de inactivas.
  - P-0009 asumió que la plataforma recibe la frase secreta en texto plano. Con P-0015 ese flujo cambia: la plataforma recibe el verificador ya calculado por el emisor y nunca ve la frase. Corrección formal pendiente en ADR posterior.

- **ADR P-0014 — Auditoría criptográfica de legitimidad del emisor mediante pruebas de conocimiento cero.** Cerrado. Decisiones formalizadas:
  - Se adopta ZK como mecanismo de auditoría de legitimidad del emisor. Superseda P-0013 Decisión 4 (que era temporal y procedimental).
  - La generación de la prueba se realiza en el servidor (emisor), no en el cliente. La generación en cliente fue descartada por tiempos de proving de 60-180 segundos en dispositivos de gama media-baja y requisitos de RAM incompatibles con smartphones del segmento objetivo.
  - Stack adoptado: circom + snarkjs con el circuito RSA de `zkemail/zk-email-verify` (auditado) como base. Esquema Groth16 sobre BN254. Pruebas de ~256 bytes, verificación en 1-5 ms en servidor.
  - La adaptación del circuito (extracción del claim `cuit`, cálculo de `anon_seed`, separación de dominios en hash Poseidon) no está auditada y requiere auditoría especializada en ZK antes de cualquier despliegue en producción. Costo estimado: USD 30.000–150.000.
  - `zkemail/zk-jwt` no debe usarse como dependencia directa (autodeclarado no apto para producción, sin auditoría). Puede consultarse como referencia de implementación.
  - El emisor debe implementar un servicio de JWKS histórico para mantener verificabilidad de pruebas tras rotaciones de clave de AUTENTICAR.
  - El emisor debe implementar un mecanismo de revocación por `kid` comprometido.
  - Requiere ceremonia de trusted setup Phase 2 antes del despliegue en producción. Phase 1 puede reutilizarse de la ceremonia pública de Hermez/Polygon.
  - La prueba ZK certifica legitimidad del `anon_seed`. La auditoría de legitimidad en la plataforma participativa (que trabaja con `anon_id`) es un problema separado que se resuelve en P-0015.
  - `design/adr/README.md` tiene un agregado pendiente de aplicar: una regla explícita sobre separación entre ADRs de plataforma y decisiones de implementaciones específicas.

- **ADR P-0013 — Integración con AUTENTICAR.** Cerrado. Decisiones formalizadas:
  - Se usa AUTENTICAR como proveedor de verificación de identidad.
  - Se aceptan únicamente ARCA y ANSES como proveedores de identidad. La razón es que ambos entregan CUIT/CUIL, que es el mismo espacio de identificadores: la misma persona tiene el mismo número en ambos proveedores, lo que permite construir un `anon_seed` estable e independiente del proveedor usado en cada autenticación. Cualquier otro proveedor disponible en AUTENTICAR (ReNaPer, Mi Argentina, NIC.ar) usa identificadores en espacios distintos y rompe la unicidad.
  - Fórmula del `anon_seed`: `HASH(salt_del_sistema + CUIT/CUIL)`. El claim a extraer es `cuit`. No se usa `sub` (varía entre reinos) ni `preferred_username` (inestable entre proveedores).
  - Nivel mínimo requerido: nivel 2. El criterio es el nivel mínimo que garantice verificación de persona real con credenciales estatales activas.
  - La Decisión 4 de P-0013 (auditoría procedimental) fue temporal y queda supersedada por P-0014.
  - La auditoría de legitimidad en la plataforma participativa se resuelve en P-0015 mediante firma del emisor sobre cada `anon_id`.
  - `notas/autenticacion_autenticar.md` fue migrado y debe eliminarse: la parte descriptiva migró a `docs/autenticar.md` y las decisiones al ADR.
  - `docs/autenticar.md` está pendiente de completar con información de implementación (estructura real del JWT, ejemplos de requests/responses, manejo de errores, scopes, ambiente de testing) mediante investigación con el plugin de Chrome.
