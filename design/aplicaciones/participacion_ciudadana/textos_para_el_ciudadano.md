# Textos para el ciudadano

Este documento especifica los textos que la aplicación de participación ciudadana muestra al ciudadano en el registro (F-CI-01 y F-AP-01), en el onboarding (tutorial y términos y condiciones, `notas/inventario_de_flujos_ui_ux.md`, F-AP-01 y H-2) y en las propuestas retiradas (catálogo de causales, P-0023). Son textos de plataforma: valen para cualquier despliegue.

## Convenciones

- Los textos usan español rioplatense con voseo, como la demo (`demo/design/aplicaciones/participacion_ciudadana/i18n_y_a11y.md`). Un despliegue en otra variante del español los adapta sin cambiar su contenido.
- Lo que va entre llaves, como `{cupo_anual}`, es un **parámetro del despliegue**: cada despliegue lo completa con sus valores. La lista está en la sección "Parámetros".
- `{bloque_del_despliegue_*}` marca un lugar donde el despliegue puede insertar texto propio. Si no lo usa, el lugar queda vacío.
- El vocabulario sigue `design/glosario.md`. En particular, al ciudadano se le habla de su **identidad anónima**, no de su "pseudónimo", salvo cuando hay que distinguir el nombre de la frase secreta (glosario, entrada "pseudónimo").
- Los textos no prometen anonimato absoluto (`AGENTS.md`, "Promesas de anonimato"). Cada afirmación sobre privacidad tiene que poder sostenerse con un ADR.

## Parámetros

| Parámetro | Qué es | Origen |
|---|---|---|
| `{verificadores}` | Servicios con los que se verifica la identidad real | P-0007, P-0013 |
| `{cupo_anual}` | Propuestas por ciudadano cada 365 días (default 2) | P-0017 |
| `{cooldown}` | Plazo mínimo entre identidades de una misma persona (default 6 meses) | P-0015 |
| `{expiracion_sesion}` | Inactividad que cierra la sesión (default 1 hora) | P-0005 |
| `{proveedor_revisor}` | Servicio externo del revisor de lenguaje | P-0011 |
| `{retencion_proveedor_revisor}` | Cuánto conserva ese servicio los textos recibidos | P-0011 |
| `{retencion_logs}` | Retención de los registros técnicos (default 7 días) | P-0020 |
| `{retencion_backups}` | Retención de las copias de seguridad de la base | Despliegue |
| `{catalogo_de_causales}` | Lista de causales de retiro vigentes, con su texto corto | P-0023 |
| `{tc_version}`, `{tc_vigencia}` | Número y fecha de vigencia de los T&C | F-AP-01 |
| `{frase_min_palabras}`, `{frase_min_caracteres}` | Mínimos de la frase secreta (default 4 palabras y 20 caracteres) | P-0008 |
| `{pseudonimo}` | El pseudónimo generado para el ciudadano (no es un parámetro del despliegue) | P-0002, P-0003 |

Si `{cupo_anual}` es 0 (sin límite), se omiten las frases que mencionan el límite.

## Registro

Tres pantallas del registro explican la identidad anónima en el momento en que el ciudadano la necesita. Las dos primeras las muestra el emisor de identidades (F-CI-01, pasos 1 y 8) y la tercera, la aplicación (F-AP-01, paso 2). El tutorial vuelve sobre estos temas solo como repaso.

### Inicio del registro — Cómo vas a participar

Para participar necesitás una **identidad anónima**. Se crea en tres pasos:

1. **Verificás que sos una persona real** con {verificadores}. Así cada persona tiene una sola identidad.
2. **Recibís un nombre**, como "Lobo Azul 714". Es tu identidad anónima: tu nombre dentro de la plataforma, sin vínculo con tu nombre real.
3. **Elegís una frase secreta**, que funciona como tu contraseña.

Funciona como la entrada a un edificio: en la puerta, un guardia controla tu documento y te da una pulsera con un número que no lleva tu nombre. Adentro participás con esa pulsera, y el guardia se queda afuera.

Para ingresar a la plataforma vas a usar tu identidad anónima y tu frase secreta. La plataforma no guarda tu nombre ni tu número de CUIT o CUIL.

Una vez creada tu identidad anónima, vas a poder obtener otra recién después de {cooldown}.

### Identidad anónima — Esta es tu identidad anónima

**{pseudonimo}**

Si no te gusta, podés generar otra. Cuando la aceptes, ya no se puede cambiar.

