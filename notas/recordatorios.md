# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Revisión final de la documentación conceptual.** Revisar `docs/propuesta/` completo para evaluar si las nuevas decisiones surgidas durante el diseño requieren ajustes, aclaraciones o referencias cruzadas. Posibles puntos: enmarcar la propuesta como descripción de una aplicación destino específica construida sobre la capa de identidad reutilizable (consigna de P-0021), coherencia con los ADRs nuevos, actualización del SVG de arquitectura conceptual si corresponde.

- **Revisión final de documentación de diseño.** Revisar `design/` completo con todas sus subcarpetas para evaluar si la documentación de diseño quedó coherente, si no se contradice, y si las posibles contradicciones fueron subsanadas en documentos posteriores.

- **Formalizar el rol del operador.** Crear un documento en `design/` (propuesta: `modelo_operativo.md` o `roles_y_responsabilidades.md`) que describa los actores institucionales del sistema y sus relaciones: qué es un "operador", si puede haber más de uno, qué separación de poderes hay entre ellos, qué diferencia hay entre quien hostea el sistema y quien lo administra. La distinción entre **operador de capa** y **operador de aplicación destino** debe ser uno de los puntos abordados (consecuencia de P-0021). No es ADR, es diseño descriptivo.

- **Formalizar la no existencia de perfil administrativo.** No existirá un usuario o perfil administrativo. Todas las configuraciones que puede realizar un operador, deberá hacerlas a través de archivos de configuración o usando herramientas de línea de comando que alteren la base de datos. Al no existir un perfil o usuario con privilegios administrativos, se limita la superficie de ataques, debiendo el atacante conseguir una shell del sistema o acceso de lectura-escritura sobre las tablas críticas (configuración o parámetros) de la base de datos. Esta decisión también cubre las operaciones excepcionales introducidas por P-0023 (retiro de propuestas, configuración del catálogo de causales): se ejecutan por línea de comando o configuración, no mediante una interfaz administrativa.

## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/`, `design/adr/`, `docs/` o `implementation/`. Ver `notas/README.md` y `estado_del_trabajo.md` para la lista actual.

- **Diseño UX del modo debug.** El modo debug requiere una advertencia prominente en todas las interfaces y una confirmación explícita antes de cualquier acción que modifique estado o genere eventos logueables (login, publicar propuesta, dar apoyo, emitir identidad anónima, renovar identidad). La especificación visual y de flujo de estas confirmaciones corresponde a la capa de UX y debe implementarse en cualquier despliegue de la plataforma. Ver P-0020.

- **Aceptación de Terminos y condiciones.** La aplicación Plataforma de Participación Ciudadana deberá tener una sección de "Terminos y condiciones" que el ciudadano deberá aceptar al momento de acceder por primera vez.Posiblemente luego del Tutorial explicativo, cuando ya tenga un conocmiento aproximado o genérico de como funciona la plataforma. En dicha sección se deberá informar los términos y condiciones que el ciudadano acepta. Entre ellos la posiblidad de que una propuesta pueda ser retirada/removida/bloqueda de acuerdo a las condiciones definidas en el ADR P-0023. Será necesario elaborar cuidadosamente el texto de esta sección.
