# AUTENTICAR — Plataforma de Autenticación Electrónica Central

## Qué es AUTENTICAR

AUTENTICAR (también denominada PAEC, Plataforma de Autenticación Electrónica Central) es el servicio de autenticación del Estado Argentino. Permite que aplicaciones cliente validen la identidad de ciudadanos argentinos delegando el proceso a proveedores de identidad estatales, sin necesidad de integrarse individualmente con cada organismo y sin almacenar ninguna credencial del usuario.

La plataforma está desarrollada sobre **Keycloak** e implementa el protocolo **OpenID Connect (OIDC)** sobre OAuth 2.0. El servicio es gratuito para organizaciones públicas y privadas. Cualquier organización puede solicitar el alta como aplicación cliente.

La plataforma está administrada por la **Dirección de Interoperabilidad y Servicios de Autenticación Electrónica (DISAE)**, que depende de la Dirección Nacional de Tramitación Digital con el Ciudadano (DNTDC) de la Jefatura de Gabinete de Ministros.

---

## Conceptos fundamentales

**Aplicación Cliente (AC):** cualquier sistema o aplicación que utiliza los servicios de AUTENTICAR para resolver las autenticaciones de sus usuarios. Puede estar desarrollada en cualquier tecnología.

**Proveedor de Autenticación (IdP):** sistema, servicio o aplicación que provee los servicios de autenticación de usuarios a través de PAEC. Los IdPs son organismos estatales (ARCA, ANSES, Mi Argentina, ReNaPer, NIC.ar).

**Dominio de Autenticación:** conjunto de configuraciones que agrupa una o más aplicaciones cliente con uno o más IdPs dentro de un reino de Keycloak. Toda AC en el mismo dominio tiene SSO automático con el resto del grupo.

**Relying Party (RP):** en la terminología OIDC, es la aplicación cliente que confía en el proveedor de autenticación para verificar la identidad del usuario.

---

## Proveedores de autenticación disponibles

| Proveedor | Credenciales del usuario | Niveles disponibles | Identificador entregado |
|---|---|---|---|
| ARCA (ex-AFIP) | CUIT + Clave Fiscal | 2, 3, 4 | CUIT (claim `cuit`) |
| ANSES | CUIL + Clave de Seguridad Social | 2, 3 | CUIL (claim `cuit`) |
| Mi Argentina | CUIL o pasaporte extranjero | 1 (básico), 3 (biométrico) | CUIL o pasaporte |
| ReNaPer | DNI + sexo + número de trámite | 1 | DNI |
| NIC.ar | Identificador propio (formato CUIT) | 1 | Identificador NIC.ar |

Los niveles de seguridad indican el grado de verificación de identidad: nivel 1 es autenticación básica, niveles superiores implican verificación más robusta (por ejemplo, validación biométrica o uso de credenciales con mayor respaldo institucional).

---

## Arquitectura técnica

### Reinos (realms)

La instalación Keycloak de AUTENTICAR está organizada en reinos independientes. Cada reino corresponde a un proveedor de identidad canónico y tiene su propio endpoint de discovery, su propio JWKS y sus propias claves de firma. Las aplicaciones cliente reciben un reino específico según la configuración solicitada al dar de alta; ese reino es del tipo `{nombre-organismo}-{idp}` (por ejemplo, `appentrerios-afip`) y se denomina **reino de dominio**.

Los reinos canónicos (que a su vez actúan como IdP para los reinos de dominio) son:

| Reino canónico | Proveedor |
|---|---|
| `afip` | ARCA (ex-AFIP) |
| `anses` | ANSES |
| `miargentina` | Mi Argentina |
| `renaper` | ReNaPer |

Cuando un usuario se autentica, AUTENTICAR genera una **identidad federada** en el reino de dominio vinculada a la identidad del mismo usuario en el reino canónico del IdP. Los datos obtenidos del IdP se cargan en ese usuario federado y se exponen en el ID Token y el access token. AUTENTICAR nunca almacena las credenciales del usuario.

### Endpoints de producción

La URL base de producción es `https://autenticar.gob.ar/auth/realms/{reino}`.

