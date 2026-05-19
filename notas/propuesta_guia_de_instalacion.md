# Propuesta para una futura guía de instalación y operación

Este documento acumula ideas, conceptos y recordatorios sobre qué debería contener una futura guía de instalación y operación del sistema dirigida a administradores. No es todavía un entregable del proyecto: es material en gestación para el momento en que corresponda desarrollarlo formalmente.

## Propósito de la futura guía

La guía estará dirigida a personas y organizaciones que decidan desplegar el sistema en su contexto institucional. Su objetivo principal es transmitir las buenas prácticas, cuidados y advertencias necesarias para que el despliegue preserve los principios del sistema, en particular la separación entre identidad real e identidad anónima.

## Contenido sugerido (acumulado durante el diseño)

- **Buenas prácticas para minimizar riesgos de correlación.** Cuidados específicos para evitar que la infraestructura de despliegue permita reconstruir la relación entre identidad real y actividad dentro de la plataforma. Incluye recomendaciones sobre correlación temporal entre eventos, persistencia de información sensible, separación de logs entre componentes, y manejo de metadatos operativos.

- **Manejo de logs en backups.** Los logs no deben incluirse dentro de los backups del sistema, o bien debe realizarse un borrado previo de los logs antes de generar el backup. Esto es importante porque los backups suelen tener políticas de retención más largas que los logs operativos, y mezclarlos puede extender involuntariamente la vida de metadatos sensibles más allá de los plazos definidos por el modelo de amenazas.

- **Riesgo del revisor automático de lenguaje.** Mencionar explícitamente el riesgo de privacidad asociado al envío de texto de propuestas a la API externa de OpenAI (P-0011), y las condiciones bajo las cuales ese riesgo se considera aceptable en contextos reales.

- **Configuración de parámetros operativos.** Guía para ajustar los parámetros configurables del sistema: costo de Argon2id, ventanas y multiplicadores del ranking (P-0010), criterios mínimos de frase secreta, umbrales del revisor de lenguaje, y otros valores que el operador puede adaptar a su contexto. Incluir para cada parámetro: significado, valor por defecto, criterios para ajustarlo al contexto del despliegue, y comportamiento ante cambios con el sistema en operación (ver subsección dedicada más abajo).

- **Semántica de cambios de parámetros con el sistema en operación.** Todos los parámetros configurables del sistema pueden cambiarse con el sistema en operación. Los cambios son prospectivos: se aplican al próximo uso del parámetro, no retroactivamente sobre datos ya generados. La guía debe documentar el comportamiento específico de cada grupo de parámetros, con las siguientes aclaraciones concretas derivadas del análisis de los ADRs vigentes:

    - **Cool-down entre emisiones (P-0015).** El check de cool-down se evalúa al momento de cada nueva solicitud, comparando la `fecha_emision` almacenada contra el parámetro vigente. Acortar el cool-down puede desbloquear antes a ciudadanos que estaban en período de espera. Alargarlo puede bloquear a ciudadanos que estaban a punto de poder emitir. Las identidades ya emitidas no se ven afectadas.

    - **Parámetros de costo de Argon2id (P-0009).** Cada registro almacena los parámetros de costo con los que fue generado (memoria, iteraciones, paralelismo). La verificación de login usa siempre los parámetros almacenados en el registro, no los vigentes. Los parámetros nuevos aplican solo a registros nuevos. El re-hasheo transparente (al verificar exitosamente un login, re-derivar con los parámetros nuevos y actualizar el registro) es una práctica estándar recomendada para migrar gradualmente a parámetros más fuertes.

    - **Parámetros del ranking: G, MT, ME, ventanas temporales (P-0010).** El ranking se recalcula periódicamente sobre el estado actual de las propuestas con los parámetros vigentes. No existe un "score histórico" almacenado que haya que preservar o migrar. Cambiar cualquiera de estos valores produce un ranking distinto en el próximo recálculo, basado en los mismos datos de propuestas y apoyos existentes. El operador puede ajustar G según la etapa de la plataforma (por ejemplo, un valor menor durante los primeros meses).

    - **Límite anual de propuestas (P-0017).** El check de cupo se evalúa al momento de intentar publicar: se cuentan las publicaciones del ciudadano en los últimos 365 días y se comparan contra el límite vigente. Las propuestas ya publicadas no se retiran ni se invalidan. Si se reduce el límite, un ciudadano que ya publicó más propuestas de las que el nuevo límite permite queda sin slots disponibles hasta que alguna de sus publicaciones salga de la ventana de 365 días. La interfaz debe mostrarle su situación actualizada.

    - **Parámetros de tolerancia a fallos (P-0022).** Gobiernan el comportamiento de requests en tiempo real (reintentos, backoff, timeouts). Cambiarlos afecta las próximas requests sin efecto sobre datos almacenados.

    - **Catálogo de causales de retiro (P-0023).** Los textos de las causales se congelan en el tombstone al momento de cada retiro. Agregar causales nuevas no afecta tombstones existentes. Retirar o modificar causales no altera los tombstones ya generados. Una causal eliminada del catálogo deja de estar disponible para retiros futuros.

    - **Plazos de retención de logs (P-0020).** El mecanismo de eliminación evalúa la antigüedad de cada log contra el plazo vigente. Si se acorta la retención, los logs que exceden el nuevo plazo se eliminan en el próximo ciclo de limpieza. **Advertencia de seguridad:** si se alarga la retención, logs que iban a destruirse se preservan por más tiempo, ampliando la superficie de correlación temporal que P-0020 busca minimizar. El operador debe evaluar esta implicancia antes de alargar los plazos.

    - **Modo debug (P-0020).** La activación y desactivación tienen efectos definidos por P-0020: al activar se habilitan campos adicionales en logs; al desactivar se eliminan automáticamente los logs de modo debug. No es un cambio de parámetro sino una operación con efectos específicos documentados.

- **Auditoría y detección de manipulación.** Prácticas recomendadas para habilitar auditoría externa del sistema en funcionamiento.

## Ideas adicionales mencionadas en otros documentos

Algunas de las ideas iniciales para esta guía surgieron originalmente anotadas en `notas/anotaciones_e_ideas_de_trabajo.md` y fueron migradas a este documento a medida que se consolidaron.
