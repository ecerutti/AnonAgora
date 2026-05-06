# Glosario

Este documento congela el vocabulario del proyecto cuando las decisiones de diseño de plataforma están cerradas. Su propósito es servir de referencia para el trabajo posterior, incluyendo el desarrollo de la demo.

El glosario no toma decisiones de diseño ni reemplaza a los ADRs. Las decisiones viven en `design/adr/` y sus subcarpetas; este documento condensa su vocabulario. Ante cualquier contradicción entre el glosario y un ADR, prevalece el ADR.

Las entradas se organizan por categoría temática. Dentro de cada categoría el orden es alfabético.

## 1. Sistema y arquitectura

**aplicación destino**

Componente que recibe del emisor la identidad anónima del ciudadano y define las reglas funcionales de participación. Cada despliegue del sistema tiene exactamente una aplicación destino. La aplicación de participación ciudadana es la única actualmente diseñada.

La aplicación destino opera de forma independiente del emisor a partir del momento de la emisión: no consulta al emisor durante el uso cotidiano del ciudadano.

_Referencia: P-0021._

**capa de identidad**

Infraestructura reutilizable de identidad anónima verificada. Compuesta por el emisor, la integración con el verificador externo, el componente de proving ZK y la infraestructura de JWKS histórico. Cada despliegue del sistema tiene exactamente una capa de identidad, asociada a exactamente una aplicación destino.

_Referencia: P-0021, `design/capa_de_identidad/README.md`._

**despliegue**

Instancia operativa del sistema, compuesta por exactamente una capa de identidad y una aplicación destino que operan juntas. Distintos despliegues son independientes entre sí; un ciudadano que quiera participar en dos aplicaciones destino distintas debe registrarse por separado en cada una.

_Referencia: P-0021._

**el sistema**

El conjunto completo formado por la capa de identidad y la aplicación destino en un despliegue dado. Cuando la documentación necesita referirse al todo, usa "el sistema". Cuando se refiere específicamente a la aplicación de participación ciudadana, usa "la plataforma" (ver entrada correspondiente).

**integridad verificable**

Principio de diseño según el cual el sistema no depende de confianza ciega en operadores o administradores. Las manipulaciones internas deben resultar detectables mediante credenciales verificables, validación de identidades anónimas y consistencia entre acciones y credenciales válidas.

_Referencia: `design/capa_de_identidad/identity_model.md`._

**la plataforma**

Forma abreviada de "la aplicación de participación ciudadana". Se usa como referencia a esa aplicación destino específica, no al sistema completo.

Nota histórica: en documentos anteriores al ADR P-0021, "la plataforma" se usaba indistintamente para referirse al sistema completo o a la aplicación participativa. El uso correcto a partir de P-0021 es el descrito aquí. Ante usos ambiguos en documentos previos, el contexto determina el significado; prevalece la interpretación consistente con P-0021.

_Referencia: P-0021._

**operador**

Entidad institucional responsable de desplegar y administrar una instancia del sistema. El operador configura el sistema mediante archivos de configuración y herramientas de línea de comando; no existe un perfil administrativo en la interfaz. Es responsable de las decisiones operativas dentro del marco definido por el diseño: por ejemplo, configurar el catálogo de causales de retiro o ajustar parámetros del ranking.

_Referencia: P-0020, P-0022, P-0023._

**termómetro social**

Propósito declarado de la aplicación de participación ciudadana: observar qué propuestas e ideas comienzan a reunir apoyo ciudadano, detectar tendencias colectivas emergentes y ofrecer una señal social a legisladores, gobiernos e investigadores. No es un sistema de decisión política vinculante.

_Referencia: `docs/architecture_overview.md`, P-0010._

## 2. Identidad anónima e identificadores

**anon_id**

Identificador técnico de la identidad anónima. Se calcula como `HASH(anon_seed + nonce)`, donde el nonce es generado aleatoriamente por el emisor en cada emisión y descartado de inmediato. El `anon_id` es el output público de la prueba ZK: sale del emisor, llega a la aplicación destino, y es el identificador que la aplicación usa internamente para reconocer al ciudadano.

A diferencia del `anon_seed`, el `anon_id` no puede recalcularse a partir del estado del emisor porque el nonce no se almacena.

> No confundir con `anon_seed`: el `anon_seed` vive exclusivamente en el emisor; el `anon_id` vive en la aplicación destino. Ningún componente tiene ambos.

