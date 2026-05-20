# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Revisión final de la documentación conceptual.** Revisar `docs/propuesta/` completo para evaluar si las nuevas decisiones surgidas durante el diseño requieren ajustes, aclaraciones o referencias cruzadas. Posibles puntos: enmarcar la propuesta como descripción de una aplicación destino específica construida sobre la capa de identidad reutilizable (consigna de P-0021), coherencia con los ADRs nuevos, actualización del SVG de arquitectura conceptual si corresponde.

- **Refinar la entrada "operador" del glosario.** La entrada actual define el operador sobre "una instancia del sistema" sin recoger la distinción operador de capa / operador de aplicación destino que `design/modelo_operativo.md` formaliza. Refactor de vocabulario (permitido sobre el glosario), no cambio de decisión. Verificar de paso que la entrada quede consistente con P-0024 y con el documento nuevo.

- **Revisión final de documentación de diseño.** Revisar `design/` completo con todas sus subcarpetas para evaluar si la documentación de diseño quedó coherente, si no se contradice, y si las posibles contradicciones fueron subsanadas en documentos posteriores.

- **Unicidad y desambiguación de pseudónimos.** El emisor descarta toda información asociada a la identidad emitida tras recibir confirmación de la aplicación destino, por lo que no puede chequear localmente si un pseudónimo candidato ya fue usado por otro ciudadano. La probabilidad de colisión es baja pero no nula. Pendiente decidir entre varias opciones no mutuamente excluyentes: (1) permitir colisiones y desambiguar en login con la frase secreta (con costo en el motor de login y con consideraciones de seguridad sobre fuerza bruta cuando se conoce que dos identidades comparten pseudónimo); (2) consultar a la aplicación destino al generar cada candidato (rompe parcialmente la independencia operativa post-emisión); (3) mantener en el emisor una tabla disjunta de pseudónimos usados, sin asociación con anon_seed (requiere modificar P-0015 en el alcance de "qué almacena el emisor", agregando una tabla separada); (4) dimensionar los wordlists y el rango numérico para que la colisión sea despreciable a la escala esperada. La combinación natural es 4 como prevención + 1 como red de seguridad. Probablemente amerite un ADR P-0025 o posterior, sobre unicidad y desambiguación de pseudónimos. Identificado al armar el inventario de flujos.

- **Registro de actividad y expiración de sesión durante redacciones largas.** P-0005 establece expiración de sesión por inactividad con default de 1 hora. Si la detección de actividad es solo del lado del servidor, un ciudadano que demore más de una hora redactando una propuesta podría perder todo el trabajo al intentar publicar. Existen mecanismos del lado del cliente para esto: detección de eventos de actividad (tecleo, scroll, mouse) con heartbeats al servidor que renuevan la sesión; re-login transparente al detectar actividad reciente; timers locales que renuevan la sesión cuando hay actividad en formularios abiertos. Decisión real con alternativas, probablemente ADR a futuro. Identificado al armar el inventario de flujos, F-AP-09.

## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/`, `design/adr/`, `docs/` o `implementation/`. Ver `notas/README.md` y `estado_del_trabajo.md` para la lista actual.

- **Diseño UX del modo debug.** El modo debug requiere una advertencia prominente en todas las interfaces y una confirmación explícita antes de cualquier acción que modifique estado o genere eventos logueables (login, publicar propuesta, dar apoyo, emitir identidad anónima, renovar identidad). La especificación visual y de flujo de estas confirmaciones corresponde a la capa de UX y debe implementarse en cualquier despliegue de la plataforma. Ver P-0020.

- **Aceptación de Terminos y condiciones.** La aplicación Plataforma de Participación Ciudadana deberá tener una sección de "Terminos y condiciones" que el ciudadano deberá aceptar al momento de acceder por primera vez.Posiblemente luego del Tutorial explicativo, cuando ya tenga un conocmiento aproximado o genérico de como funciona la plataforma. En dicha sección se deberá informar los términos y condiciones que el ciudadano acepta. Entre ellos la posiblidad de que una propuesta pueda ser retirada/removida/bloqueda de acuerdo a las condiciones definidas en el ADR P-0023. Será necesario elaborar cuidadosamente el texto de esta sección.
