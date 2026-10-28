# Datos iniciales de la demo — composición

Este documento define la composición del conjunto de datos iniciales (seed) que decide DI-0003: qué propuestas contiene, con qué antigüedad, apoyos y vínculos, y qué señales del ranking debe mostrar la demo recién cargada. Todo el contenido es ficticio.

## Convenciones

- `S01` a `S40` son referencias de este documento. El seed asigna a cada propuesta un `id` fijo en el formato de la demo (`identificador_propuestas.md`).
- La **edad** se mide en días hacia atrás desde el momento de la carga (DI-0003, Decisión 2).
- **Apoyos** es el conteo total al momento de la carga. **En 48 h** es cuántos de esos apoyos tienen hora dentro de la ventana de tendencia (P-0029).
- Las **señales esperadas** se calcularon con los valores por defecto de P-0010 y P-0029 (G = 1.0, MT = 2.0, ME = 1.5, ventanas de 48 horas y 7 días, percentil 90, mínimo de actividad de 20).

## Propuestas

| Ref | Título | Tema | Edad | Apoyos | En 48 h | Vínculos | Señal | Qué demuestra |
|---|---|---|---|---|---|---|---|---|
| S01 | Boleto educativo gratuito para estudiantes secundarios y universitarios | Transporte y educación | 320 | 380 | 2 | — | | Apoyo histórico alto con relevancia baja (decaimiento); origen de una cadena de derivadas |
| S02 | Publicación en formato abierto de todos los gastos municipales | Transparencia | 300 | 340 | 3 | — | | Propuesta muy referenciada (5 vínculos entrantes); cita fuentes como texto plano |
| S03 | Actualización trimestral automática de las jubilaciones mínimas | Previsión social | 280 | 310 | 4 | — | | Búsqueda por variantes de "jubilación" |
| S04 | Precio por unidad de medida visible en todas las góndolas | Consumo | 250 | 220 | 30 | — | 🔥 | Resurgimiento de una propuesta antigua |
| S05 | Red de bicisendas conectadas entre barrios | Movilidad | 230 | 190 | 1 | — | | |
| S06 | Turnos en centros de salud por teléfono y en línea, sin filas de madrugada | Salud | 200 | 260 | 25 | — | 🔥 | Resurgimiento de una propuesta antigua |
| S07 | Presupuesto participativo barrial sobre el 5 % del presupuesto municipal | Participación | 170 | 140 | 2 | S02 | | Derivada |
| S08 | Tablero público de seguimiento de obras con plazos y montos | Transparencia | 150 | 95 | 1 | S02 | | Cita fuentes como texto plano |
| S09 | Declaraciones juradas de funcionarios en línea y con buscador | Transparencia | 140 | 120 | 1 | S02 | | |
| S10 | Bibliotecas públicas abiertas los fines de semana | Cultura | 125 | 60 | 0 | — | | |
| S11 | Permitir la venta de medicamentos de venta libre fuera de las farmacias | Salud y consumo | 110 | 85 | 2 | — | | Postura opuesta a S12 |
| S12 | Mantener la venta de medicamentos de venta libre solo en farmacias | Salud | 105 | 90 | 2 | S11 | | Posturas opuestas que coexisten, sin voto negativo |
| S13 | Separación de residuos en origen y puntos verdes en cada barrio | Ambiente | 95 | 110 | 1 | — | | Cuerpo con listas |
| S14 | Prohibir la pirotecnia sonora en zonas urbanas | Convivencia | 85 | 130 | 1 | — | | Postura opuesta a S31 |
| S15 | Huertas comunitarias en terrenos fiscales sin uso | Ambiente y alimentación | 75 | 45 | 0 | — | | |
| S16 | Habilitación simplificada para emprendimientos gastronómicos pequeños | Producción | 65 | 35 | 0 | — | | |
| S17 | *Retirada* (ver abajo) | — | 20 | 0 | 0 | — | | Tombstone con vínculo entrante |
| S18 | Ventanilla única digital para trámites de jubilados y pensionados | Previsión social | 58 | 70 | 1 | S03 | | Derivada; búsqueda por "jubilados" |
| S19 | Iluminación y refugios en todas las paradas de colectivo | Movilidad y seguridad | 52 | 55 | 1 | — | | |
| S20 | Mercados de productores locales los fines de semana | Producción y alimentación | 45 | 40 | 0 | — | | |
| S21 | Protocolo de convivencia para ferias en espacios públicos | Convivencia | 40 | 25 | 0 | S17 | | Vínculo hacia una propuesta retirada |
| S22 | Asesoramiento gratuito a jubilados sobre estafas telefónicas | Previsión social y seguridad | 35 | 28 | 0 | — | | Búsqueda por variantes de "jubilación" |
| S23 | Educación financiera básica en el último año de la secundaria | Educación | 28 | 30 | 1 | — | | |
| S24 | Boleto educativo para estudiantes de nivel terciario | Transporte y educación | 25 | 50 | 2 | S01 | | Segunda generación de la cadena |
| S25 | Consulta pública antes de modificar recorridos de colectivos | Movilidad y participación | 22 | 20 | 0 | S19 | | |
| S26 | Castración gratuita y masiva de perros y gatos | Bienestar animal | 20 | 60 | 3 | — | | Cuerpo con listas |
| S27 | Datos abiertos de calidad del aire en tiempo real | Ambiente y transparencia | 18 | 15 | 0 | S13 | | Cita fuentes como texto plano |
| S28 | Proyecto de ordenanza de acceso a la información pública municipal | Transparencia | 15 | 45 | 2 | S02, S09 | | Texto articulado extenso (unos 8.000 caracteres) con títulos y artículos |
| S29 | Agenda integral de gobierno abierto: presupuesto, obras, declaraciones juradas, datos ambientales, trámites digitales y consulta pública como un solo programa municipal coordinado | Transparencia | 12 | 18 | 1 | S02, S06, S07, S08, S09, S16, S18, S25, S27, S28 | | Máximo de vínculos (10); título cercano al máximo de 200 caracteres |
| S30 | Wifi público gratuito en plazas | Conectividad | 10 | 22 | 1 | — | | Texto breve |
| S31 | Regular la pirotecnia sonora con horarios y zonas en lugar de prohibirla | Convivencia | 9 | 10 | 1 | S14 | | Posturas opuestas que coexisten |
| S32 | Jornada escolar extendida con talleres optativos | Educación | 8 | 28 | 2 | — | | Recién salida de la ventana de 🌱 |
| S33 | Feria de intercambio de útiles y libros escolares | Educación | 6 | 0 | 0 | — | | El autor retiró su apoyo: propuesta con cero apoyos (P-0012) |
| S34 | Tarifa social de servicios para clubes de barrio | Deporte y comunidad | 5 | 42 | 20 | — | 🔥 🌱 | |
| S35 | Boleto educativo para posgrado y formación docente | Transporte y educación | 4 | 12 | 8 | S24 | 🌱 | Tercera generación de la cadena |
| S36 | Paradas a pedido en colectivos nocturnos | Movilidad y seguridad | 3 | 70 | 55 | S19 | 🔥 🌱 | Primera del ranking |
| S37 | Composteras domiciliarias gratuitas y retiro de residuos orgánicos | Ambiente | 2,5 | 32 | 20 | S13 | 🔥 🌱 | |
| S38 | Consulta unificada de jubilaciones y pensiones en un solo sitio | Previsión social | 1,5 | 8 | 8 | S03, S18 | 🌱 | Filtro de vínculos a varias propuestas a la vez |
| S39 | Señalización en braille y relieve en edificios públicos | Accesibilidad | 1 | 3 | 3 | — | | Con 3 apoyos supera en relevancia a S01, que tiene 380 |
| S40 | Vacunatorios con horario extendido | Salud | 0,5 | 1 | 1 | S06 | | Solo el apoyo del autor |