_Referencia: P-0015 (Decisión 3), P-0014._

**anon_seed**

Identificador interno del emisor, calculado como `HASH(salt_del_sistema + CUIT/CUIL)`. El emisor lo usa para verificar unicidad y aplicar el cool-down. Nunca sale del emisor ni llega a la aplicación destino.

> No confundir con `anon_id`: el `anon_seed` es el identificador del emisor; el `anon_id` es el identificador de la aplicación destino.

_Referencia: P-0013 (Decisión 3), P-0015 (Decisión 2)._

**cool-down**

Período mínimo que debe transcurrir entre emisiones sucesivas de identidad anónima para un mismo ciudadano, medido desde la `fecha_emision` registrada por el emisor. El valor por defecto es 6 meses; es configurable por el operador. Durante el cool-down el emisor rechaza nuevas solicitudes del mismo `anon_seed`.

El cool-down es el principal mecanismo de control de abuso del sistema: impone un costo temporal que desincentiva la acumulación de identidades múltiples. El sistema no puede distinguir entre un ciudadano que genuinamente perdió sus credenciales y uno que simula haberlas perdido; esta limitación es aceptada dentro del modelo de amenazas intermedio de P-0006.

_Referencia: P-0015 (Decisión 1)._

**frase secreta**

Passphrase definida por el ciudadano durante el registro en la aplicación destino. Funciona como credencial de acceso: junto con la identidad anónima, permite al ciudadano autenticarse en visitas posteriores. Es el único mecanismo de acceso; no existe recuperación alternativa.

La frase secreta es asunto exclusivo de la aplicación destino. El emisor no la conoce ni participa en su manejo. Su pérdida hace la identidad anónima irrecuperable hasta que venza el cool-down.

_Referencia: P-0008, P-0009, P-0015._

**identidad anónima**

Identidad persistente emitida por la capa de identidad para uso del ciudadano en la aplicación destino. No contiene datos personales. Se representa ante el ciudadano mediante un pseudónimo amigable (ver entrada) y se identifica internamente mediante el `anon_id`.

El sistema garantiza, en el caso esperado, una identidad anónima por ciudadano real por aplicación destino. Una vez emitida, la aplicación destino no puede invalidarla ni el emisor puede revocarla.

En el uso cotidiano de la documentación, "identidad anónima" es el término preferido para referirse a la identidad que el ciudadano usa en la aplicación. El término "pseudónimo" se reserva para contextos donde es necesario distinguir específicamente la representación legible de otros componentes técnicos de la identidad (ver nota en la entrada de pseudónimo).

_Referencia: P-0002, P-0015, P-0016, `design/capa_de_identidad/identity_model.md`._

**nonce**

Valor aleatorio generado por el emisor en cada emisión. Se combina con el `anon_seed` para calcular el `anon_id` y luego se descarta inmediatamente. No se almacena en ningún componente.

La no persistencia del nonce es lo que hace criptográficamente imposible reconstruir la relación entre `anon_seed` y `anon_id` aunque se tenga acceso completo al estado del emisor.

_Referencia: P-0015 (Decisión 3)._

**pseudónimo (pseudónimo amigable)**

Representación legible de la identidad anónima. Tiene el formato `Animal Color/Adjetivo Número` (por ejemplo, "Lobo Azul 714"). Es generado automáticamente por el sistema; el ciudadano puede solicitar regeneraciones antes de aceptarlo, pero no puede escribirlo libremente. Una vez aceptado, es permanente.

El pseudónimo es visible únicamente para el propio ciudadano; no es público frente a otros ciudadanos.

El término "pseudónimo" se usa cuando es necesario distinguir específicamente esta representación legible de otros componentes de la identidad, por ejemplo en la tupla que el emisor entrega a la aplicación destino `{pseudónimo, anon_id, prueba ZK}`. En todos los demás contextos, el término preferido es "identidad anónima".

_Referencia: P-0002, P-0003, `design/capa_de_identidad/identity_wordlists.md`._

**salt del sistema**

Valor secreto administrado por el emisor que se combina con el CUIT/CUIL del ciudadano al derivar el `anon_seed`. Su confidencialidad protege contra ataques de diccionario sobre el espacio de CUITs, que es finito y semi-público.

_Referencia: P-0013 (Decisión 3)._

## 3. Componentes de la capa de identidad

**AUTENTICAR**

