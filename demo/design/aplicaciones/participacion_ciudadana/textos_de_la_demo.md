# Textos de la demo

Este documento completa, para la demo, los textos de plataforma de `design/aplicaciones/participacion_ciudadana/textos_para_el_ciudadano.md`: asigna los valores de los parámetros, agrega los bloques propios de la demo y define el aviso global de contenido ficticio (DI-0003, Decisión 4).

Los T&C de la demo reproducen los que tendría un despliegue real, para que quien la prueba vea cómo serían. Un recuadro al inicio aclara que son un ejemplo y que no constituyen un acuerdo legal. Los T&C no identifican a un operador ni un canal de contacto, porque no son datos relevantes para el ciudadano.

## Valores de los parámetros

La demo usa los valores de producción de la plataforma (DP-0002).

| Parámetro | Valor en la demo |
|---|---|
| `{verificadores}` | ARCA o ANSES, a través de AUTENTICAR (en la demo, simulados) |
| `{cupo_anual}` | 2 |
| `{cooldown}` | 6 meses |
| `{expiracion_sesion}` | 1 hora |
| `{proveedor_revisor}` | OpenAI |
| `{retencion_proveedor_revisor}` | 30 días |
| `{retencion_logs}` | 7 días |
| `{retencion_backups}` | 30 días |
| `{catalogo_de_causales}` | Las dos causales por defecto: orden judicial y contenido manifiestamente ilegal |
| `{tc_version}` | 1 |
| `{tc_vigencia}` | La fecha de la primera publicación de la demo |

## Bloques propios de la demo

### `{bloque_del_despliegue_tutorial}`

Se agrega al final de la pantalla 6 del tutorial:

> **Esto es una demostración.** La verificación de identidad es simulada y las propuestas y los apoyos iniciales son ficticios. No escribas datos personales reales.

### `{bloque_del_despliegue_tc_inicio}`

Recuadro destacado, antes de la sección 1, visible sin necesidad de desplazarse:

> **Esta es una demostración.** Estos términos son un ejemplo de los que tendría un despliegue real de la plataforma y no constituyen un acuerdo legal. Todo el contenido inicial es ficticio. No ingreses datos personales reales.

### `{bloque_del_despliegue_tc_fin}`

Sección agregada después de la 10:

> ### 11. Sobre esta demostración
>
> - **La verificación de identidad es simulada.** Las pantallas de ARCA y ANSES son una reproducción y aceptan CUIL ficticios que empiezan con 99. Por eso una misma persona puede obtener varias identidades, y los conteos de apoyos no representan personas reales.
> - **Las propuestas iniciales y sus apoyos son ficticios**, y se agregan periódicamente apoyos ficticios para mostrar cómo se mueve el ranking.
> - **No escribas datos personales reales.** Las copias de seguridad de la base de datos se conservan 30 días.
> - **El revisor de lenguaje es real:** el texto de tus propuestas se envía a OpenAI como se describe en la sección 4.
> - **La demo puede reiniciarse sin aviso**, y en ese caso se pierden las identidades, propuestas y apoyos creados.
> - La demo se ofrece como está, sin garantías de disponibilidad ni de funcionamiento.

## Aviso global de contenido ficticio

DI-0003 (Decisión 4) exige un aviso permanente en todas las pantallas de la aplicación. Texto:

> Demostración: las propuestas iniciales y su actividad son ficticias.

La ubicación y la forma visual del aviso se definen en la guía visual de la demo.

El aviso es distinto del banner de simulación de AUTENTICAR (`demo/design/capa_de_identidad/simulacion_autenticar.md`), que aparece solo en las pantallas de verificación simuladas.

## Referencias

- DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo
- DP-0002 — Alcance funcional del MVP de la demo
- DI-0002 — Persistencia y despliegue de la demo
- DI-0003 — Datos iniciales de la demo
- `design/aplicaciones/participacion_ciudadana/textos_para_el_ciudadano.md` — Textos de plataforma
- `demo/design/capa_de_identidad/simulacion_autenticar.md` — Simulación del flujo de verificación
