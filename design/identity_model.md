# Modelo de identidad

Este documento describe el modelo de identidad utilizado por el sistema y las propiedades que busca garantizar.

## Identidad real e identidad anónima

El sistema distingue entre dos tipos de identidad:

- identidad real: corresponde a la identidad civil de una persona verificada por un proveedor externo
- identidad anónima: identidad persistente utilizada por el ciudadano dentro de la plataforma

La identidad anónima permite participar sin revelar la identidad real del ciudadano.

## Unicidad de identidad

El sistema busca garantizar la siguiente propiedad:

**En un momento dado, una persona real puede tener como máximo una identidad anónima activa.**

Esta propiedad reduce la posibilidad de manipulación mediante múltiples identidades.

El ciclo de vida de las identidades contempla eventos como:

- pérdida de frase secreta
- revocación voluntaria
- emisión de una nueva identidad

Por este motivo, pueden existir identidades anónimas históricas asociadas a una misma persona, pero solo una puede estar activa al mismo tiempo.

## Proveedores de verificación

La plataforma no implementa directamente verificación de identidad.

En su lugar utiliza **proveedores externos de verificación o unicidad**.

Estos proveedores pueden ser, por ejemplo:

- sistemas estatales de identidad digital
- plataformas institucionales
- servicios de autenticación con credenciales verificables

Cuando el proveedor entrega credenciales firmadas asociadas a identificadores únicos estables, el sistema puede sostener una garantía fuerte de unicidad.

Cuando estas propiedades no están disponibles, el sistema puede seguir operando pero con garantías de unicidad más débiles.

## Identificadores derivados

Algunos proveedores de identidad pueden entregar identificadores sensibles del usuario. 

Por ejemplo, en Argentina los verificadores externos (ANSES, AFIP/ARCA, MiArgentina, ReNaPer, etc.) podrían entregar identificadores como:

- DNI
- CUIL/CUIT
- u otros identificadores institucionales

Estos identificadores **no deben almacenarse bajo ningún concepto** dentro del sistema.

Solo pueden utilizarse temporalmente para generar identificadores derivados irreversibles mediante funciones criptográficas seguras.

Una vez generado el identificador derivado, el identificador original debe descartarse inmediatamente.

Este mecanismo permite detectar duplicaciones sin almacenar información sensible.

## Integridad verificable

La arquitectura del sistema no depende de la confianza absoluta en operadores o administradores.

El sistema busca que cualquier manipulación interna resulte detectable mediante:

- credenciales verificables emitidas por proveedores externos
- validación de identidades anónimas
- consistencia entre acciones y credenciales válidas

Este enfoque sigue el principio de **integridad verificable en lugar de confianza ciega**.