Sistema nacional de identidad digital del Estado argentino, gestionado por ARCA y ANSES, usado como verificador externo en el contexto argentino. Provee JWTs firmados con RS256 que incluyen el CUIT/CUIL como identificador del ciudadano. El sistema restringe el uso a los proveedores ARCA y ANSES para garantizar un espacio de identificadores consistente.

_Referencia: P-0013, `docs/autenticar.md`._

**circuito ZK**

Implementación en circom de la prueba de conocimiento cero usada por el emisor. Certifica que el `anon_id` fue derivado de un JWT válido de AUTENTICAR y un nonce generado por el emisor, sin revelar ninguno de los dos valores privados. El `anon_id` es el único output público del circuito.

_Referencia: P-0014, P-0015 (Decisión 4)._

**emisor**

Componente de la capa de identidad que verifica el JWT de AUTENTICAR, deriva el `anon_seed`, genera el `anon_id` y la prueba ZK, y entrega la tupla `{pseudónimo, anon_id, prueba ZK}` a la aplicación destino. La entrega ocurre una sola vez, en el momento del registro del ciudadano. A partir de ese momento el emisor no participa en el funcionamiento cotidiano de la aplicación destino.

El emisor almacena únicamente `{anon_seed, fecha_emision}` por cada ciudadano registrado.

_Referencia: P-0015, `design/capa_de_identidad/README.md`._

**JWKS histórico**

Registro de todas las claves públicas que AUTENTICAR publicó, indexadas por `kid`. El emisor lo mantiene para que las pruebas ZK generadas bajo claves ya rotadas sigan siendo verificables. Sin este registro, una rotación de clave por parte de AUTENTICAR haría inverificables las pruebas anteriores.

_Referencia: P-0014._

**prueba ZK**

Prueba criptográfica de conocimiento cero generada por el emisor en cada emisión. Certifica que el `anon_id` fue derivado de un JWT válido de AUTENTICAR, sin revelar la identidad real ni el `anon_seed`. Tiene un tamaño aproximado de 256 bytes y no contiene datos correlacionables con la identidad real.

La aplicación destino la recibe junto con el `anon_id` y el pseudónimo, la verifica al recibirla usando el JWKS histórico, y puede ofrecerla a auditores externos como evidencia de legitimidad del `anon_id`.

_Referencia: P-0014, P-0015 (Decisión 3)._

**trusted setup**

Ceremonia criptográfica requerida para preparar el circuito ZK para uso en producción. Consta de dos fases: la Phase 1 (Powers of Tau) puede reutilizarse de la ceremonia pública perpetua de Hermez/Polygon; la Phase 2 es específica del circuito adaptado y debe realizarse con participantes independientes del equipo operador.

_Referencia: P-0014._

**verificador externo**

Servicio de terceros que el emisor usa para confirmar que el solicitante de una identidad anónima es una persona real. En el contexto argentino el verificador externo es AUTENTICAR, restringido a los proveedores ARCA y ANSES. La capa de identidad asume que el verificador externo puede registrar información sobre los ciudadanos que se autentican; por eso el emisor solicita únicamente los datos mínimos necesarios y no almacena tokens ni respuestas completas.

_Referencia: P-0007, P-0013._

## 4. Ciudadano y participación

**apoyo**

Acción binaria mediante la cual un ciudadano respalda una propuesta. Es retractable: el ciudadano puede retirar su apoyo en cualquier momento, con lo cual el conteo disminuye en uno sin dejar registro visible del retiro. No existe voto negativo ni escala de apoyo. El autor de una propuesta queda automáticamente registrado como un apoyo al momento de publicarla.

_Referencia: P-0012._

**ciudadano**

Persona real verificada que posee una identidad anónima en el sistema. El término opera en dos contextos:

En el contexto de la **capa de identidad**, "ciudadano" refiere a la persona real que interactúa con el emisor para obtener una identidad anónima, presentando sus credenciales de AUTENTICAR.

En el contexto de la **aplicación destino**, "ciudadano" refiere al titular de una identidad anónima activa que participa en la plataforma: crea propuestas, da apoyos y accede mediante su identidad anónima y frase secreta. En este contexto la identidad real del ciudadano no participa.

En ambos contextos se trata de la misma persona física; la distinción es de rol según el componente con el que interactúa.

_Referencia: P-0013, P-0004, `design/capa_de_identidad/identity_model.md`._

**cupo anual**

