# Pruebas de Conocimiento Cero (ZK) sobre JWT RS256
## Documento de referencia técnica para arquitectura del sistema de participación ciudadana

**Fecha de investigación:** Abril 2026
**Estado:** Borrador para decisión arquitectónica

---

## Resumen Ejecutivo

La aplicación de ZK sobre tokens JWT RS256 es técnicamente viable y existe infraestructura de producción que la implementa. El proyecto de mayor madurez es **zkLogin de Sui Network** (en producción desde 2023), que resuelve exactamente el problema descrito: probar que un JWT RS256 firmado por una clave pública conocida contiene un claim específico, sin revelar el claim ni el token. Usa Groth16 sobre BN254 con circom, requiere trusted setup (realizado con 111 participantes), y genera pruebas en un servidor dedicado en ~1-3 segundos.

El segundo proyecto relevante es **zkemail/zk-jwt** (beta, activo al Oct 2025), que porta la misma infraestructura de circuitos RSA-en-circom de ZK Email al dominio JWT. No tiene auditoría de seguridad y declara explícitamente no estar listo para producción.

**Recomendación principal:** Para el contexto del proyecto (plataforma estatal argentina, ciudadanos desde múltiples dispositivos, sin blockchain), la arquitectura viable es generación de prueba **en servidor** usando la pila circom/snarkjs o gnark, con el circuito de zkEmail/zk-jwt como base. La generación en cliente (navegador) es técnicamente posible pero produce tiempos de 30-120 segundos en hardware de gama media-baja, lo cual es inaceptable para UX. El trusted setup (Groth16) es el paso más delicado operativamente y debe planificarse como ceremonia MPC antes del lanzamiento. Como alternativa no-ZK con menor complejidad, el esquema de **firma ciega (blind signature) sobre el anon_seed** merece evaluación detallada.

---

## 1. Estado del Arte y Proyectos Existentes

### 1.1 zkLogin (Sui Network / MystenLabs)

**URL:** https://docs.sui.io/concepts/cryptography/zklogin
**Repositorios públicos:** https://github.com/MystenLabs/zklogin-verifier (Rust, 26 stars, activo hasta Jul 2025)

zkLogin es el proyecto más maduro que implementa ZK sobre JWT RS256 en producción. Opera desde el lanzamiento de Sui Mainnet en 2023. El circuito está implementado en **circom** y usa el esquema **Groth16** sobre la curva **BN254**. La decisión de usar Groth16 se justifica en la documentación oficial: es el sistema zkSNARK de propósito general más eficiente en términos de tamaño de prueba y velocidad de verificación.

El sistema fue auditado por dos firmas especializadas en ZK (zkSecurity realizó la auditoría del circuito). La ceremonia de trusted setup Phase 2 tuvo 111 contribuyentes (82 desde browser, 29 desde Docker) y se realizó entre el 12 y 18 de septiembre de 2023.

**Estado de madurez:** Producción. Se usa con Google, Facebook, Apple, Twitch, AWS y otros proveedores OIDC en Mainnet de Sui.

**Particularidad relevante para el proyecto:** zkLogin usa la clave pública del JWT como entrada pública al circuito. El kid (key ID) del JWT se revela como output público para que el verificador pueda asociar la prueba con la JWK correcta del JWKS público. El campo sub (o cualquier claim sensible) es entrada privada. La prueba demuestra que: (1) la firma RSA es válida contra la clave pública correspondiente al kid publicado, (2) el valor del claim clave coincide con el input del circuito, y (3) el address derivado es consistente con el claim y el salt.

**Limitación crítica para adopción directa:** El circuito de zkLogin está diseñado para Sui y no es un módulo reutilizable standalone. Incluye lógica específica de Sui (épocas, nonces con llaves efímeras, derivación de addresses). Sin embargo, la infraestructura de circuitos RSA subyacente (de zkemail) sí es reutilizable.

### 1.2 zkemail/zk-jwt (ZK Email / jwt-tx-builder)

**URL:** https://github.com/zkemail/zk-jwt
**Licencia:** MIT (circuits: GPL via circom/circomlib)
**Lenguaje:** TypeScript + Circom
**Último commit:** 7 de octubre de 2025
**Stars:** 22
**Estado declarado:** No auditado. No apto para producción.

Este proyecto porta la infraestructura de circuitos de zkemail/zk-email-verify al dominio JWT. Tiene tres paquetes principales: `@zk-email/jwt-tx-builder-circuits` (circuitos circom para verificación RSA + extracción de claims), `@zk-email/jwt-tx-builder-helpers` (TypeScript para generar inputs al circuito), y `@zk-email/jwt-tx-builder-contracts` (contratos Solidity para verificación on-chain).

El circuito JWT Auth acepta como parámetros `n=121` (bits por chunk del modulus RSA) y `k=17` (número de chunks), cuyo producto n×k = 2057 > 2048, cubriendo RSA-2048. Esto es directamente aplicable al JWKS de AUTENTICAR. Los inputs privados son el mensaje JWT (header.payload) y la firma RSA. Los inputs públicos incluyen la clave pública RSA y varios índices de posición dentro del JWT.

Los outputs del circuito incluyen `publicKeyHash` (hash Poseidon de la clave pública RSA), `jwtNullifier` (nullifier único por JWT basado en la firma), y claims específicos extraídos. El `jwtNullifier` es especialmente relevante: permite detectar reuso del mismo token sin revelar la firma.

**Adopción en producción:** No se encontró evidencia de adopción en producción. Existe una demo en jwt.zk.email (no accesible directamente para verificación).

### 1.3 zk-email-verify (base tecnológica compartida)

**URL:** https://github.com/zkemail/zk-email-verify
**Licencia:** MIT (circuits: GPL)
**Última release:** v6.4.2 (12 de junio de 2025)
**Auditorías:** zkSecurity (cubierto en v6.1.0) y yAcademy (mayo 2024)

Este es el proyecto base que contiene los circuitos RSA en circom reutilizados por zk-jwt. La librería `lib/rsa.circom` implementa `RSAVerifier65537` con parámetros n y k configurables. Para RSA-2048 con n=121, k=17: el circuito tiene aproximadamente **1.5 millones de constraints**. Para RSA-4096 con los mismos chunks se necesitaría aumentar k a 34, duplicando el tamaño del circuito y el tiempo de prueba.

