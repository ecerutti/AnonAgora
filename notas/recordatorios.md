# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Crear un glosario.** Generar `design/glosario.md` con los términos propios del proyecto (identidad anónima, pseudónimo, frase secreta, apoyo, propuesta derivada, termómetro social, vinculación, identidad activa, capa de identidad, aplicación destino, etc.). El glosario se pospone hasta este momento porque a medida que avanzan las decisiones de diseño pueden surgir términos nuevos o matices en los existentes, y conviene congelarlo cuando el vocabulario esté estable.

- **Revisión final de la documentación conceptual.** Revisar `docs/propuesta/` completo para evaluar si las nuevas decisiones surgidas durante el diseño requieren ajustes, aclaraciones o referencias cruzadas. Posibles puntos: enmarcar la propuesta como descripción de una aplicación destino específica construida sobre la capa de identidad reutilizable (consigna de P-0021), coherencia con los ADRs nuevos, actualización del SVG de arquitectura conceptual si corresponde.

- **Revisión final de documentación de diseño.** Revisar `design/` completo con todas sus subcarpetas para evaluar si la documentación de diseño quedó coherente, si no se contradice, y si las posibles contradicciones fueron subsanadas en documentos posteriores. Por ejemplo, el ADR P-0006 decide aceptar un riesgo de amenaza intermedio el cual no justifica el uso de ZK pero luego en P-0014 se decide usar ZK; debe quedar documentado el cambio de decisión y existir una referencia al documento que supersede o redefine.

- **Verificación final del uso del nombre "AnonAgora".** Ejecutar `grep -i "anonagora"` sobre todo el repositorio. Los únicos resultados legítimos deben estar en:
  - `AGENTS.md`, sección "Sobre el nombre del proyecto"
  - `docs/index.md`

  Cualquier otra ocurrencia es un error a corregir (el proyecto utiliza terminología genérica: "la plataforma", "el sistema", "el proyecto", "la capa de identidad", "la aplicación destino").

- **Verificación final de coherencia de vocabulario por capas.** Ejecutar grep sobre el repositorio para detectar usos residuales de "la plataforma" donde corresponda "la capa de identidad", "la aplicación de participación ciudadana", "la aplicación destino" o "el sistema". La regla: dentro de un ADR o documento específico de la aplicación participativa, "la plataforma" puede referirse legítimamente a esa aplicación; fuera de ese contexto, conviene revisar caso por caso.

- **Formalizar el rol del operador.** Crear un documento en `design/` (propuesta: `modelo_operativo.md` o `roles_y_responsabilidades.md`) que describa los actores institucionales del sistema y sus relaciones: qué es un "operador", si puede haber más de uno, qué separación de poderes hay entre ellos, qué diferencia hay entre quien hosta la plataforma y quien la administra. La distinción entre **operador de capa** y **operador de aplicación destino** debe ser uno de los puntos abordados (consecuencia de P-0021). No es ADR, es diseño descriptivo.

- **Formalizar la no existencia de perfil administrativo** No existirá un usuario o perfil administrativo. Todas las configuracione que puede realizar un operador, deberá hacerlas a través de archivos de configuración o usando herramientas de línea de comando que alteren la base de datos. Al no existir un perfil o usuario con privilegios administrativos, limita la superficie de ataques, debiendo el atacante conseguir una shell del sistema o acceso de lectura-escritura sobre la tablas críticas (configuración o parámetros) de la base de datos.

## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/`, `design/adr/`, `docs/` o `implementation/`. Ver `notas/README.md` y `estado_del_trabajo.md` para la lista actual.

- **Diseño UX del modo debug.** El modo debug requiere una advertencia prominente en todas las interfaces y una confirmación explícita antes de cualquier acción que modifique estado o genere eventos logueables (login, publicar propuesta, dar apoyo, emitir identidad anónima, renovar identidad). La especificación visual y de flujo de estas confirmaciones corresponde a la capa de UX y debe implementarse en cualquier despliegue de la plataforma. Ver P-0020.
