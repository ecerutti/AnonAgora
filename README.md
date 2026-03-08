# AnonAgora 🏛️

**Plataforma de Participación Ciudadana Digital con Identidad Anónima Persistente.**

> “Escuchar mejor, sin miedo, sin exposición y sin intermediarios innecesarios.”

## ¿Qué problema intenta resolver?

En muchos países, una parte significativa de la ciudadanía evita participar en debates o propuestas públicas por temor a quedar expuesta políticamente o sufrir represalias sociales, laborales o institucionales.

El resultado es conocido:  
participación baja, opiniones autocensuradas y una percepción pública distorsionada.

**AnonAgora explora una posible solución técnica a este problema.**

## ¿Qué es AnonAgora?

AnonAgora es una propuesta de infraestructura democrática diseñada para **complementar y fortalecer la democracia representativa tradicional**. Permite a los ciudadanos proponer ideas, reformas o peticiones de manera directa, garantizando un **anonimato real y persistente** frente al poder político y a los administradores del sistema.

La idea central es simple:

Un sistema donde un ciudadano pueda **demostrar que es una persona real**, pero donde **todas sus acciones dentro de la plataforma permanezcan anónimas**, incluso frente a:

- el gobierno  
- los administradores del sistema  
- los propios desarrolladores  

El objetivo es crear un **termómetro social confiable**, capaz de mostrar qué propuestas generan apoyo real dentro de la sociedad.

## Características principales

* **Identidad Validada, Participación Anónima:** El sistema verifica que el usuario es un ciudadano real mediante servicios oficiales (por ejemplo ANSES, ARCA/AFIP o MiArgentina). Una vez validado, la identidad real queda separada de las acciones dentro de la plataforma.

* **Anonimato Persistente:** Cada ciudadano recibe una identidad anónima estable que le permite:
    * votar propuestas
    * apoyar iniciativas
    * modificar su postura con el tiempo
    * seguir su historial de participación

* **Calidad sobre Cantidad:** Se establece un límite anual de propuestas por ciudadano para reducir duplicaciones y fomentar iniciativas más elaboradas.

* **Debate Constructivo:** Un asistente de redacción basado en IA ayuda a mantener un lenguaje institucional y respetuoso sin intervenir en el contenido ideológico de las propuestas.

## 🛠️ Estructura del repositorio

Este repositorio contiene la documentación conceptual y estratégica del proyecto.

`/docs` – Documentación principal  
- [`01_El_Problema_y_la_Idea.md`](./docs/01_El_Problema_y_la_Idea.md) – problema y visión
- [`02_Fundamentos_y_Lógica_de_funcionamiento.md`](./docs/02_Fundamentos_y_Lógica_de_funcionamiento.md) – lógica del sistema
- [`03_Cómo_se_usaría.md`](./docs/03_Cómo_se_usaría.md) – experiencia de uso
- [`04_Riesgos_límites_y_preocupaciones_razonables.md`](./docs/04_Riesgos_límites_y_preocupaciones_razonables.md) – objeciones políticas
- [`05_Cómo_podría_implementarse.md`](./docs/05_Cómo_podría_implementarse.md) – plausibilidad técnica
- [`06_Arquitectura_técnica_y_desafíos.md`](./docs/06_Arquitectura_técnica_y_desafíos.md) – arquitectura y riesgos técnicos
- [`00_Resumen_ejecutivo.md`](./docs/00_Resumen_ejecutivo.md)

`/notas` – ideas en desarrollo, analogías y material de trabajo.

En el futuro este repositorio podría incluir también **prototipos o implementaciones de referencia** de la plataforma.

## Contribuir

AnonAgora es un proyecto abierto a colaboración interdisciplinaria.

Buscamos especialmente:
* **Especialistas en criptografía y ciberseguridad:** para auditar los mecanismos de anonimato (por ejemplo Zero-Knowledge Proofs).
* **Desarrolladores:** para explorar prototipos o MVP del sistema.
* **Investigadores en políticas públicas y gobernanza digital:** para evaluar su posible aplicación institucional.

## Licencia

Distribuido bajo licencia [Apache 2.0.](./LICENSE)
