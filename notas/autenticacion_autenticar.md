# Uso de AUTENTICAR para verificación de identidad y emisión de identidades anónimas

*(Documento técnico actualizado con investigación exhaustiva sobre AUTENTICAR/PAEC — abril 2026)*

---

## Contexto

La plataforma requiere un mecanismo que permita verificar que cada ciudadano que participa es una persona real y única dentro del sistema.

El objetivo es garantizar la propiedad fundamental:

    una identidad real → una identidad anónima

Esto es necesario para evitar ataques del tipo Sybil, donde un mismo actor crea múltiples identidades para manipular tendencias de apoyo dentro del sistema.

Además, el diseño debe evitar un riesgo institucional importante: que un administrador con acceso al sistema pueda crear identidades anónimas ficticias sin que exista una identidad real detrás de ellas.

Para resolver este problema, la plataforma propone utilizar AUTENTICAR, la Plataforma de Autenticación Electrónica Central del Estado argentino (también llamada PAEC), como proveedor de identidad.

AUTENTICAR ya integra múltiples sistemas de verificación utilizados por el Estado. Los proveedores de autenticación activos y sus credenciales son *(confirmado — fuente: [argentina.gob.ar/jefatura/innovacion/autenticar/proveedores-de-autenticacion](https://www.argentina.gob.ar/jefatura/innovacion/autenticar/proveedores-de-autenticacion))*:

- **ARCA (ex-AFIP):** autenticación con CUIT y Clave Fiscal (niveles 2, 3 y 4)
- **ANSES:** autenticación con CUIL y Clave de Seguridad Social (niveles 2 y 3)
- **Mi Argentina:** autenticación con CUIL o pasaporte extranjero (nivel 1 básico; nivel 3 con validación biométrica de ReNaPer)
- **ReNaPer:** autenticación con DNI, sexo y número de trámite — sin registro previo (nivel 1)
- **NIC.ar:** servicio para extranjeros con número de composición similar al CUIT (nivel 1)

**AFIP/ARCA y ANSES** usan CUIT/CUIL; **ReNaPer** usa DNI; **MiArgentina** usa CUIL o pasaporte; **NIC.ar** usa un identificador propio. Esto es relevante para el análisis de unicidad que se discute más adelante.

---

## Arquitectura técnica de AUTENTICAR

*(Esta sección incorpora información confirmada que no estaba en el documento original.)*

**Implementación:** AUTENTICAR está desarrollado sobre **Keycloak** *(confirmado — fuente: [desarrolladores-autenticar/tecnologias](https://www.argentina.gob.ar/jefatura/innovacion/autenticar/desarrolladores-autenticar/tecnologias))*. Esto tiene implicaciones técnicas profundas que se detallan a lo largo de este documento.

**Protocolo:** AUTENTICAR utiliza **OpenID Connect (OIDC)** sobre **OAuth 2.0** *(confirmado — fuente: [desarrolladores-autenticar-3](https://www.argentina.gob.ar/jefatura/innovacion-publica/innovacion-administrativa/autenticar/desarrolladores-autenticar-3))*. El documento original indicaba estos protocolos correctamente.

**Estructura de reinos (realms):** La instalación Keycloak de AUTENTICAR está organizada en reinos independientes. Se confirmaron los siguientes reinos activos con discovery endpoints públicos:

| Reino | Discovery endpoint |
|---|---|
| `miargentina` | `https://autenticar.gob.ar/auth/realms/miargentina/.well-known/openid-configuration` |
| `afip` | `https://autenticar.gob.ar/auth/realms/afip/.well-known/openid-configuration` |
| `anses` | `https://autenticar.gob.ar/auth/realms/anses/.well-known/openid-configuration` |
| `renaper` | `https://autenticar.gob.ar/auth/realms/renaper/.well-known/openid-configuration` |
| `master` | `https://autenticar.gob.ar/auth/realms/master/.well-known/openid-configuration` |

El reino `nicar` no respondió al momento de la investigación.

Según la documentación oficial, AUTENTICAR funciona con un modelo de **"dominios de autenticación"**: el AC se registra en un reino específico, y ese reino federa con los IdPs reales (ARCA, ANSES, ReNaPer, etc.). Un usuario que se autentica en ARCA genera una identidad federada en el reino del IdP y otra en el reino del organismo cliente. Los datos del usuario obtenidos del IdP real se cargan en el usuario federado de AUTENTICAR y se exponen en el token que recibe el AC.

**URLs de endpoints de producción** (confirmadas con acceso directo):

```
Authorization:  https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/auth
Token:          https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/token
UserInfo:       https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/userinfo
JWKS:           https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/certs
Logout:         https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/logout
Token Introspect: https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/token/introspect
```

El entorno de **testing** usa el host `tstpaec.gde.gob.ar` (observado en el PDF de documentación). El de **producción** usa `autenticar.gob.ar`.

---

## Qué provee AUTENTICAR

AUTENTICAR funciona como un proveedor de identidad basado en OpenID Connect y OAuth 2.0 *(confirmado)*. Cuando un ciudadano se autentica mediante AUTENTICAR, el sistema que solicitó la autenticación recibe tokens firmados criptográficamente en formato JWT.

La respuesta del endpoint `/token` incluye tres objetos *(confirmado — fuente: PDF `01_obtencion_token_jwt-disae-2026_005.pdf`)*:

- `access_token`: JWT con datos del usuario y scopes de acceso. Es el que contiene los claims de identidad del ciudadano.
- `id_token`: JWT estándar OIDC que identifica al usuario autenticado.
- `refresh_token`: token para renovar la sesión.

Los tiempos de expiración observados en la documentación oficial son: `access_token` = 300 segundos (5 minutos) y `refresh_token` = 1800 segundos (30 minutos).

El algoritmo de firma es **RS256** en todos los reinos *(confirmado — discovery endpoints de todos los reinos consultados)*.

---

## Claims concretos del token JWT

*(Esta sección responde una de las preguntas abiertas del documento original con datos verificados.)*

El documento original presentaba un esquema hipotético del token. Lo siguiente reemplaza esa especulación con lo observado directamente en la documentación oficial (PDF `01_obtencion_token_jwt-disae-2026_005.pdf`, páginas 9–11).

**Header del JWT:**
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "<identificador de clave>"
}
```

**Payload del access_token (capa de Keycloak/AUTENTICAR):**
```json
{
  "jti":           "3fc4a8de-858f-4af3-93bb-5ca9f47aad07",
  "exp":           1617042055,
  "nbf":           0,
  "iat":           1617041755,
  "iss":           "https://tst.autenticar.gob.ar/auth/realms/miargentina",
  "aud":           "test",
  "sub":           "d742f3d9-2c0a-4bb0-a35e-b5478bccfdc5",
  "typ":           "Bearer",
  "azp":           "test",
  "auth_time":     1617041059,
  "session_state": "e3b43ea3-a8fd-4b03-9c0c-1bd5e36599b6",
  "acr":           "0",
  "allowed-origins": ["*"],
  "realm_access":  { "roles": ["uma_authorization"] }
}
```

**Claims de usuario (capa de identidad — con 1 IdP, por ejemplo ARCA):**
```json
{
  "preferred_username": "20348134664",
  "cuit":         20348134664,
  "name":         "LESLIE ANN CHRISTINE",
  "given_name":   "LESLIE ANN",
  "family_name":  "CHRISTINE",
  "email":        "chris@agip.gob.ar",
  "tipo_persona": "F",
  "proveedor":    "afip",
  "nivel":        "3",
  "afip": {
    "cuit":         "20002444373",
    "tipo_persona": "F",
    "name":         "MARIA CELESTE",
    "given_name":   "MARIA CELESTE",
    "family_name":  "MÜLBAYER",
    "nivel":        "3"
  }
}
```

**Claims de usuario con múltiples IdPs (por ejemplo ARCA + ANSES):**
Los atributos a nivel raíz corresponden al **último IdP usado** en el mecanismo de autenticación. Cada IdP también expone sus propios atributos bajo un claim prefijado con el nombre del proveedor (`"afip": {...}`, `"anses": {...}`).

```json
{
  "preferred_username": "20348134664",
  "cuit":   20348134664,
  "proveedor": "afip",
  "nivel":  "3",
  "afip":   { "cuit": "20002444373", "nivel": "3", ... },
  "anses":  { "cuit": "20002444373", "nombre": "MARIA CELESTE", ... }
}
```

**Sobre el claim `jti`:** Sí existe y está presente en el token *(confirmado en el ejemplo del PDF)*. Es un UUID que identifica unívocamente cada token emitido, lo que permite detectar reutilización de tokens y construir registros de auditoría.

El campo `claims_supported` declarado en el discovery endpoint es: `["sub", "iss", "auth_time", "name", "given_name", "family_name", "preferred_username", "email"]`. Los claims adicionales del proveedor (CUIT, tipo_persona, nivel, etc.) son claims extendidos específicos de AUTENTICAR que se agregan al token.

---

## El claim `sub`: comportamiento y estabilidad

*(Esta sección responde la primera pregunta abierta del documento original.)*

El claim `sub` en los tokens de AUTENTICAR es un **UUID interno generado por Keycloak**, no el CUIL ni el CUIT del ciudadano. En el ejemplo documentado: `"sub": "d742f3d9-2c0a-4bb0-a35e-b5478bccfdc5"`.

**Estabilidad dentro de un mismo reino:** En Keycloak, el `sub` es el identificador interno del usuario dentro del reino. Para un mismo usuario en un mismo reino, el `sub` es estable entre sesiones: el usuario federado se crea la primera vez y persiste.

**Entre reinos diferentes:** Este es el punto crítico. AUTENTICAR expone reinos separados por proveedor (`/auth/realms/afip`, `/auth/realms/anses`, `/auth/realms/renaper`, `/auth/realms/miargentina`), cada uno con su propio `iss` y su propio espacio de identidades. En Keycloak, el `sub` de un usuario es **único por reino**, no global. Por lo tanto, **el mismo ciudadano puede tener distintos valores de `sub` según el reino desde el cual se autenticó**.

**Implicaciones para el sistema propuesto:** Si el AC puede recibir tokens de distintos reinos (por ejemplo, un usuario se autentica una vez vía ARCA/reino `afip` y otra vez vía ANSES/reino `anses`), el `sub` no permitirá correlacionar ambos eventos como pertenecientes al mismo ciudadano, aunque el mismo `jti` garantice unicidad del token.

**El identificador confiable para unicidad es `preferred_username` o el identificador específico del proveedor** (campo `cuit`, `cuil`, etc.), no el `sub`. El `preferred_username` en los ejemplos documentados es el CUIT (`"20348134664"`), que es el identificador que normaliza AUTENTICAR para el proveedor ARCA.

**El claim `sub` soporta los tipos `public` y `pairwise`** *(confirmado — `"subject_types_supported": ["public", "pairwise"]` en todos los discovery endpoints)*. El tipo `pairwise` permitiría que el mismo usuario tenga distintos `sub` para distintos clientes, lo que haría aún más difícil su uso como identificador persistente. La configuración efectiva depende de cómo esté configurado el reino para cada AC.

**Resumen sobre `sub`:** El documento original asumía que el `sub` era un identificador estable y único por persona independientemente del verificador usado. Esto **no está confirmado** y probablemente sea **incorrecto** en el caso de múltiples reinos. Para construir el `anon_seed` de forma robusta, se recomienda usar el CUIT o CUIL obtenido de los claims de identidad (no el `sub`), combinado con el identificador del proveedor para detectar colisiones entre identificadores de distintos proveedores (CUIT de ARCA vs. DNI de ReNaPer).

---

## Endpoint JWKS para verificación de firmas offline

*(Esta sección responde la tercera pregunta abierta del documento original.)*

**AUTENTICAR sí expone endpoints JWKS públicos para verificación offline de firmas** *(confirmado con acceso directo a los endpoints en producción)*.

Cada reino tiene su propio JWKS endpoint:

| Reino | JWKS URI |
|---|---|
| `miargentina` | `https://autenticar.gob.ar/auth/realms/miargentina/protocol/openid-connect/certs` |
| `afip` | `https://autenticar.gob.ar/auth/realms/afip/protocol/openid-connect/certs` |
| `anses` | `https://autenticar.gob.ar/auth/realms/anses/protocol/openid-connect/certs` |
| `renaper` | `https://autenticar.gob.ar/auth/realms/renaper/protocol/openid-connect/certs` |

Cada reino usa su propia clave RSA privada distinta. Las claves públicas observadas son del tipo `RSA`, algoritmo `RS256`, uso `sig`. El campo `kid` del header del JWT identifica qué clave del JWKS fue usada para firmar.

Esto permite que el AC verifique la firma del token completamente offline descargando las claves del JWKS correspondiente al `iss` del token, sin necesidad de llamar al servidor de AUTENTICAR en cada verificación. Los SDKs de OIDC (Keycloak Adapters, JJWT, python-jose, etc.) realizan esta operación automáticamente.

El documento original indicaba que el token está firmado por AUTENTICAR y es verificable criptográficamente. Esto es **correcto y confirmado**, con el detalle de que la clave varía según el reino.

---

## Identificadores devueltos por cada verificador

*(Esta sección responde la cuarta pregunta abierta del documento original.)*

Basándose en la documentación de proveedores y los ejemplos de tokens observados:

| Proveedor | Identificador principal en el token | Nombre del claim |
|---|---|---|
| ARCA (ex-AFIP) | CUIT | `cuit` / `preferred_username` |
| ANSES | CUIL | `cuit` (en los ejemplos observados, mismo campo) |
| Mi Argentina | CUIL o pasaporte | `preferred_username` |
| ReNaPer | DNI + número de trámite (autenticación), CUIL (probablemente) en token | no confirmado en doc |
| NIC.ar | Número tipo CUIT | no confirmado en doc |

**¿Normaliza AUTENTICAR el identificador?** Parcialmente. AUTENTICAR no normaliza a un identificador único universal: el campo `cuit` existe en los claims de ARCA y ANSES (ambos usan CUIT/CUIL que tienen el mismo formato numérico), pero para ReNaPer (que autentica por DNI) y NIC.ar (identificador propio) el mapping puede ser diferente. El campo `proveedor` indica cuál fue el IdP, lo que permite al AC saber el contexto del identificador.

En el caso de múltiples IdPs activos simultáneamente, el `preferred_username` a nivel raíz corresponde al **último IdP usado**, y los claims de cada proveedor están disponibles bajo el prefijo del proveedor. Esto no constituye una normalización automática a un único identificador unificado.

**Consecuencia para el `anon_seed`:** Si se usa `HASH(salt + sub)`, el mismo ciudadano puede obtener distintos `anon_seed` al autenticarse por ARCA vs. ANSES porque el `sub` varía por reino. Si se usa `HASH(salt + cuit)` para proveedores que exponen CUIT, se logra unicidad entre ARCA y ANSES, pero ReNaPer autenticado solo por DNI no garantiza tener CUIL disponible. Se recomienda requerir un proveedor específico (por ejemplo ARCA nivel ≥ 2 o ANSES nivel ≥ 2) para garantizar que el identificador sea siempre CUIT/CUIL, y construir el `anon_seed` desde ese claim específico.

---

## Onboarding para integrarse como aplicación cliente

*(Esta sección responde la quinta pregunta abierta del documento original.)*

El proceso de alta está documentado en detalle y es público *(fuente: [argentina.gob.ar/servicio/solicitar-el-alta-de-una-aplicacion-cliente-por-tramites-distancia](https://www.argentina.gob.ar/servicio/solicitar-el-alta-de-una-aplicacion-cliente-por-tramites-distancia))*.

**Hay dos vías según el tipo de organización:**

- **TAD (Trámites a Distancia):** para el sector privado y público en general. Trámite: "Alta de acceso al servicio AUTENTICAR".
- **GDE (Gestión Documental Electrónica):** exclusivo para organismos de la Administración Pública Nacional.

**Información requerida para la solicitud:**
- Nombre, apellido y organismo del responsable técnico.
- Nombre y descripción del sistema.
- Tecnología de desarrollo.
- Proveedor/es de autenticación y tipo de combinación (conjunción "Y" u alternativa "O").
- URL de redirección (`redirect_uri`). Si aún no está disponible, se declara como "En Desarrollo".
- Concurrencia estimada (usuarios mensuales).
- Aceptación de términos y condiciones.

**Datos que provee AUTENTICAR tras aprobar el alta:**
- `client_id`
- `client_secret`
- URL del Endpoint (la URL base del reino asignado)

**Flujo de integración:**
1. Alta en ambiente de **testing** (`tstpaec.gde.gob.ar`).
2. El equipo técnico integra y prueba la aplicación cliente usando los adaptadores provistos.
3. Se envían evidencias de integración exitosa (el equipo de AUTENTICAR provee un documento de ejemplo de evidencias).
4. Se avanza al ambiente de **producción** (`autenticar.gob.ar`) con los datos definitivos.

**Adaptadores soportados:** AUTENTICAR provee ejemplos de integración descargables en Java, PHP, .NET, JavaScript, Angular.js y .NET Framework 4.X. Los adaptadores son módulos que se instalan en el servidor de la aplicación cliente (Apache HTTP Server, Apache Tomcat, JBoss EAP/WildFly, Proxy de Seguridad) y realizan el flujo OIDC con mínimos cambios en la aplicación.

**Disponibilidad:** El servicio es **gratuito** para todos. Los proveedores de identidad individuales podrían cobrar por sus servicios, pero eso se rige por contratos directos entre el AC y el IdP, no por AUTENTICAR. Cualquier organización pública o privada puede solicitar el alta.

---

## Uso del token para emitir una identidad anónima

La plataforma no utiliza directamente la identidad real del ciudadano. En cambio, utiliza el token emitido por AUTENTICAR únicamente como prueba de autenticación válida.

El flujo conceptual general (del documento original) es **correcto**. Con el detalle técnico incorporado, el flujo completo es:

1. El ciudadano solicita crear su identidad anónima.
2. La plataforma redirige al usuario a AUTENTICAR (`/auth/realms/{reino}/protocol/openid-connect/auth`).
3. AUTENTICAR (Keycloak) redirige a su vez al IdP configurado (ARCA, ANSES, etc.).
4. El ciudadano se autentica en el IdP real.
5. AUTENTICAR devuelve a la plataforma el `code` de autorización.
6. La plataforma intercambia el `code` por los tokens (`access_token`, `id_token`, `refresh_token`) vía POST al endpoint `/token`.
7. La plataforma verifica la firma del `access_token` contra el JWKS del reino correspondiente.
8. Si la verificación es válida, extrae el identificador del ciudadano (recomendado: `cuit` o `preferred_username`, **no** el `sub`).

El `anon_seed` se genera como:

    anon_seed = HASH(salt_del_sistema + proveedor + identificador_del_proveedor)

Donde `proveedor` es el valor del claim `proveedor` (por ejemplo `"afip"`) y `identificador_del_proveedor` es el CUIT/CUIL del claim correspondiente. Incluir el `proveedor` en el hash previene colisiones hipotéticas entre el número 20348134664 visto como CUIT de ARCA y el mismo número visto como CUIL de ANSES (que en este caso serían la misma persona, pero conviene hacer explícita la decisión de diseño).

Si se decide usar el `sub` como base del hash, esto solo es seguro si la plataforma exige un único proveedor y un único reino, y si no se permiten cambios de proveedor. En ese caso el `sub` es estable dentro del reino para esa persona.

---

## Garantía de unicidad

*(Confirmado con matices.)*

El valor `anon_seed` se usa para verificar si ese ciudadano ya recibió una identidad anónima. Si el valor ya existe en la base de datos, el sistema no emite una nueva identidad anónima. Esto garantiza que cada identidad real pueda obtener solo una identidad anónima persistente.

**Matiz importante:** La garantía de unicidad solo es robusta si el identificador de base del hash es realmente único por persona física. CUIT/CUIL es un buen candidato para ARCA y ANSES (mismo espacio de identificadores). Para ReNaPer (DNI) o NIC.ar, se necesita confirmar si el token incluye también CUIL o si solo hay DNI, en cuyo caso un ciudadano con DNI podría en teoría obtener una segunda identidad usando ARCA con su CUIT. **Se recomienda restringir los proveedores aceptados para emisión de identidad anónima** a aquellos que garantizan que el identificador entregado es siempre CUIL, para evitar esta ambigüedad.

---

## Prevención de identidades ficticias

*(Confirmado.)*

Este mecanismo limita la posibilidad de fraude institucional. Para emitir una nueva identidad anónima el sistema requiere un token válido firmado por AUTENTICAR. Un administrador del sistema no puede generar tokens válidos arbitrariamente porque:

- La firma del token depende de claves privadas RSA controladas por AUTENTICAR (clave diferente por reino), verificables contra el JWKS público.
- El sistema verifica esa firma antes de emitir una identidad.
- El claim `jti` en el token es un UUID generado por AUTENTICAR en cada autenticación, y puede ser registrado para prevenir el reuso del mismo token.

La limitación es que **AUTENTICAR no documenta públicamente un mecanismo de revocación ni de verificación activa del `jti`** (el endpoint de introspección existe pero no se documenta un uso obligatorio). Si el `access_token` tiene vigencia de 5 minutos, el riesgo de replay de tokens es acotado pero no nulo durante esa ventana.

---

## Auditoría

El sistema puede registrar evidencia auditable del proceso de emisión de identidades anónimas:

- `jti` del token recibido (claim presente y confirmado)
- `iss` del token (indica el reino)
- `iat` y `exp` (timestamps de emisión y expiración)
- `proveedor` (IdP que autenticó al ciudadano)
- `nivel` (nivel de seguridad de la autenticación)
- Momento de emisión de la identidad anónima

El claim `jti` *(confirmado como presente en el token)* es especialmente valioso para la auditoría porque permite correlacionar cada identidad anónima con un evento de autenticación específico, sin almacenar identidad real.

Esto permite detectar inconsistencias si el número de identidades emitidas no coincide con el número de `jti` únicos registrados.

---

## Protección del anonimato

*(Confirmado con precisión adicional.)*

El sistema no almacena:

- CUIL
- DNI
- CUIT
- identificadores reales del ciudadano
- `sub` del token (que podría vincularse al usuario de Keycloak)

Solo se almacena el `anon_seed` derivado mediante funciones criptográficas irreversibles. El `jti` también puede almacenarse para auditoría sin comprometer el anonimato (no revela identidad real).

El documento de federación de usuarios de AUTENTICAR confirma que **AUTENTICAR tampoco almacena las credenciales del usuario** — solo genera un usuario transitorio federado para la duración de la sesión.

---

## Limitaciones

*(El documento original era correcto. Se agregan limitaciones adicionales encontradas.)*

El mecanismo evita la creación arbitraria de identidades anónimas por parte del sistema, pero no elimina completamente todos los riesgos posibles.

**Limitaciones identificadas en la investigación:**

Un administrador con acceso total al sistema aún podría intentar manipular otros aspectos, como modificar bases de datos, alterar contadores, o eliminar registros. Estas limitaciones se mitigan con las medidas complementarias de arquitectura mencionadas en el documento de arquitectura técnica.

Adicionalmente, se identifican las siguientes limitaciones específicas de la integración con AUTENTICAR:

- **`sub` no es universal:** el claim `sub` varía entre reinos. Si la plataforma acepta autenticación por múltiples reinos/proveedores sin control, la misma persona podría obtener múltiples identidades anónimas. Mitigación: usar CUIL como identificador base y restringir los proveedores aceptados.

- **JWKS por reino:** si se cambia el proveedor requerido en el futuro (por ejemplo, de ARCA a ANSES), el JWKS a usar para verificación cambia. Los tokens emitidos en distintas épocas podrían provenir de reinos diferentes.

- **Sin `jti` blacklist documentada:** AUTENTICAR no publica un mecanismo estándar para consultar si un `jti` fue ya utilizado. La ventana de 5 minutos del `access_token` limita el riesgo de replay, pero no lo elimina. Se recomienda que la aplicación mantenga una caché de `jti` usados con TTL igual al tiempo de expiración del token.

- **`preferred_username` como CUIT:** en los ejemplos observados, el `preferred_username` es el CUIT. Esto no está garantizado para todos los proveedores. Para ReNaPer (que autentica por DNI), el `preferred_username` podría ser diferente.

- **El servicio es gratuito pero la disponibilidad no está garantizada contractualmente:** no se encontró un SLA público documentado para AUTENTICAR.

---

## Conclusión

El uso de AUTENTICAR como proveedor de identidad permite construir un mecanismo de emisión de identidades anónimas que combina verificación estatal de identidad, anonimato dentro de la plataforma, y resistencia básica a identidades ficticias. El documento original describía correctamente los principios generales del sistema.

Las investigación confirma que:

- El protocolo es OpenID Connect sobre Keycloak, con endpoints públicos activos en producción.
- Los tokens incluyen `jti`, lo que habilita auditoría robusta.
- El JWKS está públicamente disponible por reino, habilitando verificación offline de firmas.
- El `iss` en producción es `https://autenticar.gob.ar/auth/realms/{reino}`.

Las suposiciones que necesitan revisión son:

- El `sub` **no es estable y universal entre proveedores** — es un UUID de Keycloak por reino. El diseño del `anon_seed` debe basarse en el CUIT/CUIL del claim de identidad, no en el `sub`.
- AUTENTICAR **no normaliza automáticamente** todos los identificadores a un único espacio común. Se requiere diseño explícito para manejar los distintos identificadores de cada proveedor.

El enfoque aprovecha infraestructura existente del Estado argentino y preserva el principio central del sistema, con la condición de que el diseño del hash de identidad anónima incorpore los matices documentados:

    una persona real → una identidad anónima persistente

---

**Fuentes consultadas:**

- `https://www.argentina.gob.ar/jefatura/innovacion/autenticar` — Página principal de AUTENTICAR
- `https://www.argentina.gob.ar/jefatura/innovacion-publica/innovacion-administrativa/autenticar/desarrolladores-autenticar` y subsecciones (Introducción, Proceso, Integración, Tecnologías, Protocolo, Flujos, Federación de Usuarios, Seguridad, Otros Documentos)
- `https://www.argentina.gob.ar/jefatura/innovacion/autenticar/proveedores-de-autenticacion` y subsecciones (Procesos, Niveles de Seguridad)
- `https://www.argentina.gob.ar/sites/default/files/01_obtencion_token_jwt-disae-2026_005.pdf` — PDF oficial "Obtención de token JWT", 12 páginas, DISAE 2026
- `https://www.argentina.gob.ar/sites/default/files/04_paec_cookies-disae-2026.pdf` — PDF oficial "Autenticación con Cookies", DISAE 2026
- `https://www.argentina.gob.ar/servicio/solicitar-el-alta-de-una-aplicacion-cliente-por-tramites-distancia` — Procedimiento de onboarding
- `https://autenticar.gob.ar/auth/realms/{miargentina,afip,anses,renaper}/.well-known/openid-configuration` — Discovery endpoints en producción (accedidos directamente)
- `https://autenticar.gob.ar/auth/realms/{miargentina,afip,anses,renaper}/protocol/openid-connect/certs` — JWKS endpoints en producción (accedidos directamente)
