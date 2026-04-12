# Recordatorios

Cosas a recordar en momentos específicos del desarrollo del proyecto. Cada recordatorio indica cuándo debe activarse.

## Antes de iniciar el desarrollo de la demo

- **Crear un glosario.** Generar `design/glosario.md` con los términos propios del proyecto (identidad anónima, pseudónimo, frase secreta, apoyo, propuesta derivada, termómetro social, vinculación, identidad activa, etc.). El glosario se pospone hasta este momento porque a medida que avanzan las decisiones de diseño pueden surgir términos nuevos o matices en los existentes, y conviene congelarlo cuando el vocabulario esté estable.

- **Revisión final de la documentación conceptual.** Revisar `docs/propuesta/` completo para evaluar si las nuevas decisiones surgidas durante el diseño requieren ajustes, aclaraciones o referencias cruzadas. Posibles puntos a revisar en ese momento: sección "Apoyo sin voto negativo" (ya agregada en esta etapa), coherencia con los ADRs nuevos, actualización del SVG de arquitectura si corresponde.

## Luego de desarrollar la plataforma

- **Guía de instalación y operación.** Desarrollar una guía para administradores que incluya buenas prácticas, cuidados, recomendaciones y advertencias para minimizar los riesgos de revelar la identidad real del participante (por ejemplo: correlacionando logs, persistiendo información sensible, etc.). El material en gestación para esta guía vive en `notas/propuesta_guia_de_instalacion.md`.

## En cualquier momento del proyecto

- **Revisar documentos en gestación en `notas/`.** Los documentos de gestación (como `autenticacion_autenticar.md`) deben revisarse periódicamente para evaluar si ya están maduros para migrar a su lugar definitivo en `design/` o `design/adr/`. Ver `estado_del_trabajo.md` para la lista actual.
