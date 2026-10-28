# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Correlación de autoría por el evento de publicación, y revisión de P-0029.** P-0018 desasocia la propuesta de su autor y registra solo un evento `{anon_id, día}` para el cupo, confiando en que la granularidad de día impida reasociar el evento con una propuesta concreta. Esa protección depende del volumen: si en un día se publica una sola propuesta, el único evento de ese día identifica qué `anon_id` (y por lo tanto qué pseudónimo) la escribió, y con pocas publicaciones diarias el conjunto de candidatos es chico. P-0029 eliminó el cruce que agravaba el problema (eventos de publicación ∩ apoyantes de la propuesta, por el apoyo automático del autor), pero el caso del día con pocas publicaciones sigue abierto. Analizarlo como tema de modelo de amenazas en una sesión con Fable (criterio de `CLAUDE.md`), aprovechando para revisar también el diseño criptográfico de P-0029 (derivación de la clave de apoyos y recálculo de etiquetas al re-hashear). Puede afectar el modelo de datos del cupo, por eso conviene resolverlo antes del desarrollo.


## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Formalizar en ADR las decisiones de plataforma tomadas en el inventario de flujos.** H-1 (visitantes sin sesión en solo lectura) y H-2 (tutorial y T&C bloqueantes en el onboarding) son decisiones de plataforma con alternativas reales, registradas por ahora solo en `notas/inventario_de_flujos_ui_ux.md`.

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/`, `design/adr/`, `docs/` o `implementation/`. Ver `notas/README.md` y `estado_del_trabajo.md` para la lista actual.

- **Diseño UX del modo debug.** El modo debug requiere una advertencia prominente en todas las interfaces y una confirmación explícita antes de cualquier acción que modifique estado o genere eventos logueables (login, publicar propuesta, dar apoyo, emitir identidad anónima, renovar identidad). La especificación visual y de flujo de estas confirmaciones corresponde a la capa de UX y debe implementarse en cualquier despliegue de la plataforma. Ver P-0020.

- **Aceptación de Términos y condiciones.** La aplicación Plataforma de Participación Ciudadana deberá tener una sección de "Términos y condiciones" que el ciudadano deberá aceptar al momento de acceder por primera vez. Posiblemente luego del Tutorial explicativo, cuando ya tenga un conocimiento aproximado o genérico de cómo funciona la plataforma. En dicha sección se deberá informar los términos y condiciones que el ciudadano acepta. Entre ellos la posibilidad de que una propuesta pueda ser retirada/removida/bloqueada de acuerdo a las condiciones definidas en el ADR P-0023. Será necesario elaborar cuidadosamente el texto de esta sección.