- **Recordala.** La vas a necesitar, junto con tu frase secreta, cada vez que ingreses.
- Solo vos la vas a ver: no aparece en tus propuestas ni en tus apoyos.
- No la compartas ni la anotes. Es la mitad de lo que te permite ingresar.

### Frase secreta — Elegí tu frase secreta

Tu frase secreta es como una contraseña, pero más larga y más fácil de recordar: una oración de al menos {frase_min_palabras} palabras y {frase_min_caracteres} caracteres que tenga sentido para vos. Por ejemplo, "mi perro come pizza los jueves" (no uses este ejemplo).

- **Elegí una frase que puedas recordar sin anotarla.** Si la anotás, quien encuentre la nota puede usar tu identidad.
- Intentá no usar frases conocidas, como refranes, letras de canciones o citas, ni datos tuyos que otras personas que te conozcan puedan adivinar.
- No uses una frase que ya uses en otros sitios.

**Campos:** "Frase secreta" y "Repetí tu frase secreta". Cada campo tiene un botón **Mostrar** / **Ocultar** para ver lo que escribiste.

**Si las dos no coinciden:** "Las frases no coinciden. Revisalas con el botón Mostrar."

**Si la perdés, no hay forma de recuperarla.** Nadie puede restablecerla. Vas a poder obtener una identidad nueva recién {cooldown} después de haber obtenido esta, y no vas a poder volver a usar esta.

## Tutorial

Seis pantallas cortas. Cada una tiene un título y un texto. El ciudadano avanza con "Siguiente" y puede volver a la anterior. No hay botón para saltear (F-AP-01).

### Pantalla 1 — Un termómetro social

Esta plataforma es un espacio para que la ciudadanía proponga ideas, acompañe las de otras personas y vea qué propuestas reúnen más apoyo.

### Pantalla 2 — Apoyar

La forma más simple de participar es apoyar propuestas que ya existen.

- Cada propuesta se apoya con un solo botón. No hay votos negativos ni puntajes (estrellitas).
- Podés retirar tu apoyo cuando quieras.
- La cantidad de apoyos de cada propuesta es pública, pero no se puede ver quienes dieron su apoyo.

### Pantalla 3 — Crear una propuesta

Antes de crear una propuesta, buscá si alguien ya planteó algo parecido. Muchas veces la mejor forma de participar es sumar tu apoyo a una idea que ya existe.

- Podés publicar hasta {cupo_anual} propuestas cada 365 días. Cada propuesta ocupa un lugar durante 365 días desde que la publicás.
- Una propuesta publicada no se puede editar ni borrar. Revisala bien antes de publicar.
- Si querés mejorar o continuar una idea existente, podés crear una propuesta nueva vinculada a la original.
- Un revisor automático controla el lenguaje (insultos, agresiones), no las ideas. Si encuentra un problema, te lo indica antes de publicar.

### Pantalla 4 — Cómo leer el ranking

Las propuestas se ordenan por **relevancia**. La propuesta más relevante se muestra primero. Junto a la relevancia siempre ves la cantidad real de apoyos.

- 🔥 **Tendencia:** está reuniendo muchos apoyos en las últimas 48 horas.
- 🌱 **Emergente:** se publicó hace menos de 7 días y suma apoyos más rápido que el promedio.

Cuando hay poca actividad en la plataforma, estas dos señales no se muestran.

### Pantalla 5 — Tu identidad

- Tus propuestas se publican **sin autor**: no muestran tu identidad anónima ni ningún dato tuyo.
- Para ingresar necesitás tu identidad anónima y tu frase secreta. Solo vos ves tu identidad anónima; no la compartas.
- **Tu frase secreta no se puede recuperar.** Nadie puede restablecerla. Si la perdés, vas a poder obtener una identidad nueva recién {cooldown} después de haber obtenido esta.
- La sesión se cierra sola después de {expiracion_sesion} sin actividad, sin aviso previo. Lo que estabas escribiendo y no publicaste se pierde.
- Cuando escribas una propuesta, evitá detalles personales que te hagan reconocible.

### Pantalla 6 — Antes de empezar

Para terminar, te vamos a pedir que leas y aceptes los términos y condiciones. Ahí se explica con más detalle qué datos se guardan, qué límites tiene la protección de tu identidad y en qué casos una propuesta puede retirarse.

{bloque_del_despliegue_tutorial}

## Términos y condiciones

### Encabezado

**Términos y condiciones** — versión {tc_version}, vigente desde {tc_vigencia}.

{bloque_del_despliegue_tc_inicio}

### 1. El servicio

