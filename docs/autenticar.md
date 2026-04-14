# AUTENTICAR — Plataforma de Autenticación Electrónica Central

## Qué es AUTENTICAR

AUTENTICAR (también llamada PAEC, Plataforma de Autenticación Electrónica Central) es la plataforma de autenticación del Estado Argentino. Permite que aplicaciones cliente validen la identidad de ciudadanos argentinos delegando la autenticación a proveedores de identidad estatales, sin necesidad de integrarse individualmente con cada organismo.

Está desarrollada sobre **Keycloak** e implementa el protocolo **OpenID Connect (OIDC)** sobre **OAuth 2.0**.

El servicio es gratuito para organizaciones públicas y privadas. Cualquier organización puede solicitar el alta como aplicación cliente.

## Proveedores de autenticación disponibles

| Proveedor | Credenciales | Niveles disponibles | Identificador entregado |
|---|---|---|---|
| ARCA (ex-AFIP) | CUIT + Clave Fiscal | 2, 3, 4 | CUIT (claim `cuit`) |
| ANSES | CUIL + Clave de Seguridad Social | 2, 3 | CUIL (claim `cuit`) |
| Mi Argentina | CUIL o pasaporte extranjero | 1 (básico), 3 (biométrico) | CUIL o pasaporte |
| ReNaPer | DNI + sexo + número de trámite | 1 | DNI |
| NIC.ar | Identificador propio (formato CUIT) | 1 | Identificador NIC.ar |

Los niveles de seguridad indican el grado de verificación de identidad realizado: nivel 1 es autenticación básica, niveles superiores implican verificación más robusta
(por ejemplo, validación biométrica o uso de credenciales con mayor respaldo institucional).

## Arquitectura técnica

### Reinos (realms)

La instalación Keycloak de AUTENTICAR está organizada en reinos independientes, uno por proveedor. Cada reino tiene su propio endpoint de discovery, su propio
JWKS y sus propias claves de firma.

Los reinos activos en producción son:

- `afip` — para ARCA
- `anses` — para ANSES
- `miargentina` — para Mi Argentina
- `renaper` — para ReNaPer

### Endpoints

La URL base de producción es `https://autenticar.gob.ar/auth/realms/{reino}`.

| Endpoint | URL |
|---|---|
| Discovery | `https://autenticar.gob.ar/auth/realms/{reino}/.well-known/openid-configuration` |
| Autorización | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/auth` |
| Token | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/token` |
| JWKS | `https://autenticar.gob.ar/auth/realms/{reino}/protocol/openid-connect/certs` |

El ambiente de testing usa la URL base `https://tstpaec.gde.gob.ar/auth/realms/{reino}`.

### Tokens emitidos

AUTENTICAR emite tres tokens en el intercambio de código:

- `access_token` — token de acceso, vigencia ~5 minutos
- `id_token` — contiene claims de identidad del usuario
- `refresh_token` — permite renovar el acceso sin reautenticación

El `access_token` está firmado con RS256. La clave pública de verificación está disponible en el endpoint JWKS del reino correspondiente.

### Claims relevantes

| Claim | Descripción |
|---|---|
| `sub` | UUID interno de Keycloak. Varía por reino: la misma persona tiene distinto `sub` en `afip` y en `anses`. No es un identificador universal entre proveedores. |
| `iss` | Emisor del token. Formato: `https://autenticar.gob.ar/auth/realms/{reino}` |
| `jti` | UUID único por token. Generado por AUTENTICAR en cada autenticación. |
| `iat` / `exp` | Timestamps de emisión y expiración. |
| `cuit` | CUIT o CUIL del ciudadano (presente en tokens de ARCA y ANSES). |
| `preferred_username` | Varía según el proveedor. En ARCA suele ser el CUIT. No es confiable como identificador universal. |
| `proveedor` | Identifica el IdP que autenticó al ciudadano (por ejemplo `"afip"`). |
| `nivel` | Nivel de seguridad de la autenticación realizada. |

AUTENTICAR no almacena credenciales del usuario. Genera un usuario transitorio federado por la duración de la sesión.

## Flujo de integración (Authorization Code Flow)

1. La aplicación cliente redirige al usuario al endpoint de autorización del reino elegido.
2. AUTENTICAR redirige al IdP configurado (ARCA, ANSES, etc.).
3. El ciudadano se autentica en el IdP.
4. AUTENTICAR devuelve a la aplicación el `code` de autorización vía `redirect_uri`.
5. La aplicación intercambia el `code` por los tokens vía POST al endpoint `/token`, incluyendo `client_id`, `client_secret`, `code` y `redirect_uri`.
6. La aplicación verifica la firma del `access_token` contra el JWKS del reino.
7. Si la verificación es exitosa, extrae los claims necesarios del token.

La verificación de firma puede realizarse offline usando el JWKS público, sin necesidad de llamar al endpoint de introspección en cada request.

## Alta como aplicación cliente

El proceso de alta es público y está documentado en:
`https://www.argentina.gob.ar/servicio/solicitar-el-alta-de-una-aplicacion-cliente-por-tramites-distancia`

Hay dos vías según el tipo de organización:

- **TAD (Trámites a Distancia):** para el sector privado y organismos públicos en general.
- **GDE (Gestión Documental Electrónica):** exclusivo para la Administración Pública Nacional.

La información requerida para la solicitud incluye: datos del responsable técnico, nombre y descripción del sistema, tecnología de desarrollo, proveedores de autenticación requeridos, `redirect_uri` y concurrencia estimada.

AUTENTICAR provee tras el alta: `client_id`, `client_secret` y URL del endpoint del reino asignado.

El flujo estándar es: alta en testing → integración y pruebas → envío de evidencias → alta en producción.

AUTENTICAR provee adaptadores de integración de referencia para Java, PHP, .NET, JavaScript y Angular.js.

## Consideraciones técnicas

- El claim `sub` no es estable entre reinos. No debe usarse como identificador universal entre distintos proveedores.
- El JWKS es específico por reino. Si se integran múltiples proveedores, debe usarse el JWKS del reino correspondiente al `iss` del token.
- No existe un SLA público documentado para AUTENTICAR.
- AUTENTICAR no publica un mecanismo estándar de blacklist de `jti`. La ventana de expiración del `access_token` (~5 minutos) limita el
  riesgo de replay pero no lo elimina por completo.
