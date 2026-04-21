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

## Limitaciones conocidas del principio de identidad única

El principio declarado del modelo es "un ciudadano real, una identidad anónima dentro de la plataforma". Las decisiones adoptadas en P-0015 y P-0016 respetan ese principio en el caso esperado: el ciudadano mantiene una única identidad anónima activa y, si la pierde, debe esperar el cool-down antes de obtener una nueva.

Sin embargo, el sistema no puede distinguir entre un ciudadano que realmente perdió su identidad y uno que simula haberla perdido para obtener otra adicional. Esta limitación es una consecuencia directa de dos decisiones del diseño:

- El emisor no almacena ningún vínculo entre el ciudadano y su identidad anónima.
- La plataforma no implementa mecanismos de invalidación de identidades anónimas (ver P-0016).

En consecuencia, un ciudadano que conserva acceso a su identidad anónima original y, cumplido el cool-down, solicita una nueva alegando pérdida, puede operar con dos identidades activas de forma simultánea. Esto le permitiría duplicar su cupo anual de propuestas (ver P-0017), apoyar la misma propuesta desde ambas identidades inflando el conteo de apoyos, o participar con dos identidades en propuestas vinculadas de forma que parezca apoyo independiente.

Esta limitación es aceptada dentro del modelo de amenazas intermedio de P-0006. El cool-down (default 6 meses) impone un costo temporal significativo que reduce pero no elimina el incentivo de este comportamiento. El diseño del sistema prioriza la no construcción de mecanismos de invalidación (porque introducirían vectores de ataque de denegación de servicio, ver P-0016) por sobre la eliminación completa de esta posibilidad de abuso.

La limitación debe estar documentada en cualquier material orientado a operadores o asesores técnicos para que el alcance real del principio sea correctamente entendido.

## Integridad verificable

La arquitectura del sistema no depende de la confianza absoluta en operadores o administradores.

El sistema busca que cualquier manipulación interna resulte detectable mediante:

- credenciales verificables emitidas por proveedores externos
- validación de identidades anónimas
- consistencia entre acciones y credenciales válidas

Este enfoque sigue el principio de **integridad verificable en lugar de confianza ciega**.