Esta plataforma permite a las personas publicar propuestas y apoyar las de otras personas sin mostrar su identidad real. Es un termómetro social: sus resultados no son vinculantes ni obligan a ninguna autoridad.

### 2. Quién puede participar

Participan personas que verificaron su identidad real con {verificadores}. Con esa verificación obtienen una **identidad anónima**, un nombre como "Lobo Azul 714" sin vínculo con su nombre real, y eligen una frase secreta para ingresar con ella.

### 3. Tu identidad anónima y tu frase secreta

- Tu pseudónimo y tu frase secreta son tu credencial. Quien los conozca puede participar en tu nombre.
- La frase secreta no se guarda en la plataforma y **no se puede recuperar ni restablecer** por ningún medio.
- Una identidad anónima no se puede dar de baja, ni a tu pedido ni por decisión de quien administra la plataforma. Si obtenés una identidad nueva después de {cooldown}, la anterior no se anula.
- Cuidar tu credencial es tu responsabilidad.

### 4. Reglas de participación

- Cada identidad puede apoyar una vez cada propuesta y retirar su apoyo cuando quiera.
- Cada identidad puede publicar hasta {cupo_anual} propuestas cada 365 días.
- **Las propuestas publicadas no se pueden editar ni borrar**, tampoco a pedido de quien las escribió. Para continuar o mejorar una idea se crea una propuesta nueva vinculada.
- Antes de publicar, un revisor automático evalúa el lenguaje de la propuesta. No evalúa las ideas. Si el revisor no está disponible, la publicación no es posible hasta que vuelva a estarlo.
- **El texto de cada propuesta se envía, antes de publicarse, a un servicio externo de moderación ({proveedor_revisor})**, que lo conserva hasta {retencion_proveedor_revisor}. Solo se envía el texto de la propuesta, sin datos de tu identidad.

### 5. Contenido que no está permitido

Sos responsable de lo que publicás. No publiques:

- contenido ilegal según la ley aplicable;
- amenazas contra personas concretas;
- datos personales de otras personas;
- contenido que infrinja derechos de terceros, como derechos de autor.

El revisor automático controla la forma del lenguaje. Que una propuesta lo supere no significa que su contenido sea lícito.

### 6. Retiro de propuestas

No hay moderación editorial: nadie revisa ni aprueba las propuestas. Una propuesta publicada solo puede retirarse por estas causales legales:

{catalogo_de_causales}

El retiro lo ejecuta quien administra la plataforma ante una orden judicial, o por iniciativa propia ante contenido manifiestamente ilegal. Cuando se retira una propuesta, se eliminan su contenido, sus apoyos y los vínculos que declaraba. En su lugar queda un aviso con la causal, que conserva su número para que los vínculos de otras propuestas sigan funcionando.

### 7. Qué datos se guardan

**Al verificar tu identidad**, el componente que emite las identidades anónimas guarda un valor derivado de tu CUIT o CUIL, que no permite recuperar el número. Los usa solo para que cada persona tenga una sola identidad por vez. Ese componente no guarda nada que permita vincular la identidad anónima con vos.

**En la plataforma** se guarda:

- tu identidad anónima y un valor derivado de tu frase secreta que permite verificarla, pero no la frase;
- el día en que te registraste, si completaste el tutorial y qué versión de estos términos aceptaste;
- tus propuestas, **sin indicar quién las escribió**;
- para controlar el límite de propuestas, el día en que publicaste cada una, sin indicar cuál publicaste;
- tus apoyos, **sin tu identidad**;
- registros técnicos de funcionamiento, sin tu dirección IP ni tu identidad anónima, que se borran a los {retencion_logs}.

Las copias de seguridad de la base de datos se conservan {retencion_backups}.

### 8. Límites de la protección de tu identidad

La plataforma está diseñada para separar tu identidad real de lo que hacés en ella. Esa protección tiene límites que tenés que conocer:

- **El servicio de verificación** ({verificadores}) puede saber que te verificaste para usar esta plataforma. No recibe información sobre lo que hacés en ella.
- **Lo que escribís podría identificarte** si dejas detalles personales o información que le permita a alguien reconocerte.
- **Cuando hay poca actividad**, tu participación se distingue menos de la de los demás. Las protecciones que dependen de mezclar tu actividad con la de otras personas son más débiles en esos momentos.
- **Tu dispositivo y tu conexión** quedan fuera del alcance de la plataforma. Quien tenga acceso a tu dispositivo o a tu navegador, o tu proveedor de internet, puede saber que usás este sitio.
- **El diseño no protege frente a que todos los responsables del sistema se pongan de acuerdo** para identificarte, ni frente a alguien que controle por completo su infraestructura.
- **Modo de diagnóstico.** Si quien administra la plataforma activa este modo para resolver un problema técnico, todas las pantallas muestran un aviso visible, los registros técnicos son más detallados y la plataforma te pide confirmación antes de cada acción.
- **Frente a un requerimiento legal**, quien administra la plataforma puede retirar una propuesta según la sección 6. La plataforma no guarda quién escribió cada propuesta.

