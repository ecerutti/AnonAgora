# Propuesta para una futura guía de instalación y operación

Este documento acumula ideas, conceptos y recordatorios sobre qué debería contener una futura guía de instalación y operación de la plataforma dirigida a administradores. No es todavía un entregable del proyecto: es material en gestación para el momento en que corresponda desarrollarlo formalmente.

## Propósito de la futura guía

La guía estará dirigida a personas y organizaciones que decidan desplegar la plataforma en su contexto institucional. Su objetivo principal es transmitir las buenas prácticas, cuidados y advertencias necesarias para que el despliegue preserve los principios del sistema, en particular la separación entre identidad real e identidad anónima.

## Contenido sugerido (acumulado durante el diseño)

- **Buenas prácticas para minimizar riesgos de correlación.** Cuidados específicos para evitar que la infraestructura de despliegue permita reconstruir la relación entre identidad real y actividad dentro de la plataforma. Incluye recomendaciones sobre correlación temporal entre eventos, persistencia de información sensible, separación de logs entre componentes, y manejo de metadatos operativos.

- **Manejo de logs en backups.** Los logs no deben incluirse dentro de los backups del sistema, o bien debe realizarse un borrado previo de los logs antes de generar el backup. Esto es importante porque los backups suelen tener políticas de retención más largas que los logs operativos, y mezclarlos puede extender involuntariamente la vida de metadatos sensibles más allá de los plazos definidos por el modelo de amenazas.

- **Riesgo del revisor automático de lenguaje.** Mencionar explícitamente el riesgo de privacidad asociado al envío de texto de propuestas a la API externa de OpenAI (P-0011), y las condiciones bajo las cuales ese riesgo se considera aceptable en contextos reales.

- **Configuración de parámetros operativos.** Guía para ajustar los parámetros configurables del sistema: costo de Argon2id, ventanas y multiplicadores del ranking (P-0010), criterios mínimos de frase secreta, umbrales del revisor de lenguaje, y otros valores que el operador puede adaptar a su contexto.

- **Auditoría y detección de manipulación.** Prácticas recomendadas para habilitar auditoría externa del sistema en funcionamiento.

## Ideas adicionales mencionadas en otros documentos

Algunas de las ideas iniciales para esta guía surgieron originalmente anotadas en `notas/anotaciones_e_ideas_de_trabajo.md` y fueron migradas a este documento a medida que se consolidaron.