| Endpoint | URL |
|---|---|
| Discovery (OpenID Configuration) | `https://autenticar.gob.ar/auth/realms/{reino}/.well-known/openid-configuration` |
| Autorización | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/auth` |
| Token | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/token` |
| Introspección | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/token/introspect` |
| UserInfo | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/userinfo` |
| Logout | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/logout` |
| JWKS | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/certs` |
| Check Session iframe | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/login-status-iframe.html` |
| Registro de clientes | `https://autenticar.gob.ar/auth/realms/{reino}/clients-registrations/openid-connect` |

Todos estos endpoints son confirmados por los documentos de discovery en vivo. El endpoint de discovery devuelve el JSON completo que describe las capacidades del reino; se recomienda usarlo como fuente dinámica de configuración en lugar de hardcodear las URLs.

### Capacidades declaradas por los reinos (confirmadas vía discovery)

Todos los reinos canónicos declaran capacidades idénticas:

- **grant_types_supported:** `authorization_code`, `implicit`, `refresh_token`, `password`, `client_credentials`
- **response_types_supported:** `code`, `none`, `id_token`, `token`, `id_token token`, `code id_token`, `code token`, `code id_token token`
- **response_modes_supported:** `query`, `fragment`, `form_post`
- **subject_types_supported:** `public`, `pairwise`
- **id_token_signing_alg_values_supported:** `RS256`
- **userinfo_signing_alg_values_supported:** `RS256`
- **request_object_signing_alg_values_supported:** `none`, `RS256`
- **token_endpoint_auth_methods_supported:** `private_key_jwt`, `client_secret_basic`, `client_secret_post`
- **token_endpoint_auth_signing_alg_values_supported:** `RS256`
- **scopes_supported:** `openid`, `offline_access`
- **claims_supported:** `sub`, `iss`, `auth_time`, `name`, `given_name`, `family_name`, `preferred_username`, `email`
- **claim_types_supported:** `normal`
- **claims_parameter_supported:** `false`
- **request_parameter_supported:** `true`
- **request_uri_parameter_supported:** `true`

### Ambiente de testing

El ambiente de testing tiene dos URLs documentadas en distintos materiales oficiales:

- **URL en PDFs de integración (2026):** `https://tst.autenticar.gob.ar/auth/realms/{reino}/`
- **URL en PDFs de obtención de token (versiones anteriores):** `https://tstpaec.gde.gob.ar/auth/realms/{reino}/`

La más reciente (PDFs 2026) indica `tst.autenticar.gob.ar`. Ninguna de estas URLs es accesible desde internet en general; el acceso al ambiente de testing se obtiene al tramitar el alta de la aplicación cliente, junto con las credenciales de testing. Al momento de esta investigación (abril 2026), ambos subdominios de testing no respondían desde redes externas, lo que es esperable para un ambiente de acceso controlado.

Las credenciales de testing (`client_id`, `client_secret` y URL del endpoint) se entregan automáticamente al responder la solicitud de alta. El cliente debe usar esas credenciales para integrar y luego proveer evidencias de integración exitosa antes de pasar a producción.

### JWKS y claves de firma

Cada reino gestiona su propio par de claves RSA para firma de tokens. Las claves activas al momento de esta investigación son:

**Reino `afip`:**
```json
{
  "kid": "RBgZVCSDFBpfE2oriE6vSrg74Gbq3FiBImTyJ1A3-lg",
  "kty": "RSA",
  "alg": "RS256",
  "use": "sig",
  "n": "scbNs8f92ITwjHq7oeYswtYUYxBuabqM-vM8vV52RdzE9to--LefrFcZ0SqG5mMnLvhyIy5jz8xJwfjtaufaIm5CvTnNt7drSnMdMvYS9u36SVhw1NJ1cMPnaCBO3kO4uxYRbIjoSmFHlhZdvqfc9t9hDurOp6i0Q4-3aAFWgRS0terOQPSw-0kHv9vH4HAKOJkN3FZ_j_PgzCmfTDUzfjT34d_2N9I0H_wjPx0w9zpdzGVeT07E1RbeGxF-DU4rrOpmuvADHXiJuKd8fpT-0rmyr5xyG0bb-e04Fola94-G9aqBCab0szfkujLClKowJtb8U5RkoeEkRtzQcEzQSw",
  "e": "AQAB"
}
```