### 9. Disponibilidad

El servicio puede no estar disponible en algunos momentos. La sesión se cierra sola después de {expiracion_sesion} sin actividad, sin aviso previo, y se pierde lo que no hayas publicado.

### 10. Cambios en estos términos

Si estos términos cambian, la próxima vez que ingreses te vamos a mostrar el texto nuevo. Para seguir participando vas a tener que aceptarlos. Si no los aceptás, no vas a poder participar, pero tus propuestas y tus apoyos anteriores se mantienen.

{bloque_del_despliegue_tc_fin}

### Botones

**Acepto** · **No acepto**

## Mensajes del onboarding

| Momento | Texto |
|---|---|
| Rechazo de los T&C | Para participar es necesario aceptar los términos y condiciones. Tu identidad anónima ya está creada: cuando quieras, volvé a ingresar con tu identidad anónima y tu frase secreta y te los vamos a mostrar de nuevo. |
| Encabezado ante una versión nueva | Actualizamos los términos y condiciones. Para seguir participando, leelos y aceptalos. |

## Catálogo de causales de retiro por defecto

P-0023 define dos causales por defecto. El título del aviso que reemplaza a la propuesta se arma como "Propuesta removida por" seguido del texto corto. El texto largo es el cuerpo del aviso. La fecha que muestra la propuesta retirada pasa a ser la del retiro.

### Orden judicial

- **Identificador:** `orden_judicial`
- **Texto corto:** orden judicial
- **Texto largo:** Esta propuesta fue retirada para cumplir una orden judicial recibida por quien administra la plataforma. Su contenido, sus apoyos y los vínculos que declaraba fueron eliminados y no se conservan en el sistema. El retiro no es una evaluación de las ideas que planteaba. Las propuestas que la mencionaban siguen publicadas. La fecha que se muestra es la del retiro.

### Contenido manifiestamente ilegal

- **Identificador:** `contenido_manifiestamente_ilegal`
- **Texto corto:** contenido manifiestamente ilegal
- **Texto largo:** Esta propuesta fue retirada por quien administra la plataforma porque su contenido era manifiestamente ilegal según la ley aplicable; por ejemplo, amenazas contra personas concretas, datos personales de terceros o infracciones a derechos de autor. Su contenido, sus apoyos y los vínculos que declaraba fueron eliminados y no se conservan en el sistema. El retiro no es una evaluación de las ideas que planteaba. Las propuestas que la mencionaban siguen publicadas. La fecha que se muestra es la del retiro.

## Referencias

- P-0001 — Visibilidad pública de la identidad anónima
- P-0002 — Representación de identidades anónimas mediante pseudónimos amigables
- P-0003 — Selección del pseudónimo de identidad anónima
- P-0004 — Autenticación de identidad anónima
- P-0005 — Gestión de sesiones de usuario
- P-0006 — Modelo de amenazas y supuestos de confianza
- P-0008 — Mecanismo de credencial de acceso: passphrase vs password
- P-0009 — Recepción y almacenamiento de la frase secreta en la plataforma
- P-0010 — Ranking y visualización de propuestas
- P-0011 — Revisor automático de lenguaje en propuestas
- P-0012 — Mecanismo de apoyo a propuestas
- P-0015 — Modelo de datos del emisor y ciclo de vida de identidades anónimas
- P-0016 — Invalidación de identidades anónimas en la plataforma participativa
- P-0017 — Límite anual de propuestas por ciudadano
- P-0018 — Modelo de datos de propuestas
- P-0020 — Política de logs y retención de metadatos
- P-0022 — Comportamiento ante fallos de servicios externos y componentes críticos
- P-0023 — Moderación de contenido y retiro de propuestas
- P-0027 — Actividad y expiración de la sesión
- P-0028 — Visibilidad del pseudónimo durante la sesión
- P-0029 — Registro de apoyos desvinculado de la identidad y señales temporales del ranking
- `design/aplicaciones/participacion_ciudadana/vinculacion_de_propuestas.md` — Vinculación entre propuestas
- `notas/inventario_de_flujos_ui_ux.md` — F-AP-01 (onboarding) y H-2