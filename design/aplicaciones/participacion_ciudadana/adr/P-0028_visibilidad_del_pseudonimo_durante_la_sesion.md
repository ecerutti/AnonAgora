# P-0028 — Visibilidad del pseudónimo durante la sesión

**Estado:** Activo

## Contexto

P-0001 decide que la identidad anónima es visible solo para el propio ciudadano, dentro de su sesión, y nunca frente a otros ciudadanos. Como ilustración menciona un encabezado del tipo `Participando como: Lobo Azul 714`, pero no decide cómo ni cuándo se muestra. P-0002 establece que el pseudónimo existe para que el ciudadano pueda reconocer su identidad anónima y verificar que participa siempre con la misma. El inventario de flujos asumió una presencia persistente en la interfaz y dejó la forma concreta abierta como hueco H-6.

La decisión tiene un aspecto de seguridad. El pseudónimo es la mitad de la credencial de acceso: el login se hace con pseudónimo y frase secreta (P-0004). Un pseudónimo visible durante toda la sesión queda expuesto a quien mire la pantalla por encima del hombro y aparece en cualquier captura de pantalla que el ciudadano comparta. Con el pseudónimo conocido, a un atacante solo le falta la frase.

La pregunta que este ADR responde es: ¿cuándo y cómo se muestra el pseudónimo al ciudadano durante la sesión?

## Opciones consideradas

### Opción A — Siempre visible

El pseudónimo aparece en un lugar fijo de todas las pantallas con sesión.

Ventajas

- Reconocimiento constante de la identidad con la que se participa.

Desventajas

- Exposición máxima a miradas y capturas de pantalla de la mitad de la credencial.

### Opción B — Bienvenida y consulta a pedido

Al iniciar sesión, la aplicación muestra un mensaje de bienvenida con el pseudónimo. Después, el pseudónimo no permanece en pantalla: el ciudadano puede consultarlo cuando quiera desde un elemento discreto de la interfaz ("Mi identidad").

Ventajas

- El ciudadano reconoce su identidad al ingresar y puede verificarla en cualquier momento (P-0002).
- Reduce la exposición del pseudónimo a lo que el ciudadano decide ver.

Desventajas

- La identidad no está a la vista en todo momento; verificarla requiere una acción.

### Opción C — Solo a pedido

El pseudónimo nunca aparece salvo que el ciudadano lo solicite.

Ventajas

- Exposición mínima.

Desventajas

- Al ingresar, el ciudadano no recibe confirmación visible de con qué identidad entró, lo que debilita el propósito de P-0002.

## Decisión

Se adopta la **Opción B**.

- Al iniciar sesión (y al aterrizar en la página principal tras el onboarding), la aplicación muestra un mensaje de bienvenida con el pseudónimo. El mensaje desaparece al navegar o al cerrarlo.
- Durante el resto de la sesión, el pseudónimo no permanece en pantalla. El ciudadano puede consultarlo desde un elemento discreto "Mi identidad", que lo muestra al activarlo.
- En el registro, mientras el ciudadano elige el pseudónimo (F-CI-01) y define su frase secreta (F-AP-01), el pseudónimo se muestra de forma destacada, con la indicación de que es su identificador de ingreso y debe recordarlo.

## Justificación

La Opción B cumple el propósito de P-0002 —reconocer la identidad y verificar que es siempre la misma— sin dejar expuesta de forma permanente la mitad de la credencial de acceso. La bienvenida confirma la identidad en el momento en que importa, al ingresar; la consulta a pedido cubre cualquier verificación posterior. La Opción C sacrifica esa confirmación sin una ganancia de seguridad proporcional, porque la bienvenida es breve y ocurre cuando el ciudadano acaba de escribir el pseudónimo.

## Consecuencias

- Se resuelve el hueco H-6 del inventario de flujos en cuanto a la presencia del pseudónimo. La forma visual concreta del mensaje de bienvenida y del elemento "Mi identidad", y el contenido de un eventual menú asociado (cupo, atajo a "apoyadas por mí", cierre de sesión), corresponden al diseño de interfaz.
- El ejemplo de encabezado persistente mencionado en P-0001 no se adopta. P-0001 no se modifica: su decisión (visible solo para el propio ciudadano) se mantiene íntegra.
- Combinado con la limpieza inmediata de P-0027, el pseudónimo nunca queda a la vista de quien use el dispositivo después de una sesión expirada.

## Referencias

- P-0001 — Visibilidad pública de la identidad anónima
- P-0002 — Representación de identidades anónimas mediante pseudónimos amigables
- P-0004 — Autenticación de identidad anónima
- P-0005 — Gestión de sesiones de usuario
- P-0027 — Actividad y expiración de la sesión
- `notas/inventario_de_flujos_ui_ux.md` — H-6, F-AP-01, F-AP-04