**Reino `anses`:**
```json
{
  "kid": "4apvNyRO3dv8WIhfRsz5KB5LzLPuBvPTTY5wOYXoUBY",
  "kty": "RSA",
  "alg": "RS256",
  "use": "sig",
  "n": "njhwGZecr66DyM5bd7e79zvxAt2ECCObSDZ4waZOS1X4k9iwI8cCCNvYXTqOiLh0prgRc2iwk4nMjSg_GS2GkXj7-mCCAqxt90cUNdqEJpGuho5wR27xz2eISzOe_6OWYr2xvXB53UZc_nGn81ZveWiF5NrMnNHWvzLWBF-Rxl7i2q0zsoqKhRvjhA8Y9tnIc369VlyyZomTLfQ7ABkAh7jXmkB3ihStT-ISLDhcRI11WPRqfXG-j6vJyxr5L7It9X4R4QpEtnWs21mtGxPOUn8dDWSnIfxcw3XAZPUgZ5gxFOZvYvCyevIZ6genMoACvHdV9VIJ-cuE218vIfBfHQ",
  "e": "AQAB"
}
```

**Reino `miargentina`:**
```json
{
  "kid": "ZK7pje2MBPoW_drTagqjC-j213SrtqKF0fSAUzMtL4Y",
  "kty": "RSA",
  "alg": "RS256",
  "use": "sig",
  "n": "iirSIHkVXLAkgd8IMImQjeKKvvRkuwdfZB6_vYSsuy_Do8zqiVe-H4NZz89MNDBBvz7cjkeYLgII4p54AQ-qXwOp2dMxQFU71GuOl6VPMP1z3MjaYoYwMPH0EO4rlD_caQzx0830T_OF07B0KBdD53RNtEcF8Lkalr_qq7AKHSwUmqb7JcA8qCEP8XR93ET24liT8b7XDPMA-auY_K-eoq07eNOv_5cCTqB32mcZ3Hbhj8Fe0lkUAnZJfw7HDexObztJKQoSZBDs1hBA0_QwANMgb0gNvWptYEVumyVwIt027J3-w9hz_OsRIq9CQaIsT2olQemyH0MDwiHdr85yvQ",
  "e": "AQAB"
}
```

**Reino `renaper`:**
```json
{
  "kid": "wB3ZRk56gjJX2BUmVG3XhQwUx2cIga21MIcprYrTbNI",
  "kty": "RSA",
  "alg": "RS256",
  "use": "sig",
  "n": "tAW20XjdxQQu2PYXNc3miL_RYFqwVZcbCeYzVFOy-mbE8diIsdiAb9SDnJZnIl3cigBsB5l79gdGqhLgGa-LyhSoarg8H-hgpeew_QrIPVFsRjwPwLL9GEzZAkG3YdPfueX5Hk9EtHrobRpNtvQo5Bc4sVXsUtdOfppycLm5WWSQzrkxhIMmZbt6q-nmUbsEXwte-RZ_prNNh1T6-3rVWok-v4uQoWn6mKdyFGwgFBGrysgj0JXsDTSJD-Q-culBut8G91NZQcb10ytkXI-ocflgaAurWodZ3dudynT7MHkJj0bIeLykVy34sRSMDPJStYMYtprUZzqgh28BeRZxIw",
  "e": "AQAB"
}
```

Estas claves están sujetas a rotación. La integración no debe hardcodear los valores `n` o `e`; debe obtenerlos dinámicamente del endpoint JWKS del reino correspondiente antes de validar cada token (o con cacheo con TTL razonable).

---

## Flujos de autenticación

AUTENTICAR soporta dos flujos OIDC:

**Authorization Code Flow (flujo estándar):** es el flujo recomendado y el más utilizado. Permite al IdP autenticar tanto al usuario como al cliente. El cliente usa `response_type=code`. Es adecuado para aplicaciones con backend seguro capaz de custodiar el `client_secret`.

**Implicit Flow (flujo implícito):** diseñado para aplicaciones client-side donde no es posible guardar el `client_secret` de forma segura. El cliente usa `response_type=token` o `response_type=id_token`. En Keycloak, el flujo implícito devuelve los tokens directamente en el fragment de la URL de redirección, sin el paso de intercambio de código.

---

## Flujo de integración — Authorization Code Flow (detalle completo)

### Paso 1: Request de autorización

La aplicación cliente construye una URL GET hacia el endpoint de autorización del reino y redirige al usuario.

**Formato de la URL:**
```
{URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/auth
  ?response_type=code
  &client_id={client_id}
  &redirect_uri={redirect_uri_codificada}
  &state={valor_opaco}
  &scope=openid
  &login=true
```