## La propuesta retirada (S17)

El seed publica S17 con una edad de 60 días y le da apoyos. S21 la vincula a los 40 días. A los 20 días antes de la carga, el seed le aplica la operación real de retiro de P-0023 con la causal "orden judicial". Como resultado, S17 conserva solo su `id`; su título y su cuerpo pasan a ser los textos de la causal; su fecha es la del retiro, de modo que muestra 20 días de edad; sus apoyos quedan en cero. El vínculo de S21 hacia S17 se mantiene y lleva al tombstone. El contenido original de S17 es irrelevante porque se descarta; basta con un texto neutro sobre ferias en espacios públicos, coherente con S21.

## Distribución de los apoyos en el tiempo

- El primer apoyo de cada propuesta es el de su autor, con la hora de publicación (P-0012). En S33 no existe porque el autor lo retiró.
- Los apoyos que no caen en las últimas 48 horas se reparten entre la publicación y el inicio de esa ventana con un perfil decreciente: la mayoría en los primeros días después de publicada, cada vez menos después. Es el perfil habitual de interés de una propuesta y hace que el resurgimiento de S04 y S06 se note.
- Los apoyos de las últimas 48 horas se reparten en horas al azar dentro de la ventana.

## Resultado esperado al cargar

- **40 propuestas** (39 activas y 1 retirada), **3.294 apoyos**, de los cuales **205** caen en la ventana de tendencia: muy por encima del mínimo de 20, así que las señales están activas.
- **🔥 tendencia (5):** S36, S37, S34, S04 y S06. El percentil 90 de apoyos nuevos es 20; el valor es el mismo con los dos métodos habituales de cálculo del percentil (rango más cercano e interpolación lineal), así que el resultado no depende de la implementación.
- **🌱 emergente (5):** S36, S37, S34, S38 y S35. El ritmo promedio de la plataforma es 1,96 apoyos por día.
- **Primeras del ranking por relevancia:** S36 (100), S37 (52), S34 (40), S38 (9), S35 (7). Las propuestas antiguas quedan con relevancia entre 1 y 5 aunque tengan cientos de apoyos, y S39, publicada ayer con 3 apoyos, queda por encima de S01, que tiene 380. Es el efecto del decaimiento con G = 1.0 y de los multiplicadores de P-0010.