Límite anual de propuestas publicables por ciudadano en la aplicación destino. Configurable por el operador; el valor por defecto es 2. El valor 0 indica que no existe límite. Las propuestas derivadas consumen cupo en las mismas condiciones que las propuestas originales.

El conteo usa año móvil (ver entrada).

_Referencia: P-0017._

**login**

Acto de autenticarse en la aplicación destino presentando la identidad anónima y la frase secreta. No requiere interacción con el emisor ni con AUTENTICAR: ocurre íntegramente dentro de la aplicación destino. La aplicación normaliza la identidad anónima ingresada para tolerar variaciones tipográficas (mayúsculas, acentos, espacios, guiones).

_Referencia: P-0004, P-0008._

**sesión**

Período de uso autenticado de la aplicación destino. La aplicación no persiste información de la identidad anónima entre sesiones: no precarga la identidad usada anteriormente ni ofrece opciones de tipo "continuar como [identidad]". Este principio refuerza la percepción de anonimato persistente.

_Referencia: P-0005, `design/capa_de_identidad/identity_model.md`._

**tutorial**

Flujo de introducción que el ciudadano atraviesa en su primer ingreso a la aplicación destino. Explica el propósito del sistema, las formas de participación disponibles, el límite anual de propuestas y la recomendación de buscar propuestas existentes antes de crear una nueva.

_Referencia: `docs/propuesta/03_Cómo_se_usaría.md`._

## 5. Propuestas

**propuesta**

Texto estructurado en formato Markdown que un ciudadano publica en la aplicación destino para exponer una idea. Es inmutable tras su publicación: el contenido no puede modificarse. La propuesta no almacena ninguna referencia a su autor en el modelo de datos. Puede declarar vínculos a otras propuestas al momento de publicarse.

_Referencia: P-0018, P-0023 (Decisión 2)._

**propuesta derivada**

Propuesta que declara uno o más vínculos a propuestas existentes de las cuales toma como base conceptual. No es una entidad distinta en el modelo de datos: es una propuesta ordinaria con vínculos declarados. La interfaz puede ofrecer un flujo de creación específico para propuestas derivadas, pero el modelo subyacente es idéntico al de cualquier propuesta con vínculos. Consume cupo anual en las mismas condiciones que cualquier otra propuesta.

_Referencia: P-0017, `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`._

**vínculo**

Referencia genérica sin tipo declarada por una propuesta hacia otra al momento de su publicación. Expresa que existe una relación entre dos propuestas sin calificar su naturaleza. Es inmutable: no puede agregarse ni eliminarse tras la publicación. No requiere aceptación de la propuesta referenciada. Los vínculos forman un grafo dirigido, no un árbol.

_Referencia: `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`._

**vinculación**

Mecanismo por el cual una propuesta declara referencias a otras propuestas al momento de su publicación. Ver también: vínculo, propuesta derivada.

_Referencia: `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md`._

**año móvil**

Método de conteo del cupo anual de propuestas. Cada propuesta publicada ocupa un slot por 365 días contados desde su fecha de publicación. La interfaz muestra al ciudadano la fecha exacta en que cada slot vuelve a estar disponible. Se usa en lugar del año calendario para eliminar la distorsión predecible de fin/inicio de año.

_Referencia: P-0017 (Decisión 2)._

## 6. Moderación y ciclo de vida de propuestas

**catálogo de causales**

Lista configurable de causales que habilitan el retiro de una propuesta. Cada entrada incluye un identificador, un texto corto (usado en el título del tombstone) y un texto largo (usado en el cuerpo del tombstone). El catálogo por defecto incluye "orden judicial" y "contenido manifiestamente ilegal". El operador puede agregar o retirar entradas según el marco legal de su jurisdicción; las causales agregadas deben corresponder a obligaciones legales, no a criterios editoriales.

_Referencia: P-0023 (Decisión 4)._

**causal**

Entrada del catálogo de causales que habilita el retiro de una propuesta. Define el motivo categorizado que se muestra públicamente en el tombstone resultante.

_Referencia: P-0023 (Decisiones 4 y 6)._

**retiro**

Operación excepcional que descarta el contenido de una propuesta publicada y la reemplaza por un tombstone. Solo puede activarse ante causales del catálogo (obligaciones legales), a instancia de requerimiento judicial o por iniciativa del operador ante contenido manifiestamente ilegal. No existe retiro a pedido del autor.

