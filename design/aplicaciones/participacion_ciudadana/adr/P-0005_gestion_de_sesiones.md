# P-0005 — Gestión de sesiones de usuario

**Estado:** Activo

## Contexto

Una vez que un ciudadano ha ingresado a la plataforma utilizando su identidad anónima y frase secreta, el sistema necesita gestionar su acceso durante la navegación.

El problema de diseño a resolver es cómo manejar las sesiones de usuario manteniendo un equilibrio entre:

- seguridad
- experiencia de uso
- coherencia con el modelo conceptual de anonimato de la plataforma

La gestión de sesiones es especialmente sensible en este sistema, ya que la plataforma promueve explícitamente el uso de **identidades anónimas persistentes**.  
Por lo tanto, ciertas decisiones de experiencia de usuario pueden afectar la **percepción de anonimato** por parte del ciudadano.

## Alternativas consideradas

### 1. Sesión persistente indefinida

Mientras el usuario no cierre explícitamente su sesión, el sistema recuerda la identidad del usuario entre visitas, permitiendo el ingreso automático sin necesidad de autenticación adicional.

**Ventajas**

- máxima comodidad para el usuario
- reduce la frecuencia de autenticaciones

**Problemas**

- aumenta el riesgo de acceso no autorizado en dispositivos compartidos
- transmite la sensación de que el sistema recuerda permanentemente al usuario
- contradice la narrativa de anonimato del sistema

**Resultado**

Descartado.

### 2. Sesión persistente con reautenticación parcial

Modelo utilizado por algunas plataformas (como Google) donde:

- el sistema recuerda la identidad del usuario
- al expirar la sesión solicita únicamente la contraseña o secreto asociado

Ejemplo de comportamiento:

> "Hola, Lobo Azul 712. Tu sesión expiró por inactividad, por favor, vuelve a introducir la frase secreta para corroborar que eres tú!"

**Ventajas**

- buena experiencia de usuario
- reduce la fricción de ingreso

**Problemas**

- la interfaz revela explícitamente que el sistema recuerda la identidad previa
- puede generar desconfianza respecto al anonimato real del sistema
- contradice el principio de que la plataforma no mantiene una relación identificable con el ciudadano

**Resultado**

Descartado.

### 3. Sesión temporal con expiración completa

El sistema mantiene una sesión de usuario únicamente mientras existe actividad.

Cuando la sesión expira:

- la sesión se destruye completamente
- el sistema no recuerda la identidad previamente utilizada
- el usuario vuelve a la pantalla genérica de ingreso

Para volver a acceder, el ciudadano debe introducir nuevamente:

- su identidad anónima
- su frase secreta

**Ventajas**

- refuerza la percepción de anonimato
- evita accesos accidentales en dispositivos compartidos
- mantiene coherencia conceptual con el modelo de identidad anónima

**Resultado**

Seleccionado.

## Decisión

La plataforma utiliza **sesiones temporales con expiración por inactividad**.

Cuando la sesión expira:

- el sistema elimina completamente la sesión
- el usuario es redirigido a la pantalla de ingreso
- el sistema no muestra la identidad anónima previamente utilizada
- el ciudadano debe ingresar nuevamente su identidad anónima y frase secreta

El tiempo de expiración por inactividad se establece inicialmente en:

```
1 hora
```

Este valor puede ajustarse en el futuro según necesidades operativas o de seguridad.

## Recomendación de diseño de interfaz

La interfaz **no debe sugerir que el sistema recuerda la identidad anónima utilizada previamente una vez que la sesión haya expirado**.

Esto significa que:

- no debe mostrarse el nombre de la identidad anónima en la pantalla de ingreso
- no deben precargarse identidades utilizadas anteriormente
- no deben ofrecerse opciones como "continuar como..."

El objetivo es reforzar la percepción de que el sistema **no mantiene una memoria identificable del ciudadano una vez finalizada la sesión**.

## Consecuencias

Esta decisión:

- refuerza la percepción de anonimato del sistema
- reduce riesgos de acceso indebido en dispositivos compartidos
- simplifica el modelo mental del usuario respecto al funcionamiento de la plataforma

Como contrapartida:

- el usuario deberá ingresar nuevamente su identidad anónima y frase secreta si su sesión expira por inactividad.