## Cobertura de la búsqueda y los filtros (P-0019)

| Función | Cómo la ejercita el seed |
|---|---|
| Búsqueda con morfología | S03, S18, S22 y S38 usan "jubilación", "jubilaciones", "jubilados" y "pensionados", el ejemplo de P-0019 |
| Ruido aceptado de los tombstones | Buscar "removida" devuelve S17 (P-0023) |
| Emergente / Tendencia | Las cinco propuestas de cada señal |
| Cantidad de vínculos | Desde 0 (la mayoría) hasta 10 (S29) |
| Vínculos a propuestas específicas | "S02 y S09" devuelve S28 y S29; "S03 y S18" devuelve S38 |
| Rango de fechas | Antigüedades desde medio día hasta 320 días |
| Rango de apoyos | Desde 0 (S17, S33) hasta 380 (S01) |
| Apoyadas / no apoyadas por mí | Se ejercita con las identidades que crean los visitantes |

## Formatos

- S28 es un texto articulado extenso, con títulos y artículos, cercano a un proyecto de ordenanza.
- S13, S26 y otras usan listas y subtítulos de Markdown (P-0018).
- S02, S08 y S27 citan fuentes con URLs del dominio reservado `example.org`, que se muestran como texto no clickeable (P-0018).
- S29 tiene un título cercano al máximo de 200 caracteres.
- El resto son textos breves o medianos, de entre 300 y 2.000 caracteres.

## Equilibrio temático

Los temas cubren movilidad, salud, educación, previsión social, transparencia y participación, ambiente, consumo, producción, convivencia, cultura, bienestar animal, accesibilidad y conectividad, en escalas municipales y generales. Dos pares de posturas opuestas (S11 y S12, S14 y S31) muestran que propuestas contrarias coexisten y compiten por apoyo, sin voto negativo (P-0012). Ningún texto menciona personas, partidos, organizaciones, marcas ni lugares reales.

## Reserva para oleadas

Propuestas que el comando de oleada puede publicar (DI-0003, Decisión 2), en este orden, hasta agotarlas:

1. Bebederos públicos en plazas y paradas de transporte
2. Mapa en línea de rampas y accesos para personas con movilidad reducida
3. Talleres gratuitos de reparación de electrodomésticos y bicicletas
4. Horario nocturno de atención en centros de salud barriales
5. Registro público de árboles urbanos y su mantenimiento
6. Cursos de alfabetización digital para jubilados en bibliotecas públicas
7. Calles de juego sin tránsito los domingos
8. Boleto educativo para acompañantes de estudiantes con discapacidad (vinculada a S01)