El retiro no es edición: descarta el contenido en lugar de modificarlo. La propuesta retirada conserva únicamente su `id`; el resto se reescribe con los textos del catálogo correspondientes a la causal aplicada.

_Referencia: P-0023._

**revisor de lenguaje**

Componente que analiza el texto de una propuesta antes de su publicación usando la API de moderación de OpenAI (`omni-moderation-latest`). Detecta lenguaje inapropiado (agresivo, discriminatorio, violento, con instrucciones para actividades ilegales) y solicita corrección al ciudadano si lo detecta. Opera sobre la forma del texto, no sobre su contenido ideológico. Toda propuesta debe pasar por el revisor antes de publicarse; si el revisor no está disponible, la publicación se bloquea (fail-closed).

_Referencia: P-0011, P-0022._

**tombstone**

Estado de una propuesta retirada. La propuesta conserva únicamente su `id` (para preservar la integridad referencial de vínculos entrantes) y los textos derivados del catálogo de causales aplicado. El resto del contenido original (título, cuerpo, fecha de publicación, apoyos, vínculos salientes) se elimina o resetea. Los textos del tombstone quedan congelados al momento del retiro; cambios posteriores al catálogo no los afectan.

_Referencia: P-0023 (Decisiones 6 y 7)._

## 7. Ranking y visibilidad

**🌱 emergente**

Señal visual que indica que una propuesta publicada recientemente muestra un ritmo de crecimiento de apoyos por encima del promedio de la plataforma. La ventana de tiempo para considerar una propuesta como "reciente" es configurable por el operador (valor por defecto: 7 días). El umbral de crecimiento se calcula como percentil de la actividad real de la plataforma, lo que lo hace adaptativo sin ajuste manual.

_Referencia: P-0010._

**🔥 tendencia**

Señal visual que indica que una propuesta recibió un crecimiento de apoyos inusualmente alto en las últimas horas. La ventana de tiempo es configurable por el operador (valor por defecto: 48 horas). El umbral se calcula como percentil de la actividad real de la plataforma.

_Referencia: P-0010._

**decaimiento**

Mecanismo por el cual el peso de los apoyos de una propuesta disminuye con la edad de la propuesta en el cálculo del score. Controlado por el parámetro `G` de la fórmula de ranking. Un valor de `G` mayor penaliza más las propuestas antiguas. El decaimiento refleja el propósito del sistema de funcionar como termómetro social del presente, no del pasado.

_Referencia: P-0010._

**multiplicadores (`MT`, `ME`)**

Factores que amplifican el score de una propuesta en la fórmula de ranking cuando cumple las condiciones de tendencia (`MT`, valor por defecto 2.0) o emergente (`ME`, valor por defecto 1.5). Si una propuesta cumple ambas condiciones simultáneamente, se aplican de forma acumulativa (`MT × ME`). Son configurables por el operador.

_Referencia: P-0010._

**relevancia**

Nombre visible en la interfaz del score ponderado de una propuesta. Se muestra en una escala normalizada de 0 a 100 relativa al mayor score del momento. El ciudadano ve la relevancia junto con el conteo real de apoyos; el score interno no se expone directamente.

_Referencia: P-0010._

**score**

Valor numérico calculado por el sistema para ordenar las propuestas en el ranking. Fórmula: `score_final = (apoyos / (edad_en_dias + 1)^G) × multiplicador`, normalizado a una escala de 0 a 100 relativa al mayor score del momento. El ciudadano ve el score normalizado como "relevancia"; el conteo real de apoyos se muestra siempre por separado.

_Referencia: P-0010._

**ventana temporal**

Período de tiempo usado para calcular las señales de tendencia y emergente. Configurable por el operador independientemente para cada señal. Valores por defecto: 48 horas para tendencia, 7 días para emergente.

_Referencia: P-0010._

## 8. Operación y auditoría

**modo debug**

Estado de operación activado explícitamente por el operador mediante un flag de configuración dedicado. Cuando está activo, habilita el registro de campos adicionales en los logs que no se registran en modo operativo normal (por ejemplo: motivo específico de rechazo de login, componentes intermedios del score, parámetros técnicos de la prueba ZK). El modo debug debe ser visible para todos los usuarios del sistema mientras esté activo. Los logs de modo debug se eliminan automáticamente al desactivarlo o tras un máximo de 1 día desde su generación.

_Referencia: P-0020._