**Parámetros:**

| Parámetro | Obligatorio | Descripción |
|---|---|---|
| `response_type` | Sí | `"code"` para Authorization Code Flow; `"token"` para Implicit Flow |
| `client_id` | Sí | El client ID provisto por AUTENTICAR al dar de alta la aplicación |
| `redirect_uri` | Sí | URL de retorno registrada. Debe coincidir exactamente con una URI registrada en AUTENTICAR |
| `state` | Recomendado | Valor opaco generado por el cliente para correlacionar el request y proteger contra CSRF. AUTENTICAR lo devuelve sin modificación |
| `scope` | Sí | Debe incluir `openid`. Se puede combinar con `offline_access` para obtener refresh token de larga duración |
| `login` | Opcional | `true` fuerza la pantalla de login incluso si existe sesión activa |
| `secret` | No (no va en la URL) | El client secret nunca debe exponerse en la URL ni en frontend |

**Ejemplo real (ambiente de testing):**
```
https://tstpaec.gde.gob.ar/auth/realms/miargentina/protocol/openid-connect/auth
  ?response_type=code
  &client_id=test
  &redirect_uri=http%3A%2F%2Flocalhost
  &state=74b52d19-3704-403f-9809-74d47cbc1a4e
  &login=true
  &scope=openid
```

Si el usuario no tiene sesión activa en AUTENTICAR, el sistema lo redirige al IdP configurado (ARCA, ANSES, etc.) para autenticarse. AUTENTICAR maneja este proceso internamente sin que la aplicación cliente necesite flujos adicionales.

**Respuesta de AUTENTICAR (redirección al redirect_uri):**
```
HTTP/1.1 302 Found
Location: http://localhost
  ?code=9b7629bd-7921-416c-8b8b-6e1d236bb258.841603ad-7775-4a99-8bf5-990f5a51ba1f.2d73c5a4-175e-4058-9130-1773c6c8c3d7
  &session_state=841603ad-7775-4a99-8bf5-990f5a51ba1f
```

El `code` tiene formato de tres UUIDs concatenados con puntos. El `session_state` es el identificador de la sesión en el reino.

### Paso 2: Intercambio del código por tokens

La aplicación cliente usa el `code` para solicitar los tokens vía POST al endpoint de token. Este request se realiza desde el backend y nunca debe exponerse al navegador.

**Request:**
```
POST {URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code={code recibido}
&redirect_uri={redirect_uri registrada}
&client_id={client_id}
&client_secret={client_secret}
```

AUTENTICAR soporta tres métodos de autenticación del cliente en el endpoint de token (confirmados por discovery): `client_secret_basic` (credenciales en header Authorization), `client_secret_post` (credenciales en el body, como en el ejemplo anterior) y `private_key_jwt`.

