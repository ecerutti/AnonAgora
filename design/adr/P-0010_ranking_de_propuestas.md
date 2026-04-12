# P-0010 — Ranking y visualización de propuestas

## Contexto

La plataforma necesita determinar en qué orden se presentan las propuestas
a los ciudadanos y qué información se muestra de cada una en la lista
principal.

Esta decisión afecta directamente qué ideas reciben visibilidad y, por lo
tanto, qué propuestas tienen mayor probabilidad de acumular apoyo.

El orden de presentación es uno de los mecanismos más influyentes del
sistema: una propuesta visible acumula más apoyos, y una propuesta con más
apoyos se vuelve más visible. Un algoritmo de ranking mal diseñado puede
distorsionar la señal social que la plataforma busca capturar.

El propósito central de la plataforma es funcionar como un termómetro social
del presente, no del pasado. Una propuesta puede haber acumulado miles de
apoyos en el pasado pero haber perdido relevancia porque el contexto social
cambió, porque el problema que planteaba fue atendido, o porque los
decisores la ignoraron y el interés ciudadano se desplazó hacia otras
preocupaciones. El sistema debe reflejar el interés ciudadano actual, no
el histórico.

## Opciones consideradas

### Opción 1 — Conteo total de apoyos

Las propuestas se ordenan únicamente por la cantidad acumulada de apoyos,
de mayor a menor.

**Ventajas**

- Simple de implementar y de explicar al ciudadano.
- Transparente: el orden refleja directamente el apoyo recibido.

**Desventajas**

- Genera el efecto "rich get richer": las propuestas que llegaron primero
  o tuvieron un impulso inicial acumulan ventaja permanente e inamovible.
- Las propuestas nuevas con alto potencial quedan enterradas detrás de
  propuestas antiguas con mucho acumulado histórico.
- No refleja si una propuesta sigue siendo relevante en el presente o si
  su apoyo ocurrió hace años.

### Opción 2 — Decaimiento temporal puro

El peso de cada apoyo decrece con el tiempo según una función de
decaimiento. Los apoyos recientes valen más que los históricos. El orden
resulta exclusivamente de ese cálculo.

**Ventajas**

- Evita que propuestas antiguas monopolicen el ranking indefinidamente.
- Refleja mejor la relevancia actual de las propuestas.

**Desventajas**

- Puede hacer que propuestas con apoyo genuino y sostenido pierdan
  visibilidad solo por ser antiguas.
- No distingue entre propuestas que están creciendo activamente y
  propuestas que simplemente son recientes.

### Opción 3 — Velocidad de crecimiento (momentum)

Las propuestas se ordenan por el ritmo al que están acumulando apoyos en
un período reciente, independientemente de su acumulado histórico.

**Ventajas**

- Hace visibles ideas emergentes con alto interés actual.
- Muy sensible a cambios en el interés ciudadano.

**Desventajas**

- Favorece propuestas recientes sobre propuestas con apoyo sostenido a
  largo plazo.
- Inestable: el orden puede cambiar drásticamente en horas.
- Una propuesta con poco acumulado pero un pico puntual puede desplazar
  propuestas con apoyo genuino y sostenido.

### Opción 4 — Score ponderado combinado con señales visuales

El orden por defecto se calcula con una fórmula que combina el acumulado
histórico de apoyos con un factor de decaimiento temporal y multiplicadores
para propuestas en tendencia o emergentes:

```
score_base = apoyos / (edad_en_dias + 1)^G
multiplicador = 1.0
si tendencia: multiplicador *= MT
si emergente: multiplicador *= ME
score_final = score_base * multiplicador
```

El score final se normaliza a una escala de 0 a 100 relativa, donde 100
corresponde siempre a la propuesta con mayor score en ese momento. Esta
normalización se recalcula cada vez que se actualiza el ranking.

El uso de multiplicadores proporcionales al score base garantiza que el
efecto de los bonus sea relativo al contexto actual de la plataforma, sin
depender de valores absolutos que pierdan sentido a medida que la
plataforma crece o decrece en participación.

Adicionalmente, el sistema calcula dos señales complementarias que se
muestran visualmente sobre cada propuesta:

- 🔥 **Tendencia:** propuestas cuyo crecimiento de apoyos en las últimas
  48 horas se encuentra en el percentil superior de actividad de la
  plataforma en ese período.
- 🌱 **Emergente:** propuestas publicadas hace menos de 7 días que
  muestran un ritmo de crecimiento por encima del promedio de la
  plataforma.

Los umbrales de tendencia y emergente se calculan como percentiles de la
actividad real de la plataforma, lo que los hace adaptativos al volumen
de participación sin necesidad de ajuste manual.

**Ventajas**

- El ranking por defecto balancea relevancia histórica y actualidad.
- Los multiplicadores son proporcionales: su efecto es consistente tanto
  en plataformas con poco uso como en plataformas maduras con miles de
  propuestas y apoyos.
- Cada señal responde una pregunta distinta: la relevancia responde "¿qué
  propuestas generan interés ciudadano ahora?", 🔥 responde "¿qué está
  pasando en este momento?", 🌱 responde "¿qué idea nueva está despertando
  interés?".

