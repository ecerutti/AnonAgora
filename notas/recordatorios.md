# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Crear un glosario.** Generar `design/glosario.md` con los términos propios del proyecto (identidad anónima, pseudónimo, frase secreta, apoyo, propuesta derivada, termómetro social, vinculación, identidad activa, etc.). El glosario se pospone hasta este momento porque a medida que avanzan las decisiones de diseño pueden surgir términos nuevos o matices en los existentes, y conviene congelarlo cuando el vocabulario esté estable.

- **Revisión final de la documentación conceptual.** Revisar `docs/propuesta/` completo para evaluar si las nuevas decisiones surgidas durante el diseño requieren ajustes, aclaraciones o referencias cruzadas. Posibles puntos a revisar en ese momento: sección "Apoyo sin voto negativo" (ya agregada en esta etapa), coherencia con los ADRs nuevos, actualización del SVG de arquitectura si corresponde.

- **Revisión final de documentación de diseño** Revisar `design/` completo con todas sus sub carpetas para evaluar sin la documentación de diseño quedó coherente, si no se contradice, y si las posibles contradicciones fueron subsanadas en documentos posteriores. Por ejemplo, el ADR P-0006 decide aceptar un riesgo de amenaza intermedio el cual no justifica el uso de ZK pero luego en P-0014 se decide usar ZK, entonces debe quedar documentado el cambió de decisión y existir una referencia al documento que supercede o redefine.

- **Verificación final del uso del nombre "AnonAgora".** Ejecutar `grep -i "anonagora"` sobre todo el repositorio. Los únicos resultados legítimos deben estar en:
  - `AGENTS.md`, sección "Sobre el nombre del proyecto"
  - `docs/index.md`

- **Verificación final de coherencia de la arquitectura modular.** Tras completar la refactorización derivada de P-0021, ejecutar grep sobre el repositorio para detectar usos residuales de "la plataforma" donde corresponda "la capa de identidad", "la aplicación de participación ciudadana" o "el sistema". Verificar que las referencias entre ADRs sigan siendo válidas, que las nuevas carpetas tengan READMEs coherentes, y que los documentos descriptivos reflejen el modelo de capas.

  Cualquier otra ocurrencia es un error a corregir (el proyecto utiliza terminología genérica: "la plataforma", "el sistema", "el proyecto").

- **Formalizar el rol del operador.** Crear un documento en `design/` (propuesta: `modelo_operativo.md` o `roles_y_responsabilidades.md`) que describa los actores institucionales del sistema y sus relaciones: qué es un "operador", si puede haber más de uno, qué separación de poderes hay entre ellos, qué diferencia hay entre quien hosta la plataforma y quien la administra. Esto concreta afirmaciones del modelo de amenazas P-0006 sobre operadores y configuración del sistema. No es ADR, es diseño descriptivo. No es ADR, es diseño descriptivo. Tarea desbloqueada tras el cierre de P-0021; la distinción entre operador de capa y operador de aplicación destino debe ser uno de los puntos abordados.

## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/`, `design/adr/`, `docs/` o `implementation/`. Ver `notas/README.md` y `estado_del_trabajo.md` para la lista actual.

- **Diseño UX del modo debug.** El modo debug requiere una advertencia prominente en todas las interfaces y una confirmación explícita antes de cualquier acción que modifique estado o genere eventos logueables (login, publicar propuesta, dar apoyo, emitir identidad anónima, renovar identidad). La especificación visual y de flujo de estas confirmaciones corresponde a la capa de UX y debe implementarse en cualquier despliegue de la plataforma. Ver P-0020.