**Respuesta exitosa (HTTP 200):**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
  "token_type": "bearer",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
  "not-before-policy": 0,
  "session_state": "e3b43ea3-a8fd-4b03-9c0c-1bd5e36599b6"
}
```

Los tres tokens son JWT firmados con RS256. El campo `not-before-policy` es un timestamp Unix; si es mayor a 0, indica que tokens emitidos antes de esa fecha no deben aceptarse.

### Paso 3: Verificación del token

La aplicación cliente verifica la firma del `access_token` usando la clave pública del JWKS del reino correspondiente. El proceso:

1. Decodificar el header del JWT (base64url). El campo `kid` identifica qué clave del JWKS se usó.
2. Obtener el JWKS del endpoint `{URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/certs`.
3. Localizar la clave cuyo `kid` coincida con el del header del token.
4. Construir la clave pública RSA a partir de los valores `n` y `e` (modulus y exponent en base64url).
5. Verificar la firma del token contra esa clave pública usando RS256.

La verificación puede realizarse **completamente offline** usando el JWKS público, sin llamar al endpoint de introspección en cada request. Esto es la práctica recomendada para no introducir latencia.

---

## Estructura del JWT

### Header

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "{kid de la clave activa del reino}"
}
```

### Payload del access_token

El payload incluye claims estándar de OIDC/OAuth2 más claims específicos de AUTENTICAR. A continuación se muestra un ejemplo real con todos los campos observados (autenticación vía AFIP):

```json
{
  "jti": "3fc4a8da-053f-4af3-93bb-3ca9f47aad07",
  "exp": 1617042855,
  "nbf": 0,
  "iat": 1617041755,
  "iss": "https://tst.autenticar.gob.ar/auth/realms/miargentina",
  "aud": "test",
  "sub": "d7423b9-2c0a-4bb0-a33a-c5478bccfdc5",
  "typ": "Bearer",
  "auth_time": 1617041655,
  "session_state": "e3b43ea3-a8fd-4b03-9c0c-1bd5e36599b6",
  "acr": "0",
  "allowed-origins": [],
  "realm_access": {
    "roles": ["uma_authorization"]
  },
  "preferred_username": "20348134664",
  "cuit": 20348134664,
  "name": "LESLIE ANN CHRISTINE",
  "given_name": "LESLIE ANN",
  "family_name": "CHRISTINE",
  "email": "chris@agip.gob.ar",
  "tipo_persona": "F",
  "proveedor": "afip",
  "nivel": "3",
  "afip": {
    "cuit": "20002444373",
    "tipo_persona": "F",
    "name": "MARIA CELESTE",
    "given_name": "MARIA CELESTE",
    "family_name": "MÜLBAYER",
    "nivel": "2"
  }
}
```

### Claims del payload — descripción completa

| Claim | Tipo | Descripción |
|---|---|---|
| `jti` | string (UUID) | Identificador único del token. Generado por AUTENTICAR en cada autenticación. Puede usarse para detección de replay. |
| `exp` | número (Unix timestamp) | Timestamp de expiración. Los access tokens expiran a los **300 segundos (5 minutos)** de su emisión. |
| `nbf` | número (Unix timestamp) | Not Before. Generalmente 0, indica que el token es válido desde su emisión. |
| `iat` | número (Unix timestamp) | Timestamp de emisión del token. |
| `iss` | string (URL) | Emisor. Formato: `https://autenticar.gob.ar/auth/realms/{reino}`. Se usa para seleccionar el JWKS correcto al validar. |
| `aud` | string | Audiencia. Contiene el `client_id` de la aplicación cliente. |
| `sub` | string (UUID) | Subject. Identificador interno de Keycloak para el usuario dentro del reino. No es estable entre reinos distintos: la misma persona tendrá diferente `sub` en el reino `afip` y en el reino `anses`. No debe usarse como identificador universal. |
| `typ` | string | Tipo de token. `"Bearer"` para access tokens. |
| `auth_time` | número (Unix timestamp) | Momento en que el usuario se autenticó originalmente en el IdP. |
| `session_state` | string (UUID) | Identificador de la sesión en el reino de Keycloak. |
| `acr` | string | Authentication Context Class Reference. Indica el nivel de autenticación según clasificación interna de Keycloak. |
| `allowed-origins` | array de strings | Orígenes permitidos para el token (configuración CORS de la AC). |
| `realm_access` | objeto | Roles asignados al usuario en el reino. El rol `uma_authorization` se asigna por defecto. |
| `preferred_username` | string | Varía según el proveedor. En ARCA suele ser el CUIT como string. No debe usarse como identificador universal. |
| `cuit` | número | CUIT o CUIL del ciudadano. Presente en tokens de ARCA y ANSES. Valor numérico (sin guiones). |
| `name` | string | Nombre completo del usuario según el IdP. |
| `given_name` | string | Nombre(s) del usuario. |
| `family_name` | string | Apellido(s) del usuario. |
| `email` | string | Correo electrónico del usuario, si el IdP lo provee. |
| `tipo_persona` | string | Tipo de persona según el proveedor. `"F"` para persona física, `"J"` para persona jurídica (en ARCA). |
| `proveedor` | string | Identifica el IdP que autenticó al ciudadano. Valores posibles: `"afip"`, `"anses"`, `"miargentina"`, `"renaper"`. |
| `nivel` | string | Nivel de seguridad de la autenticación realizada (como string). Por ejemplo `"3"`. |
| `{nombre_idp}` | objeto | Objeto con claims específicos del IdP, disponible cuando la AC configuró múltiples IdPs. Contiene los claims propios de ese proveedor de forma namespaceada. Ejemplo: el objeto `"afip"` contiene `cuit`, `tipo_persona`, `name`, `given_name`, `family_name`, `nivel`. |

### Claims con múltiples IdPs

Cuando la aplicación cliente tiene configurados varios IdPs en su mecanismo de autenticación, el token expone los datos del **último autenticador utilizado** como claims de nivel superior (sin prefijo), y los datos de cada autenticador del proceso en objetos namespaceados con el nombre del proveedor. Por ejemplo, si el proceso requirió autenticación con AFIP, el objeto `"afip"` contendrá los datos de ese proveedor adicionalmente a los claims de nivel superior.

---

## Scopes disponibles

Según los documentos de discovery, AUTENTICAR declara dos scopes:

| Scope | Descripción |
|---|---|
| `openid` | Obligatorio. Activa el flujo OIDC y hace que se emita un `id_token`. |
| `offline_access` | Solicita un refresh token de larga duración que sobrevive al cierre del navegador (offline access token). En la práctica Keycloak lo trata como un refresh token de duración extendida. |

El scope `openid` es el mínimo requerido. No se documentan scopes adicionales para solicitar claims específicos; los claims de identidad se incluyen en el token según la configuración del reino y el IdP usado.

---

## Tokens: vigencia y renovación

### Vigencias confirmadas

| Token | Vigencia |
|---|---|
| `access_token` | 300 segundos (5 minutos) |
| `refresh_token` | 1800 segundos (30 minutos) |

Estos valores son confirmados tanto por el campo `expires_in: 300` en la respuesta del endpoint de token como por la documentación oficial de cookies de AUTENTICAR.

### Renovación del access_token con refresh_token

Mientras el refresh token esté vigente, la aplicación puede obtener un nuevo access token sin requerir intervención del usuario:

**Request:**
```
POST {URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token={refresh_token}
&client_id={client_id}
&client_secret={client_secret}
```

**Respuesta:** misma estructura que el intercambio inicial: nuevos `access_token`, `refresh_token`, `id_token`, `expires_in` y `refresh_expires_in`.

### Qué ocurre cuando el refresh token expira

Cuando el refresh token expira (después de 30 minutos de inactividad) el intento de renovación falla con un error de token inválido o expirado. La aplicación debe iniciar un nuevo flujo de autenticación completo redirigiendo al usuario al endpoint de autorización.

---

## Endpoint de UserInfo

Disponible en `{URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/userinfo`. Acepta el `access_token` en el header `Authorization: Bearer {token}` y devuelve claims adicionales del usuario según lo que el IdP haya provisto. El endpoint existe y está documentado en el discovery de todos los reinos, pero la documentación oficial recomienda extraer los claims directamente del token JWT para evitar una llamada de red adicional por request.

---

## Endpoint de introspección

Disponible en `{URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/token/introspect`. Permite a una aplicación verificar la validez de un token consultando directamente a AUTENTICAR. Requiere autenticación de cliente (con `client_id` y `client_secret`). Devuelve un objeto JSON con `active: true/false` y los claims del token si está activo.

No se recomienda como mecanismo primario de validación en cada request dado que introduce latencia de red. La alternativa preferida es la validación offline contra el JWKS.

---

## Endpoint de logout

Para cerrar la sesión del usuario en AUTENTICAR (SLO — Single Log Out), la aplicación debe:

1. Invalidar la cookie de sesión en el navegador.
2. Limpiar los tokens almacenados en el servidor.
3. Redirigir al endpoint de logout de PAEC:

```
GET {URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/logout
  ?redirect_uri={url_de_retorno_post_logout}
```

Ejemplo:
```
https://autenticar.gob.ar/auth/realms/afip/protocol/openid-connect/logout
  ?redirect_uri=https://miapp.ejemplo.gob.ar/sesion-cerrada
```

AUTENTICAR cierra la sesión en el reino y redirige al usuario a la `redirect_uri` indicada.

---

## SSO y SLO entre aplicaciones del mismo dominio

AUTENTICAR implementa SSO (Single Sign-On) y SLO (Single Log-Out) a nivel de dominio de autenticación:

**SSO:** si un usuario ya se autenticó en una AC del dominio, al acceder a otra AC del mismo dominio AUTENTICAR detecta la sesión activa y la completa sin que el usuario deba ingresar credenciales nuevamente. El flujo de autorización se completa transparentemente.

**SLO:** si un usuario cierra sesión en una AC del dominio (siguiendo el flujo de logout descrito), AUTENTICAR cierra la sesión en todo el dominio. Las demás ACs del mismo dominio verán al usuario como no autenticado en el próximo acceso.

---

## Verificación de firma del token — guía de implementación

### Algoritmo

Todos los tokens de AUTENTICAR están firmados con **RS256** (RSASSA-PKCS1-v1_5 con SHA-256). La clave pública necesaria para verificar se encuentra en el JWKS del reino.

### Proceso de verificación

1. Separar el JWT en sus tres partes: `header.payload.signature`.
2. Decodificar el header en base64url y extraer el campo `kid`.
3. Obtener el JWKS del reino: `GET {URL_PAEC}/auth/realms/{reino}/protocol/openid-connect/certs`.
4. Localizar la entrada en `keys[]` cuyo `kid` coincida.
5. Construir la clave pública RSA a partir de `n` (modulus) y `e` (exponent), ambos en base64url.
6. Verificar la firma del JWT usando RS256.
7. Validar los claims estándar: `exp` (no expirado), `iss` (debe coincidir con `https://autenticar.gob.ar/auth/realms/{reino}`), `aud` (debe contener el `client_id` propio).

### Ejemplo en Python (usando PyJWT y cryptography)

```python
import jwt
from cryptography.hazmat.primitives.asymmetric.rsa import RSAPublicNumbers
from cryptography.hazmat.backends import default_backend
import base64, requests, struct

def base64url_to_int(b64):
    data = base64.urlsafe_b64decode(b64 + '==')
    return int.from_bytes(data, 'big')

def get_public_key(realm, kid):
    jwks = requests.get(
        f"https://autenticar.gob.ar/auth/realms/{realm}/protocol/openid-connect/certs"
    ).json()
    for key in jwks["keys"]:
        if key["kid"] == kid:
            n = base64url_to_int(key["n"])
            e = base64url_to_int(key["e"])
            return RSAPublicNumbers(e, n).public_key(default_backend())
    raise ValueError(f"kid {kid} no encontrado en JWKS")

def verify_token(token, realm, client_id):
    unverified = jwt.get_unverified_header(token)
    public_key = get_public_key(realm, unverified["kid"])
    return jwt.decode(
        token,
        public_key,
        algorithms=["RS256"],
        audience=client_id
    )
```

### Ejemplo en Node.js (usando jsonwebtoken y jwks-rsa)

```javascript
const jwksClient = require('jwks-rsa');
const jwt = require('jsonwebtoken');

const client = jwksClient({
  jwksUri: `https://autenticar.gob.ar/auth/realms/${realm}/protocol/openid-connect/certs`
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    callback(err, key?.getPublicKey());
  });
}