Tiene adopción real: Proof of Twitter (https://twitter.zk.email/), ZKP2P (on/off-ramp DeFi), zkEmail Safe (multisigs operados por email), y Email Wallet.

### 1.4 kipz/zkp-jwt

**URL:** https://github.com/kipz/zkp-jwt
**Licencia:** MIT
**Lenguaje:** Go (gnark + PLONK)
**Último commit:** 28 de noviembre de 2025
**Stars:** 2

Implementa ZK sobre JWT pero **solo para ES256 (ECDSA P-256)**, no RS256. No es aplicable directamente al caso de AUTENTICAR. Usa gnark con PLONK y KZG commitments. El enfoque de PLONK elimina la necesidad de un trusted setup por circuito. Incluye soporte de batching de múltiples JWTs en una prueba. El tamaño de prueba es constante independientemente del número de JWTs en el batch.

### 1.5 2dvorak/zk-jwt-poc

**URL:** https://github.com/2dvorak/zk-jwt-poc
**Lenguaje:** Circom + JavaScript (snarkjs)
**Último commit:** 27 de octubre de 2023
**Stars:** 2
**Estado:** PoC académico, abandonado

Implementa un esquema simplificado que **no verifica la firma RSA del JWT dentro del circuito**. En cambio, verifica el hash del JWT completo y prueba la inclusión de claims. Esto es criptográficamente insuficiente para el caso de uso: no demuestra que el JWT fue firmado por AUTENTICAR. Los benchmarks de este PoC (prove time: ~5 segundos en Node.js) deben entenderse en ese contexto de circuito más simple.

### 1.6 RISC Zero jwt-validator

**URL:** https://github.com/risc0/risc0/tree/main/examples/jwt-validator
**Licencia:** Apache 2.0 / MIT
**Estado:** Ejemplo oficial, actualizado enero 2026

Este ejemplo demuestra verificación ZK de JWT RS256 usando la zkVM de RISC Zero, que ejecuta código Rust arbitrario en un entorno STARK. El guest (código verificado) usa el crate `jwt-compact` para verificar la firma RS256 del JWT contra una clave pública, y registra el claim `subject` en el journal como output público. El verificador puede confirmar que el código de verificación fue ejecutado correctamente sin acceso al JWT.

**Diferencia clave respecto a circom:** RISC Zero no requiere circuito aritmético custom. Cualquier código Rust que verifique el JWT es suficiente. Esto incluye extracción de claims arbitrarios.

### 1.7 Tabla comparativa de proyectos

| Proyecto | Esquema ZK | RS256 | Estado | Última actividad | Auditado | Producción |
|---|---|---|---|---|---|---|
| zkLogin (Sui) | Groth16/BN254/circom | Sí | Producción | Jul 2025 (verifier) | Sí (zkSecurity) | Sí (Sui Mainnet) |
| zkemail/zk-jwt | Groth16/BN254/circom | Sí | Beta | Oct 2025 | No | No |
| zk-email-verify | Groth16/BN254/circom | Sí | Producción | Jun 2025 | Sí (2 firmas) | Sí (varias apps) |
| kipz/zkp-jwt | PLONK/KZG/gnark | No (solo ES256) | Alpha | Nov 2025 | No | No |
| 2dvorak/zk-jwt-poc | Groth16/circom | No (hash only) | PoC abandonado | Oct 2023 | No | No |
| RISC Zero jwt-validator | STARK/Groth16 (recursivo) | Sí | Ejemplo oficial | Ene 2026 | Parcial (protocolo) | No (ejemplo) |

---

## 2. Compatibilidad con el Problema Concreto

### 2.1 ¿Algún proyecto resuelve exactamente el caso?

El caso exacto requerido es: **"existe un JWT RS256 firmado por AUTENTICAR cuyo claim `cuit` produce este `anon_seed`, sin revelar el CUIT ni el token"**.

Ningún proyecto resuelve exactamente este caso de forma lista para usar. Sin embargo, **zkemail/zk-jwt es el punto de partida más próximo** y requiere adaptación. El circuito JWT Auth de ese proyecto verifica RSA-2048 y extrae claims arbitrarios. La adaptación consistiría en: (1) configurar el claim a extraer como `cuit` en lugar de `email` o `sub`, (2) hacer que el claim extraído sea entrada privada y que el anon_seed = HASH(salt + cuit) sea la salida pública, (3) eliminar la lógica de `accountCode`, `azp` y `command` que no aplica.

Alternativamente, usando RISC Zero, el "circuito" es código Rust que verifica el JWT RS256, extrae el CUIT, calcula el anon_seed y lo escribe al journal. No requiere implementar aritmética de campo manual.

### 2.2 Manejo de la clave pública del emisor (JWKS de AUTENTICAR)

En los circuitos basados en circom (zkLogin, zk-jwt), la clave pública RSA (modulus n) es un **input público al circuito**. La prueba es válida para exactamente la clave pública que se declara. El verificador de la prueba recibe como inputs públicos: el hash de la clave pública, el kid, y el anon_seed. Para verificar, consulta el JWKS de AUTENTICAR, obtiene la clave correspondiente al kid, calcula su hash, y verifica que coincide con el hash declarado en la prueba. Así, la prueba es verificable usando solo el JWKS público de AUTENTICAR.

En zkLogin, la clave pública se integra al circuito de una manera particular: el hash Poseidon de la clave pública (dividida en chunks) es un output del circuito. El verificador computa ese mismo hash a partir de la JWK pública y verifica que coincide.

**Implicancia para rotación de clave:** Ver sección 7.4.

### 2.3 Verificabilidad pública

Las pruebas Groth16 generadas por circom/snarkjs son verificables con solo la `verification_key.json` (derivada del trusted setup y el circuito) y los inputs públicos. La `verification_key.json` es un archivo de ~2KB que puede publicarse. Cualquier tercero puede verificar una prueba con snarkjs en JavaScript o con cualquier implementación de verificador Groth16 (bellman en Rust, gnark en Go, etc.) sin acceso al token ni al sistema que lo emitió. La verificación es una operación de emparejamientos de curva elíptica, computacionalmente barata.

### 2.4 RSA-2048 vs RSA-4096

Para RSA-2048 con parámetros n=121, k=17 (recomendados por zkemail): el producto n×k=2057, suficiente para representar el modulus de 2048 bits. Este es el tamaño estándar de JWT RS256 y es el soportado por zkemail/zk-jwt y zkLogin.

Para RSA-4096: requiere k=34 (con n=121), duplicando el número de chunks y aproximadamente duplicando el número de constraints del sub-circuito RSA (el componente más pesado). El tiempo de prueba y el tamaño del proving key aumentan proporcionalmente. No existe evidencia pública de implementaciones de RSA-4096 en circom en uso real. Si AUTENTICAR usa claves de 4096 bits en el futuro, el costo computacional sería aproximadamente el doble.

**Verificar el tamaño de clave actual de AUTENTICAR:** Consultar el JWKS endpoint de AUTENTICAR para confirmar el tamaño del modulus en producción. Los JWTs RS256 de Keycloak típicamente usan 2048 bits.

---

## 3. Dónde se Genera la Prueba: Cliente vs. Servidor

Este es el punto de mayor impacto en el diseño del sistema, especialmente dado el requisito de multi-dispositivo y el perfil de usuarios (ciudadanos argentinos con smartphones de gama baja).

### 3.1 Generación en cliente (navegador o app móvil)

**Viabilidad técnica:** Sí, es técnicamente posible hoy. snarkjs (v0.7.6, enero 2026) soporta generación de pruebas Groth16 y PLONK completamente en browser vía WebAssembly. El código circom compilado genera un archivo `.wasm` para cálculo del witness, y snarkjs realiza el proving sobre él. La demo de zkLogin del navegador y la ceremonia de trusted setup (donde los contribuyentes podían participar desde el browser) son prueba de viabilidad.

**Librerías disponibles para cliente:**
- `snarkjs` (npm, GPL-3, v0.7.6): funciona en Node.js y browser. Usa worker threads para paralelización. Para entornos sin worker threads (algunas extensiones, Bun) existe modo `singleThread: true`.
- `@zk-email/jwt-tx-builder-helpers`: TypeScript, genera inputs para el circuito JWT Auth.

**Tiempo de generación en dispositivos de gama media-baja:** No existen benchmarks públicos oficiales específicos para el circuito JWT Auth sobre RS256-2048 en browsers móviles de gama baja. Sin embargo, a partir de datos disponibles en el ecosistema ZK Email y zkLogin, se puede estimar:

El circuito zk-email-verify con RSA-2048 y payload de ~1KB tiene aproximadamente 1.5 millones de constraints. En un laptop moderno (M1 MacBook, 2022), el proving Groth16 con snarkjs tarda ~15-30 segundos. En un smartphone Android de gama media-baja (Snapdragon 660, 4GB RAM, 3-5 años de antigüedad), los tiempos estimados son **60-180 segundos** para un circuito de ese tamaño, basados en la observación general de que dispositivos de gama baja son 4-6x más lentos que laptops en operaciones de álgebra lineal intensiva.

El PoC 2dvorak/zk-jwt-poc (circuito más simple, sin verificación RSA completa) reportó `prove time: 4977ms` en Node.js para su circuito simplificado. El circuito con RSA completo es significativamente más grande.

**Memoria RAM requerida:** El archivo `.zkey` (proving key) para un circuito de 1.5M constraints pesa varios cientos de MB. snarkjs lo carga en memoria durante el proving. Esto puede causar OOM en dispositivos con menos de 3-4GB de RAM libre, lo cual es común en smartphones de gama baja con Android.

**Modo offline:** El cálculo del witness (con el `.wasm`) y el proving (con el `.zkey`) pueden ejecutarse completamente offline una vez que los archivos están descargados. Sin embargo, el `.zkey` pesa cientos de MB, lo que lo hace impractical para distribución vía browser sin caché agresiva.

**Demos públicos ejecutables:** jwt.zk.email (no accesible directamente en este entorno). zkLogin provee una demo funcional en browser en Sui Devnet/Testnet. La ceremonia de trusted setup de zkLogin se ejecutó en browser usando snarkjs, lo que constituye evidencia de funcionalidad browser-side para operaciones de esa envergadura.

### 3.2 Generación en servidor (emisor)

**Librerías server-side disponibles:**

| Librería | Lenguaje | Licencia | Versión | Estado | Soporte RSA |
|---|---|---|---|---|---|
| snarkjs | JavaScript/Node.js | GPL-3 | v0.7.6 (ene 2026) | Activo | Sí (via circom) |
| bellman | Rust | MIT | v0.14.x | Activo (Zcash) | Manual |
| gnark | Go | Apache 2.0 | v0.14.0 (ago 2025) | Activo, producción | No nativo (solo ECDSA/EdDSA en std/) |
| arkworks | Rust | MIT/Apache | v0.4.x | Activo | Manual |
| RISC Zero zkVM | Rust | Apache/MIT | v3.0.5 (feb 2026) | Producción | Sí (cualquier crate Rust) |
| SP1 | Rust | MIT | v6.1.0 (abr 2026) | Producción | Sí (cualquier crate Rust) |

**Observación sobre gnark:** gnark v0.14.0 no incluye RSA en su `std/signature/` (solo BLS, ECDSA, EdDSA). Para RSA habría que implementar el circuito de big integer modular exponentiation manualmente en Go usando las primitivas de gnark. La librería kipz/zkp-jwt eligió gnark para ES256 (ECDSA) porque esa curva está disponible en `std/signature/ecdsa`.

**Tiempo de generación en hardware de servidor:** Para un circuito RSA-2048 de ~1.5M constraints con Groth16:
- Con snarkjs en Node.js (servidor estándar de 8 cores, 16GB RAM): aproximadamente **15-45 segundos** por prueba.
- Con bellman en Rust (paralelizado): **3-10 segundos** en hardware de servidor de 8-16 cores. MystenLabs opera un proving service para zkLogin con latencia reportada de ~1-3 segundos en hardware especializado.
- Con RISC Zero (modo Bonsai/cloud): tiempos variables según circuito, típicamente **10-60 segundos** para programas simples. La ventaja es no requerir implementación de circuito custom.

**Implicancias para el modelo de seguridad:** El servidor vería el JWT completo (incluyendo el CUIT) antes de generar la prueba. Esto no elimina el problema de "administrador malicioso", pero lo **reduce** porque: (1) el servidor solo ve el JWT durante el proving, no lo almacena, y (2) sin el JWT real, no puede generar una prueba válida. La prueba actúa como un recibo criptográfico de que el servidor vio un JWT válido. Un administrador malicioso podría todavía falsificar una prueba si comprometiera la proving key (lo cual no debería estar disponible en texto plano en el servidor de producción), pero no puede fabricar una prueba sin un JWT real de AUTENTICAR.

### 3.3 Comparación y enfoques híbridos

**Enfoque híbrido (análogo a zkLogin):** zkLogin separa la responsabilidad en tres servicios: el frontend del wallet (genera la llave efímera), un salt service (gestiona el salt del usuario), y un proving service (genera la prueba ZK a partir del JWT). El JWT viaja al proving service, que lo procesa y devuelve la prueba. Este diseño es el más practicable: el ciudadano no necesita hardware potente, la latencia es controlable, y el JWT solo es visible durante el proving.

Para el caso del proyecto, el emisor actual ya juega el rol del proving service. La diferencia sería que además de calcular el anon_seed, generaría la prueba ZK. La separación de responsabilidades puede reforzarse si el proving service es una componente auditada y aislada del resto del sistema.

**Problema del multi-dispositivo:** El requisito de que ninguna llave o prueba esté vinculada a un dispositivo específico es completamente compatible con generación en servidor. La prueba ZK se asocia al anon_seed (que es determinístico para cada CUIT + salt del sistema) y no al dispositivo. El ciudadano puede generar su anon_id desde cualquier dispositivo presentando su JWT de AUTENTICAR al emisor, que genera la prueba en el servidor.

**zkLogin y el requisito de no-vinculación a dispositivo:** zkLogin vincula la prueba a una llave efímera almacenada en el dispositivo, precisamente para transacciones blockchain. Este requisito es opuesto al del proyecto argentino, donde no se quiere vinculación a dispositivo. Por eso la arquitectura de zkLogin no es directamente trasladable, pero su núcleo criptográfico (circuito RSA + Groth16) sí lo es.

---

## 4. Trusted Setup

### 4.1 Groth16 y el trusted setup por circuito

Groth16 requiere dos fases de trusted setup:

**Fase 1 (Powers of Tau):** Independiente del circuito. Genera una Common Reference String (CRS) que puede ser reutilizada por cualquier circuito. Existe una ceremonia perpetua bien establecida: `powersOfTau28_hez_final.ptau` con 54 contribuyentes y un beacon Drand. Cubre hasta 2^28 ≈ 268M constraints. Está disponible públicamente en los servidores de Hermez/Polygon. **No es necesario repetir esta fase**: el proyecto puede usar el resultado de la ceremonia pública.

**Fase 2 (circuit-specific setup):** Específica por cada circuito. Genera el proving key y el verifying key para el circuito particular. Si el circuito cambia, la Fase 2 debe rehacerse. La garantía de seguridad es: al menos 1 de N contribuyentes debe haber descartado su entropía correctamente (el protocolo MMORPG, descrito en Bowe, Gabizon y Miers).

**Implicancias operativas para el proyecto:**
- La Fase 2 debe organizarse antes del lanzamiento.
- Requiere N participantes (al menos 3-5 para un sistema de identidad ciudadana; zkLogin usó 111 para mayor credibilidad).
- Cada participante descarga el estado previo, agrega entropía, y pasa el resultado al siguiente. El proceso puede tomar días.
- Snarkjs provee las herramientas para toda la ceremonia.
- El transcript (evidencia de participación) puede publicarse para auditoría pública.
- Si se decide cambiar el circuito después del lanzamiento (por ejemplo, para soportar RSA-4096 o nuevos claims), la Fase 2 debe repetirse.

### 4.2 PLONK y FFLONK: alternativas sin trusted setup por circuito

PLONK (y su variante FFLONK) solo requieren la Fase 1 (Powers of Tau universal), no la Fase 2 específica por circuito. Esto simplifica enormemente la operación: cambios en el circuito no requieren nueva ceremonia. Snarkjs soporta PLONK y FFLONK. gnark soporta PLONK con KZG.

**Contras de PLONK para este caso:**
- Las pruebas PLONK son más grandes que Groth16 (~1.5-3KB vs ~200-256 bytes para Groth16).
- El tiempo de proving y verificación es mayor que Groth16.
- La configuración universal del trusted setup (KZG) sigue dependiendo de una ceremonia Phase 1, aunque esta ya existe públicamente (la ceremonia KZG de Ethereum, usada por PLONK).
- Menos librerías probadas en producción para el caso JWT RSA específico.

### 4.3 STARKs: sin trusted setup alguno

STARKs (como los que usa RISC Zero) no requieren ningún trusted setup. La seguridad se basa únicamente en hashes criptográficos (modelo de oráculo aleatorio). Las desventajas son:
- Pruebas mucho más grandes: cientos de KB o incluso MB (aunque RISC Zero las comprime con Groth16 recursivo a ~200 bytes para el seal final).
- Tiempos de prueba más largos en hardware comparable.
- RISC Zero combina STARKs con un Groth16 recursivo final, lo que técnicamente sí requiere un trusted setup para ese wrapper Groth16. Sin embargo, RISC Zero realiza esa ceremonia por la comunidad.

### 4.4 Cómo manejan el trusted setup los proyectos identificados

- **zkLogin:** Groth16, ceremonia Phase 2 propia (111 participantes, sept 2023). CRS publicada. La Phase 1 reutilizó la ceremonia perpetua Powers of Tau de Hermez (challenge #0081).
- **zkemail/zk-jwt:** Groth16. No especifica una ceremonia pública propia. En desarrollo/testing usa el ptau público de Hermez. Para producción requeriría su propia Phase 2.
- **RISC Zero:** STARKs + Groth16 recursivo (ceremonia gestionada por RISC Zero Inc.).
- **kipz/zkp-jwt:** PLONK con parámetros KZG derivados de la ceremonia de Ethereum.

---

## 5. Tamaño y Costo de las Pruebas

### 5.1 Tamaño de la prueba

- **Groth16 (circom/snarkjs):** La prueba consta de 3 puntos de curva elíptica. En BN254: **192-256 bytes** en formato comprimido. Es el tamaño de prueba más pequeño entre todos los sistemas ZK.
- **PLONK:** Aproximadamente **1.5-3 KB** dependiendo del sistema de compromisos.
- **RISC Zero (STARK + Groth16 seal):** El seal final es ~200 bytes (idéntico a Groth16), pero el receipt completo (incluyendo el journal) puede ser varios KB.

Para el caso del proyecto, cada anon_seed tendría asociada una prueba de **~256 bytes** (Groth16) o ~2KB (PLONK). Las pruebas se almacenarían en la base de datos del sistema, una por identidad anónima.

### 5.2 Tiempo de verificación

La verificación de una prueba Groth16 requiere exactamente **3 emparejamientos de curva elíptica** (pairings), independientemente del tamaño del circuito. En hardware moderno:
- En servidor (CPU): **1-5 milisegundos** por verificación.
- En cliente browser (WASM, gnarkjs/snarkjs): **10-50 milisegundos**.
- En Solidity (on-chain, por referencia): ~200,000 gas ≈ $0.50-2 USD al precio de ETH circa 2025 (no aplicable al proyecto, solo referencia).

La verificación es determinística y no escala con el número de constraints del circuito.

### 5.3 Costo computacional de verificación en servidor

Verificar 1,000 pruebas por segundo en un servidor moderno es completamente factible con Groth16. Esto no es un cuello de botella para el sistema.

### 5.4 Acumulabilidad

Las pruebas Groth16 son **independientes** entre sí. Cada anon_seed tiene su propia prueba independiente. No existe acumulación nativa: no se puede producir una prueba única que certifique múltiples anon_seeds. Sin embargo, usando proof aggregation (por ejemplo, el Groth16 recursivo de RISC Zero, o sistemas como Plonky2), sería posible agrupar múltiples pruebas en una sola, lo que no parece necesario para este caso de uso.

### 5.5 Relación entre JWT y prueba

Una prueba Groth16 de este tipo tiene las siguientes propiedades de unicidad:
- Un mismo JWT puede producir la misma prueba si se re-genera con los mismos inputs (determinismo).
- El campo `jwtNullifier` del circuito zk-jwt es el hash Poseidon de la firma RSA, que es único por JWT (ya que la firma RS256 es determinística para el mismo mensaje y clave). Esto permite detectar si el mismo JWT fue usado dos veces, sin revelar la firma.
- El campo `anon_seed` (equivalente al `addr_seed` de zkLogin) es el output que permite vincular la prueba a la identidad anónima.

---

## 6. Librerías y Ecosistema

### 6.1 Tabla de librerías

| Librería | Lenguaje | Licencia | Versión | Última release | Mantenimiento | URL | Soporte RSA/RS256 |
|---|---|---|---|---|---|---|---|
| circom | Rust (compilador) | GPL-3 | v2.2.3 | Oct 2025 | Activo (iden3) | github.com/iden3/circom | Sí (via circuitos custom) |
| snarkjs | JS/WASM | GPL-3 | v0.7.6 | Ene 2026 | Activo (iden3) | github.com/iden3/snarkjs | Sí (via circom) |
| @zk-email/circuits | JS+Circom | MIT+GPL | v6.4.2 | Jun 2025 | Activo | github.com/zkemail/zk-email-verify | Sí (RSAVerifier65537) |
| @zk-email/jwt-tx-builder-circuits | Circom | MIT+GPL | No release formal | Oct 2025 | Activo | github.com/zkemail/zk-jwt | Sí (RS256/2048) |
| gnark | Go | Apache 2.0 | v0.14.0 | Ago 2025 | Activo (Consensys) | github.com/Consensys/gnark | No nativo (Groth16/PLONK backend, circuito RSA debe implementarse) |
| bellman | Rust | MIT | v0.14.x | 2024 | Mantenimiento | github.com/zkcrypto/bellman | No nativo |
| arkworks | Rust | MIT/Apache | v0.4 | 2024 | Activo | github.com/arkworks-rs | No nativo |
| RISC Zero zkVM | Rust | Apache/MIT | v3.0.5 | Feb 2026 | Activo | github.com/risc0/risc0 | Sí (código Rust nativo) |
| SP1 | Rust | MIT | v6.1.0 | Abr 2026 | Activo | github.com/succinctlabs/sp1 | Sí (código Rust nativo) |
| Noir | Rust/DSL | MIT/Apache | Activo | 2026 | Activo (Aztec) | github.com/noir-lang/noir | Parcial (experimental, no auditado para RSA) |

### 6.2 Nivel de abstracción

**Opciones de alto nivel (circuito pre-construido, adaptar parámetros):**
- `@zk-email/jwt-tx-builder-circuits` + `@zk-email/jwt-tx-builder-helpers`: el más cercano a "configuro parámetros y llamo una función". Requiere adaptar el claim extraído (`cuit`), remover lógica no necesaria (`accountCode`, `command`), y agregar el cálculo del anon_seed como output. Estimación de effort: 2-4 semanas de ingeniero con experiencia básica en circom.

**Opciones de medio nivel (zkVM, código Rust standard):**
- RISC Zero + crate `jwt-compact`: escribir el código de verificación en Rust estándar, sin circuito aritmético. El zkVM genera la prueba STARK del código. Estimación de effort: 1-2 semanas. No requiere expertise en criptografía ZK, solo conocimiento de Rust y la API de RISC Zero. El trusted setup de la Groth16 final es gestionado por RISC Zero.
- SP1: análogo a RISC Zero con API similar.

**Opciones de bajo nivel (implementar circuito desde cero):**
- gnark (Go): implementar big integer modular exponentiation en Go usando primitivas de gnark. Requiere expertise profundo en circuitos aritméticos. Semanas-meses de desarrollo. No recomendado para este caso cuando existen alternativas de alto nivel.

### 6.3 Expertise requerido

- **Circom + snarkjs + @zk-email:** Requiere entender el modelo de circuitos R1CS, Poseidon hashing, y cómo codificar RSA como constraints aritméticas. Expertise medio en ZK. Hay documentación suficiente en el repo de zkemail.
- **RISC Zero / SP1:** Requiere conocimiento de Rust y comprensión básica del modelo de ejecución zkVM (host/guest). No requiere expertise en criptografía ZK. Es la opción más accesible para equipos sin experiencia previa en ZK.
- **gnark / bellman / arkworks:** Requiere expertise avanzado en álgebra de campos finitos y diseño de circuitos aritméticos. No recomendado sin al menos un especialista senior en ZK en el equipo.

### 6.4 Librerías específicas para ZK sobre JWT

Solo `@zk-email/jwt-tx-builder-circuits` y `@zk-email/jwt-tx-builder-helpers` están diseñadas específicamente para el problema de ZK sobre JWT. El ejemplo jwt-validator de RISC Zero es el único otro caso específico encontrado. No existe una librería madura, genérica y auditada que resuelva el problema de forma plug-and-play.

---

## 7. Riesgos y Limitaciones Conocidas

### 7.1 Vectores de ataque contra implementaciones ZK de este tipo

**Underconstrained circuits (el riesgo más grave):** Un circuito ZK puede estar "subconstrained" —es decir, los constraints no verifican completamente la propiedad deseada, permitiendo inputs inválidos que producen pruebas que pasan la verificación. Para circuitos de verificación RSA, este riesgo es alto por la complejidad de la aritmética big integer. El circuito de zkemail fue auditado específicamente por este tipo de vulnerabilidad.

La auditoría de zkSecurity de zkemail/zk-email-verify (que cubre el circuito RSA base) encontró vulnerabilidades reales que fueron corregidas en v6.1.0. El circuito `zk-jwt` aún **no ha sido auditado** y hereda los mismos patrones de circuito.

**Soundness attacks vía inputs malformados:** Un atacante que controla los inputs al circuito (no el JWT real) podría intentar encontrar un witness inválido que satisfaga los constraints. Esto requiere romper la soundness del SNARK, que en la práctica es computacionalmente infeasible si el circuito está correctamente constrained.

**Trusted setup compromise:** Si todos los participantes de la ceremonia Phase 2 coluden o son comprometidos, el atacante podría generar pruebas falsas para cualquier statement. La garantía del protocolo MMORPG es que basta con que un solo participante haya actuado honestamente y descartado su entropía. Para el proyecto argentino, una ceremonia con 10-30 participantes de diversas instituciones (universidades, ONGs, organismos estatales de distintas dependencias) sería suficiente para una credibilidad razonable en el contexto nacional.

**Prefix attacks:** Un atacante podría intentar construir un JWT con contenido diseñado para confundir al circuito sobre dónde empieza y termina un claim, produciendo una prueba válida para un claim distinto al real. El commit más reciente de zk-jwt (7 de octubre de 2025) añade precisamente `boundary checks` para mitigar este vector, lo que indica que el riesgo era real en versiones anteriores.

**Replay de JWT:** Un JWT presentado múltiples veces al emisor siempre producirá el mismo `anon_seed` (comportamiento esperado). El circuito zk-jwt expone el `jwtNullifier` (hash Poseidon de la firma RSA) como output público, lo que permite al sistema detectar si el mismo JWT fue presentado dos veces sin revelar la firma. Esto añade defensa en profundidad sin comprometer la privacidad.

**Validez temporal ignorada dentro del circuito:** El circuito expone el claim `iat` (issued at) como output público pero no impone un constraint sobre su valor. El verificador debe implementar externamente la política de aceptar solo JWTs emitidos dentro de una ventana de tiempo. Si esta validación se omite en la capa de aplicación, un JWT robado podría usarse para generar una prueba válida mucho tiempo después de su emisión.

**Comprometer el proving service:** Si el servidor que genera las pruebas es comprometido, el atacante accede a los JWTs en tránsito. ZK no elimina este vector; lo que elimina es la capacidad de fabricar pruebas sin JWTs reales de AUTENTICAR. La superficie de ataque se reduce pero no desaparece.

### 7.2 Errores de implementación comunes y sus consecuencias

**Confusión entre inputs públicos y privados:** En circom, los signals sin la declaración `private` son públicos por defecto. Si el CUIT se declara accidentalmente como input público, quedará expuesto en la prueba y cualquier verificador podrá leerlo, anulando la garantía de privacidad. Requiere revisión explícita del código del circuito.

**Padding RSA PKCS#1 v1.5 no verificado:** La verificación de firma RS256 no solo verifica la exponenciación modular; verifica también que el resultado sigue el esquema de padding PKCS#1 v1.5 con el OID SHA-256 y el hash del mensaje. Un circuito que solo verifica la exponenciación pero no el padding permite que un atacante forge una "firma" sin tener la clave privada. La librería `RSAVerifier65537` de zkemail verifica el padding correctamente. Una implementación custom debe incluirlo explícitamente.

**Poseidon hash sin separación de dominios:** Usar el mismo hash Poseidon con los mismos parámetros para calcular el `anon_seed` y el `jwtNullifier` puede crear vulnerabilidades de colisión cruzada entre dominios. El estándar es incluir un tag de dominio como primer input al hash para separar usos distintos del mismo primitivo.

**Longitudes máximas mal dimensionadas:** El circuito declara longitudes máximas fijas para el JWT (header + payload). Si AUTENTICAR emite JWTs más largos que ese máximo (algo común en tokens con muchos claims o valores largos en campos como `aud`), la generación del witness fallará silenciosamente o con un error difícil de diagnosticar, y el ciudadano no podrá registrarse. Es necesario analizar la distribución real de longitudes de los JWTs de AUTENTICAR antes de compilar el circuito y fijar esos parámetros con margen.

**Witness generation vs. constraint satisfaction:** La generación del witness (código JavaScript/WASM) y los constraints del circuito son dos artefactos independientes. Es posible que el generador del witness acepte un input inválido y compute un witness que satisface los constraints cuando no debería. La única fuente de verdad sobre qué es válido es el circuito, no el generador. Las pruebas de corrección deben cubrir ambos.

### 7.3 Auditorías de seguridad publicadas

| Proyecto | Auditor | Fecha | Observaciones |
|---|---|---|---|
| zk-email-verify v6.x | zkSecurity | May 2024 | Cubre RSA verifier y regex. Vulnerabilidades corregidas en v6.1.0. PDF en `/audits/zksecurity-audit.pdf` del repo. |
| zk-email-verify | yAcademy | May 2024 | Segunda auditoría independiente. PDF en `/audits/yacademy-audit.pdf` del repo. |
| zkLogin circuit | zkSecurity | Sep 2023 | Publicada antes de la ceremonia de trusted setup. Referenciada en docs.sui.io. |
| gnark (Groth16 verifier Solidity) | LeastAuthority | Ago 2023 | Contratada por Worldcoin. Disponible en `/audits/` del repo de gnark. |
| gnark (PLONK verifier Solidity) | Consensys Diligence | Jun 2023 | Disponible en `/audits/` del repo de gnark. |
| gnark standard library | ZKSecurity.xyz | May 2024 | Disponible en `/audits/` del repo de gnark. |
| gnark (PLONK prover/verifier) | OpenZeppelin | Jun 2024 | Disponible en `/audits/` del repo de gnark. |
| SP1 | Veridise, Cantina, Zellic, KALOS | 2024-2025 | Links disponibles en README de succinctlabs/sp1. Proyecto declarado apto para producción. |
| zk-jwt (jwt-tx-builder) | Sin auditoría | — | README declara explícitamente: "This project has not yet been audited. Not intended for production use." |

**Consecuencia directa:** Cualquier circuito personalizado derivado de `@zk-email/jwt-tx-builder-circuits` para el proyecto argentino requeriría su propia auditoría de seguridad antes de uso en producción. El costo de auditorías especializadas en ZK es elevado (estimaciones de mercado: USD 30.000–150.000 según el tamaño del circuito y la firma auditora).

### 7.4 Rotación de la clave privada de AUTENTICAR

**Comportamiento del circuito ante rotación:** La clave pública RSA (el modulus) es un input público al circuito, no está compilada dentro de él. El circuito verifica "dada esta clave pública como parámetro, la firma es válida". Cuando AUTENTICAR rota su clave privada y publica nuevas JWKs, el circuito no necesita ser recompilado ni el trusted setup repetido. Las nuevas pruebas simplemente declararán el nuevo `kid` y el nuevo modulus como inputs públicos, y serán verificadas contra la JWK nueva.

**El problema real: pruebas antiguas bajo clave rotada.** Las pruebas ZK ya generadas bajo la clave anterior siguen siendo criptográficamente válidas indefinidamente, porque la prueba certifica "dado este modulus específico (el de la clave anterior), la firma era válida". Sin embargo, si AUTENTICAR elimina la JWK anterior de su endpoint JWKS, el verificador ya no puede consultar públicamente ese modulus para verificar la prueba. La prueba quedaría técnicamente válida pero inverificable en la práctica.

**Solución requerida: JWKS histórico.** El sistema debe mantener un registro histórico de todas las JWKs que AUTENTICAR haya publicado, indexadas por `kid`, para poder verificar pruebas generadas bajo claves ya rotadas. MystenLabs implementa exactamente este servicio: `historical-jwks-zklogin` (https://github.com/MystenLabs/historical-jwks-zklogin). Para el proyecto argentino, este componente es un requisito de infraestructura no opcional si se espera que los registros persistan a través de rotaciones de clave.

**Clave comprometida y revocación:** Si una clave de AUTENTICAR es comprometida y rotada de emergencia, todas las pruebas generadas bajo ese `kid` quedan en estado ambiguo: son criptográficamente válidas, pero los JWTs subyacentes podrían haber sido forjados por el atacante que obtuvo la clave privada. El sistema necesita un mecanismo para marcar un `kid` como comprometido y revocar (o marcar como sospechosas) todas las identidades registradas bajo ese `kid`. Dado que el `kid` es un input público de cada prueba, esta operación es técnicamente posible sin revelar datos privados: basta con consultar qué `anon_ids` tienen pruebas asociadas al `kid` comprometido.

---

## 8. Alternativas a ZK para el Mismo Problema

El problema a resolver es: demostrar retrospectivamente que un `anon_id` fue emitido con respaldo de un token real de AUTENTICAR, sin almacenar el token ni datos correlacionables con el CUIT. Se presentan alternativas criptográficas no-ZK evaluadas con el mismo nivel de detalle.

### 8.1 Firma ciega (Blind Signature) sobre el anon_seed

**Descripción del esquema:** El ciudadano calcula localmente `anon_seed = HASH(salt_sistema + CUIT)` en su dispositivo antes de enviar nada al emisor. Luego aplica un factor de cegado sobre el `anon_seed` usando un esquema de blind signature (Chaum RSA blind signatures o variantes más modernas como blind BLS). El flujo sería:

1. El ciudadano recibe el JWT de AUTENTICAR y extrae su CUIT localmente.
2. Calcula `anon_seed = HASH(salt + CUIT)` en el dispositivo.
3. Ciega el `anon_seed` con un factor aleatorio y envía el valor cegado al emisor junto con el JWT completo.
4. El emisor verifica el JWT contra el JWKS de AUTENTICAR. Si es válido, firma ciegamente el valor cegado sin poder ver el `anon_seed` real ni el CUIT.
5. El ciudadano descega la firma y obtiene una firma del emisor sobre su `anon_seed` real.
6. Esta firma se almacena como credencial verificable: demuestra que el emisor autorizó ese `anon_seed` dado un JWT válido.

**Ventajas sobre ZK:** No requiere trusted setup. Implementación con criptografía estándar disponible en librerías maduras. Tiempos de generación y verificación en milisegundos. No requiere circuitos ni expertise en ZK.

**Limitaciones críticas:**
- El emisor sigue viendo el JWT completo en el paso 4. El cegado protege el `anon_seed` pero no el CUIT durante la transmisión. No resuelve la privacidad del CUIT ante el emisor.
- El esquema no demuestra que `anon_seed = HASH(salt + CUIT_del_JWT)`. Un ciudadano podría presentar un JWT de otra persona y cegar un `anon_seed` arbitrario. La firma del emisor certifica "este valor fue autorizado dado un JWT válido", no "este valor es el hash correcto del CUIT de ese JWT". Esto permite fabricar identidades falsas si el ciudadano tiene acceso a JWTs ajenos.
- Requiere que el ciudadano calcule el CUIT localmente antes del cegado, lo que implica confiar en el código del cliente para hacer ese cálculo correctamente.

**Veredicto:** Resuelve parcialmente el problema. No es un reemplazo completo de ZK porque no certifica la relación CUIT → `anon_seed`. Podría usarse como capa adicional de autorización pero no como prueba de correctitud.

### 8.2 Compromiso criptográfico con apertura diferida

**Descripción:** El emisor almacena un compromiso de Pedersen `C = g^CUIT * h^r` junto al `anon_seed`, donde `r` es aleatoriedad que el ciudadano conserva. En caso de auditoría, el ciudadano podría revelar `r` para demostrar que el CUIT comprometido es el que produjo el `anon_seed`.

**Limitaciones críticas:**
- Requiere que el ciudadano conserve `r` indefinidamente, lo que implica almacenamiento persistente en su dispositivo, contrariando el requisito de no vinculación a dispositivo.
- La apertura del compromiso revela el CUIT, destruyendo el anonimato en el momento de la auditoría.
- Sin apertura, el compromiso no demuestra nada verificable públicamente.

**Veredicto:** No resuelve el problema. El compromiso sin apertura no certifica nada; con apertura viola el anonimato. Descartado.

### 8.3 Tercero de confianza con log append-only sellado

**Descripción:** Un organismo independiente (por ejemplo, la Auditoría General de la Nación, el Defensor del Pueblo, o una universidad pública) verifica en tiempo real cada JWT presentado al emisor y registra el evento en un log append-only sellado criptográficamente. El log almacena `{timestamp, HASH(JWT), anon_seed, status=valid}` sin almacenar el JWT original. Las raíces del árbol de Merkle del log se publican periódicamente en un registro público.

**Ventajas:** No requiere ZK. Verificabilidad legal y técnica del log. Implementación con tecnología estándar (Merkle trees, RFC 3161 timestamping).

**Limitaciones:**
- Introduce un tercero de confianza que conoce en tiempo real la relación JWT → `anon_seed`. Si el TTP almacena el JWT o es presionado legalmente para revelar datos, viola la privacidad.
- Disponibilidad operacional: si el TTP está caído, los registros no pueden procesarse.
- La verificabilidad es social y legal, no matemática: depende de confiar en la honestidad del TTP y la integridad de su infraestructura.
- El `HASH(JWT)` almacenado en el log, combinado con una base de datos de JWTs conocidos, podría permitir correlación.

**Veredicto:** Viable como alternativa de menor complejidad técnica si ZK es inabordable en el corto plazo, pero traslada el problema de confianza a una entidad adicional. La privacidad es más débil que con ZK.

### 8.4 TLSNotary / DECO

**Descripción:** TLSNotary y DECO (Decentralized Oracle) son protocolos que permiten a un usuario probar que ciertos datos recibidos en una sesión TLS provienen de un servidor específico, sin revelar el contenido completo. Podrían usarse para probar que el CUIT fue obtenido del endpoint de AUTENTICAR sin revelarlo.

**Estado de madurez:** TLSNotary está activo (https://github.com/tlsnotary/tlsnotary) y usa un notario que debe estar en línea durante la sesión. DECO es investigación académica de Cornell/Chainlink sin implementación de producción lista para este caso de uso. Ambos requieren un componente de red adicional durante la autenticación.

**Aplicabilidad al caso:** Probarían que los datos vienen de AUTENTICAR pero no que el `anon_seed` es la función correcta del CUIT recibido. Requieren combinación con otro esquema (posiblemente ZK) para cerrar el argumento completo.

**Veredicto:** Demasiado experimental para producción en este contexto. No resuelve el problema de forma autónoma. No recomendado.

### 8.5 Resumen comparativo

| Alternativa | Resuelve el problema completo | Complejidad técnica | Requiere TTP | Verificabilidad pública |
|---|---|---|---|---|
| ZK con circom/Groth16 | Sí, matemáticamente | Alta | No | Sí |
| ZK con RISC Zero / SP1 | Sí, matemáticamente | Media | No | Sí |
| Blind signature | No (no certifica CUIT→anon_seed) | Baja | No | Parcial |
| Pedersen commitment | No (revela CUIT al abrir) | Baja | No | No |
| TTP con log sellado | Sí, con supuesto de confianza | Media | Sí | Legal/Social |
| TLSNotary / DECO | No (parcial, experimental) | Alta | Parcial | Experimental |

**Conclusión:** No existe alternativa no-ZK que resuelva el problema completo (verificabilidad sin revelación, sin TTP, sin almacenar el token) con complejidad significativamente menor que ZK. La alternativa más práctica si ZK se descarta en el corto plazo es el TTP con log sellado, asumiendo los costos organizacionales, el riesgo de privacidad adicional y la verificabilidad más débil que implica.

---

## Referencias

### zkLogin y MystenLabs
- Documentación oficial zkLogin: https://docs.sui.io/concepts/cryptography/zklogin
- zkLogin verifier (Rust, servidor de verificación): https://github.com/MystenLabs/zklogin-verifier
- Historical JWKs service para zkLogin: https://github.com/MystenLabs/historical-jwks-zklogin
- React Native zkLogin PoC: https://github.com/MystenLabs/react-native-zklogin-poc

### ZK Email / zk-jwt
- zkemail/zk-jwt (jwt-tx-builder, proyecto principal): https://github.com/zkemail/zk-jwt
- zkemail/zk-email-verify (base tecnológica con circuitos RSA): https://github.com/zkemail/zk-email-verify
- Releases de zk-email-verify: https://github.com/zkemail/zk-email-verify/releases
- Auditoría zkSecurity de zk-email-verify: https://github.com/zkemail/zk-email-verify/tree/main/audits
- Documentación de circuitos @zk-email/circuits: https://github.com/zkemail/zk-email-verify/tree/main/packages/circuits
- npm @zk-email/jwt-tx-builder-circuits: https://www.npmjs.com/package/@zk-email/jwt-tx-builder-circuits
- npm @zk-email/circuits v6.4.2: https://www.npmjs.com/package/@zk-email/circuits

### Otros proyectos ZK sobre JWT
- kipz/zkp-jwt (ES256 con PLONK y gnark, Go): https://github.com/kipz/zkp-jwt
- 2dvorak/zk-jwt-poc (PoC circom, sin RSA completo): https://github.com/2dvorak/zk-jwt-poc
- RISC Zero jwt-validator (ejemplo RS256): https://github.com/risc0/risc0/tree/main/examples/jwt-validator
- RISC Zero jwt-validator README: https://github.com/risc0/risc0/blob/main/examples/jwt-validator/README.md

### Librerías ZK
- circom v2.2.3 (oct 2025): https://github.com/iden3/circom
- snarkjs v0.7.6 (ene 2026): https://github.com/iden3/snarkjs
- gnark v0.14.0 (ago 2025, Apache 2.0): https://github.com/Consensys/gnark
- gnark releases y auditorías: https://github.com/Consensys/gnark/releases
- RISC Zero zkVM v3.0.5 (feb 2026): https://github.com/risc0/risc0
- SP1 zkVM v6.1.0 (abr 2026): https://github.com/succinctlabs/sp1
- Noir DSL (Aztec Labs, MIT/Apache): https://github.com/noir-lang/noir
- Semaphore (protocolo de anonimato ZK, PSE): https://github.com/semaphore-protocol/semaphore

### Trusted Setup / Powers of Tau
- Powers of Tau perpetua (Hermez/Polygon): referenciada en README de snarkjs (archivos `powersOfTau28_hez_final_XX.ptau`)
- Protocolo MMORPG (Bowe, Gabizon, Miers): referenciado en docs de zkLogin ceremonia

### Protocolo ZK Email: demos y ecosistema
- Proof of Twitter (demo zk-email en producción): https://twitter.zk.email/
- ZKP2P (on/off-ramp usando zk-email): referenciado en docs de zk-email-verify
- zkEmail Safe (multisigs por email): referenciado en docs de zk-email-verify

### Alternativas no-ZK investigadas
- TLSNotary: https://github.com/tlsnotary/tlsnotary
- MACI (protocolo de votación anónima ZK, referencia de contexto): https://github.com/privacy-ethereum/maci