**Desventajas**

- Mayor complejidad de implementación respecto a un conteo simple.
- El factor de decaimiento y los multiplicadores requieren calibración
  inicial y revisión periódica por parte del operador.

## Decisión

Se adopta la **Opción 4 — Score ponderado combinado con señales visuales**.

### Fórmula de ranking

```
score_final = (apoyos / (edad_en_dias + 1)^G) * multiplicador
```

El score final se normaliza a una escala de 0 a 100 relativa al mayor
score del momento.

### Parámetros configurables

| Parámetro | Descripción | Valor por defecto |
|-----------|-------------|-------------------|
| `G` | Factor de gravedad del decaimiento temporal | `1.0` |
| `MT` | Multiplicador para propuestas en tendencia | `2.0` |
| `ME` | Multiplicador para propuestas emergentes | `1.5` |
| `ventana_tendencia` | Ventana de tiempo para calcular tendencia | `48 horas` |
| `ventana_emergente` | Ventana de tiempo para considerar una propuesta emergente | `7 días` |

Cuando una propuesta cumple simultáneamente la condición de tendencia y
emergente, se aplican ambos multiplicadores de forma acumulativa
(`MT * ME`).

### Información visible por propuesta en la lista principal

Cada propuesta muestra las siguientes columnas:

- **Relevancia:** score normalizado de 0 a 100. Al posar el cursor sobre
  el valor o sobre el encabezado de la columna, aparece el siguiente
  texto explicativo:

  > "La relevancia refleja el interés ciudadano actual de cada propuesta.
  > Combina la cantidad de apoyos recibidos con qué tan reciente es esa
  > actividad. Una propuesta con menos apoyos puede tener mayor relevancia
  > si está generando interés ahora, porque la plataforma busca mostrar
  > lo que le importa a la ciudadanía hoy."

- **Apoyos:** cantidad total de apoyos recibidos, siempre el valor real
  sin ponderación.

- **🔥 Tendencia:** visible cuando la propuesta cumple el criterio de
  tendencia. Al posar el cursor aparece el texto:

  > "Esta propuesta está en tendencia: está recibiendo un crecimiento
  > de apoyos inusualmente alto en las últimas 48 horas."

- **🌱 Emergente:** visible cuando la propuesta cumple el criterio de
  emergente. Al posar el cursor aparece el texto:

  > "Esta propuesta es emergente: fue publicada recientemente y ya está
  > generando un interés por encima del promedio."

El orden por defecto al ingresar a la plataforma es por Relevancia
descendente. El ciudadano puede modificar el criterio de ordenamiento
en cualquier momento.

## Justificación

El conteo simple de apoyos produce un sesgo estructural hacia propuestas
antiguas que distorsiona la señal social que la plataforma busca capturar.
Un ranking que solo mide velocidad de crecimiento es inestable y no
reconoce el apoyo sostenido.

El decaimiento temporal no es únicamente un mecanismo técnico: es una
decisión de diseño alineada con el propósito de la plataforma. Las
propuestas envejecen porque la realidad social cambia. Una propuesta que
acumuló miles de apoyos en el pasado puede haber perdido vigencia porque
el contexto cambió, porque el problema fue atendido, o porque los
decisores la ignoraron y el interés ciudadano se desplazó. Mostrar esa
propuesta en el tope del ranking distorsionaría la señal que la plataforma
ofrece a legisladores, gobiernos e investigadores sociales.

Se consideró la posibilidad de hacer G, MT y ME dinámicos, ajustándose
automáticamente según la edad o el volumen de participación de la
plataforma. Esta alternativa fue descartada porque introduce cambios
silenciosos en el ranking que el ciudadano no puede anticipar ni
comprender, afectando la transparencia y previsibilidad del sistema. La
adaptabilidad necesaria se delega en su lugar a los umbrales percentiles
de los íconos, que son adaptativos por naturaleza sin afectar el orden
principal.

Los valores por defecto de G=1.0, MT=2.0 y ME=1.5 fueron validados
mediante simulaciones con escenarios representativos de plataformas
jóvenes y maduras, verificando que las tendencias y emergentes alcancen
visibilidad sin desplazar de forma injustificada propuestas con apoyo
sostenido.

La normalización del score a una escala de 0 a 100 permite mostrar la
relevancia al ciudadano de forma comprensible, evitando exponer valores
absolutos sin unidades ni contexto intuitivo.

## Consecuencias

- El sistema calcula y actualiza periódicamente el score de cada
  propuesta, incluyendo la normalización relativa.
- Los parámetros de cálculo deben documentarse en la guía de operación
  de la plataforma.
- El operador puede ajustar G al momento del despliegue según la etapa
  de la plataforma, por ejemplo usando un valor menor durante los primeros
  meses si considera que el decaimiento penaliza demasiado el historial
  corto.
- La interfaz debe mostrar las cuatro columnas definidas con sus
  respectivos tooltips. Los tooltips son obligatorios, no opcionales.
- Los apoyos visibles para el ciudadano reflejan siempre el conteo real,
  nunca el score ponderado interno.
- Una propuesta puede aparecer con ambos íconos simultáneamente si cumple
  los dos criterios al mismo tiempo.