jwt.verify(token, getKey, { algorithms: ['RS256'], audience: clientId }, (err, decoded) => {
  if (err) throw err;
  // decoded contiene todos los claims
});
```

---

## Manejo de errores

AUTENTICAR implementa los códigos de error estándar de OAuth 2.0 / OIDC. Los errores en el flujo de autorización se devuelven como parámetros en la URL de redirección; los errores en el endpoint de token se devuelven como JSON en el cuerpo de la respuesta HTTP con status 4xx.

### Errores en el endpoint de autorización (en redirect_uri)

```
{redirect_uri}?error={codigo}&error_description={descripcion}
```

| `error` | Descripción |
|---|---|
| `access_denied` | El usuario denegó la autorización o las credenciales del cliente son inválidas. |
| `invalid_request` | El request está mal formado (parámetros faltantes o inválidos). |
| `unauthorized_client` | El cliente no está autorizado para usar este tipo de respuesta. |
| `unsupported_response_type` | El `response_type` solicitado no es soportado. |
| `invalid_scope` | El scope solicitado es inválido o no reconocido. |
| `server_error` | Error interno del servidor de AUTENTICAR. |
| `temporarily_unavailable` | El servidor está temporalmente no disponible. |

### Errores en el endpoint de token (JSON en body)

```json
{ "error": "...", "error_description": "..." }
```

| `error` | Descripción |
|---|---|
| `invalid_grant` | El `code` es inválido, expirado, ya fue usado, o el `redirect_uri` no coincide con el registrado. |
| `invalid_client` | Las credenciales del cliente (`client_id`/`client_secret`) son inválidas. |
| `invalid_request` | Parámetros faltantes o mal formados en el request. |
| `unauthorized_client` | El cliente no está autorizado para el grant type solicitado. |
