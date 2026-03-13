# Uso de AUTENTICAR para verificación de identidad y emisión de identidades anónimas

## Contexto

La plataforma requiere un mecanismo que permita verificar que cada ciudadano que participa es una persona real y única dentro del sistema.

El objetivo es garantizar la propiedad fundamental:

    una identidad real → una identidad anónima

Esto es necesario para evitar ataques del tipo **Sybil**, donde un mismo actor crea múltiples identidades para manipular tendencias de apoyo dentro del sistema.

Además, el diseño debe evitar un riesgo institucional importante: que un administrador con acceso al sistema pueda crear **identidades anónimas ficticias** sin que exista una identidad real detrás de ellas.

Para resolver este problema, la plataforma propone utilizar **AUTENTICAR**, la Plataforma de Autenticación Electrónica Central del Estado argentino, como proveedor de identidad.

AUTENTICAR ya integra múltiples sistemas de verificación utilizados por el Estado, como:

- MiArgentina
- ANSES
- AFIP / ARCA
- sistemas vinculados al ReNaPer

Esto permite verificar la identidad de un ciudadano utilizando infraestructura estatal existente.

## Qué provee AUTENTICAR

AUTENTICAR funciona como un proveedor de identidad basado en estándares ampliamente utilizados en internet, principalmente:

- OAuth2
- OpenID Connect

Cuando un ciudadano se autentica mediante AUTENTICAR, el sistema que solicitó la autenticación recibe un **token firmado criptográficamente**, normalmente en formato **JWT (JSON Web Token)**.

Este token contiene información como:

    {
      iss: "autenticar.gob.ar",
      sub: "identificador-unico-del-usuario",
      aud: "aplicacion-solicitante",
      iat: timestamp,
      exp: timestamp
    }

El elemento fundamental es que el token está **firmado por AUTENTICAR**.

Esto permite que el sistema verifique criptográficamente que:

- la autenticación ocurrió realmente
- fue emitida por AUTENTICAR
- el token no fue modificado

## Uso del token para emitir una identidad anónima

La plataforma no utiliza directamente la identidad real del ciudadano.

En cambio, utiliza el token emitido por AUTENTICAR únicamente como **prueba de autenticación válida**.

El flujo conceptual es el siguiente:

1. El ciudadano solicita crear su identidad anónima.
2. La plataforma redirige al usuario a AUTENTICAR.
3. AUTENTICAR autentica al ciudadano.
4. AUTENTICAR devuelve a la plataforma un **ID Token firmado**.
5. La plataforma verifica criptográficamente ese token.

Si la verificación es válida, el sistema procede a generar un identificador irreversible asociado a ese ciudadano.

Por ejemplo:

    anon_seed = HASH(salt_del_sistema + identificador_del_token)

Este valor permite representar al ciudadano dentro del sistema sin almacenar su identidad real.

## Garantía de unicidad

El valor `anon_seed` se utiliza para verificar si ese ciudadano ya recibió una identidad anónima.

Si el valor ya existe en la base de datos:

    el sistema no emite una nueva identidad anónima

Esto garantiza que cada identidad real pueda obtener **una sola identidad anónima persistente**.

## Prevención de identidades ficticias

Este mecanismo también limita la posibilidad de fraude institucional.

Para emitir una nueva identidad anónima el sistema requiere un **token válido firmado por AUTENTICAR**.

Un administrador del sistema no puede generar tokens válidos arbitrariamente, ya que:

- la firma del token depende de claves controladas por AUTENTICAR
- el sistema verifica esa firma antes de emitir una identidad

Por lo tanto, cada identidad anónima emitida debe estar respaldada por un evento real de autenticación.

## Auditoría

El sistema puede registrar evidencia auditables del proceso de emisión de identidades anónimas, por ejemplo:

- hash del token recibido
- momento de autenticación
- evento de emisión de identidad anónima

Esto permite detectar inconsistencias si el número de identidades emitidas no coincide con el número de autenticaciones verificadas.

## Protección del anonimato

El sistema no almacena:

- CUIL
- DNI
- identificadores reales del ciudadano

Solo se almacena un identificador derivado mediante funciones criptográficas irreversibles.

Esto permite garantizar:

- unicidad de participación
- preservación del anonimato dentro de la plataforma

## Limitaciones

Este mecanismo evita la creación arbitraria de identidades anónimas por parte del sistema, pero no elimina completamente todos los riesgos posibles.

Un administrador con acceso total al sistema aún podría intentar manipular otros aspectos, como:

- modificar bases de datos
- alterar contadores de votos
- eliminar registros

Por esta razón, la arquitectura de la plataforma incorpora otras medidas complementarias como:

- separación de funciones entre servicios
- auditoría pública
- software abierto y verificable

Estas medidas se describen con mayor detalle en el documento de arquitectura técnica.

## Conclusión

El uso de AUTENTICAR como proveedor de identidad permite construir un mecanismo de emisión de identidades anónimas que combina:

- verificación estatal de identidad
- anonimato dentro de la plataforma
- resistencia básica a identidades ficticias

Este enfoque aprovecha infraestructura existente del Estado argentino mientras preserva el principio central del sistema:

    una persona real → una identidad anónima persistente
