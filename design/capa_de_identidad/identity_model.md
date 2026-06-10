# Modelo de identidad

Este documento describe el modelo de identidad utilizado por la capa de identidad y las propiedades que busca garantizar.

## Identidad real e identidad anónima

El sistema distingue entre dos tipos de identidad:

- identidad real: corresponde a la identidad civil de una persona verificada por un proveedor externo
- identidad anónima: identidad persistente emitida por la capa de identidad y utilizada por el ciudadano dentro de la aplicación destino

La identidad anónima permite participar sin revelar la identidad real del ciudadano.

## Unicidad de identidad

El sistema busca garantizar la siguiente propiedad:

**En un momento dado, una persona real puede tener como máximo una identidad anónima activa por aplicación destino.**

Esta propiedad reduce la posibilidad de manipulación mediante múltiples identidades.

El ciclo de vida de las identidades contempla eventos como:

- pérdida de frase secreta
- emisión de una nueva identidad

Por este motivo, pueden existir identidades anónimas históricas asociadas a una misma persona; en el caso esperado, solo una está activa al mismo tiempo (ver las limitaciones conocidas más abajo).

## Proveedores de verificación

La capa de identidad no implementa directamente verificación. En su lugar integra **proveedores externos de verificación o unicidad**.

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

Estos identificadores **no deben almacenarse bajo ningún concepto** dentro de la capa de identidad ni propagarse a la aplicación destino.

Solo pueden utilizarse temporalmente para generar identificadores derivados irreversibles mediante funciones criptográficas seguras.

Una vez generado el identificador derivado, el identificador original debe descartarse inmediatamente.

Este mecanismo permite detectar duplicaciones sin almacenar información sensible.

## Limitaciones conocidas del principio de unicidad

El principio de unicidad ("una identidad anónima activa por ciudadano por aplicación destino") se respeta en el caso esperado, pero el diseño no puede distinguir entre un ciudadano que realmente perdió su identidad y uno que simula haberla perdido para obtener otra adicional. Esto deriva de dos decisiones: el emisor no almacena vínculo entre ciudadano e identidad anónima (P-0015), y la aplicación no implementa mecanismos de invalidación de `anon_id` (P-0016 para participación ciudadana). La limitación es aceptada dentro del modelo de amenazas intermedio de P-0006.

El análisis completo —escenarios concretos de abuso, justificación del trade-off frente a alternativas con mecanismo de invalidación, y consecuencias operativas— vive en P-0015, sección "Limitaciones conocidas del principio de identidad única". Cualquier material orientado a operadores o asesores técnicos debe documentar esta limitación para que el alcance real del principio sea correctamente entendido.

## Integridad verificable

La arquitectura del sistema no depende de la confianza absoluta en operadores o administradores.

El sistema busca que cualquier manipulación interna resulte detectable mediante:

- credenciales verificables emitidas por proveedores externos
- validación de identidades anónimas
- consistencia entre acciones y credenciales válidas

Este enfoque sigue el principio de **integridad verificable en lugar de confianza ciega**.

## No sugerir memoria entre sesiones

El modelo de identidad requiere que ninguna aplicación destino sugiera, en su interfaz, que el sistema recuerda la identidad anónima utilizada previamente una vez finalizada una sesión.

Concretamente, una aplicación destino no debe:

- mostrar la última identidad anónima utilizada en la pantalla de login
- precargar identidades anteriores
- ofrecer opciones tipo "continuar como [identidad anónima]"

Este principio refuerza el modelo de anonimato persistente: si la aplicación "recuerda" al ciudadano entre sesiones, contradice la percepción de que cada sesión es una interacción independiente. Cada aplicación destino lo concreta en su propia política de gestión de sesiones; en la aplicación de participación ciudadana, esa concreción es P-0005.
