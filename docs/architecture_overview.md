# Architecture Overview

## Propósito

Este proyecto desarrolla un sistema modular para habilitar participación ciudadana con identidad anónima verificada. El sistema se compone de dos partes (P-0021):

- una **capa de identidad anónima verificada** reutilizable, que se encarga de verificar que cada participante es una persona real y de emitir identidades anónimas persistentes;
- una **aplicación destino** construida sobre esa capa, que define qué pueden hacer los ciudadanos con sus identidades anónimas.

La primera aplicación destino diseñada es una **plataforma de participación ciudadana**. Su objetivo no es tomar decisiones vinculantes, sino funcionar como un termómetro social que permita observar qué propuestas e ideas comienzan a reunir apoyo dentro de la ciudadanía. El modelo permite construir otras aplicaciones destino sobre la misma capa (por ejemplo, un sistema de denuncias anónimas) como despliegues independientes; un ciudadano que quiera participar en dos aplicaciones distintas debe registrarse por separado en cada una.

## Principios centrales

- un ciudadano real, una identidad anónima activa por aplicación destino
- separación entre identidad real e identidad anónima
- anonimato persistente
- ausencia de reputación pública visible entre participantes
- evaluación de propuestas por contenido y no por autor
- minimización de correlaciones y metadatos sensibles

## Componentes del sistema

```text
┌─────────────────────────────────────────┐
│           Capa de identidad             │
│                                         │
│   [ Verificación de identidad ]         │
│              ↓                          │
│   [ Emisor de identidad anónima ]       │
│                                         │
└──────────────────┬──────────────────────┘
                   │  pseudónimo, anon_id, prueba ZK
                   ↓
┌─────────────────────────────────────────┐
│           Aplicación destino            │
│   (ej: plataforma participativa)        │
└─────────────────────────────────────────┘
```

### Capa de identidad

La capa encapsula la integración con un verificador externo y la emisión de identidades anónimas. Sus componentes son:

**Verificación de identidad.** Confirma que la persona es un ciudadano real y único usando infraestructura de identidad existente. En el contexto argentino, P-0013 adopta AUTENTICAR. La identidad real nunca circula hacia la aplicación destino.

**Emisor de identidad anónima.** Genera, después de la verificación, una identidad anónima persistente que el ciudadano usará dentro de la aplicación destino. Esa identidad:

- no contiene datos personales
- tiene un identificador interno del sistema (`anon_id`)
- se representa al usuario mediante un pseudónimo amigable
- se entrega a la aplicación destino acompañada de una prueba criptográfica (ZK) que certifica su legitimidad sin revelar la identidad real (P-0014)

La capa garantiza una identidad activa por ciudadano por aplicación destino, sujeta a un período de cool-down entre emisiones (P-0015). El contrato completo entre la capa y la aplicación destino vive en `design/capa_de_identidad/README.md`.

### Aplicación destino

Una aplicación destino recibe del emisor el pseudónimo, el `anon_id` y la prueba ZK al momento de la emisión, y a partir de ahí opera de forma independiente de la capa. Cada aplicación destino define:

- el mecanismo de credencial de acceso del ciudadano (cómo se autentica en login posteriores)
- la gestión de sesiones
- el modelo de datos del comportamiento del ciudadano dentro de la aplicación
- las reglas funcionales propias de la aplicación

## Aplicación de participación ciudadana

Es la primera aplicación destino diseñada. Sus funciones principales:

- exploración de propuestas
- acompañamiento (apoyo) de propuestas
- creación de propuestas
- propuestas derivadas
- ranking por apoyo
- revisión automática del lenguaje
- reglas de participación como el límite anual de propuestas

### Identidad visible dentro de la aplicación

La identidad anónima no es pública frente a otros ciudadanos (P-0001). El pseudónimo es visible únicamente para el propio usuario y funciona como representación amigable de su identidad persistente.

### Propuestas

- una propuesta publicada es inmutable
- una mejora o reformulación se expresa mediante una propuesta derivada
- las propuestas compiten por apoyo ciudadano
- el sistema no busca construir reputación pública de autores

### Reingreso

El reingreso del ciudadano a la aplicación se realiza mediante pseudónimo + frase secreta (P-0004, P-0008). La frase secreta es definida por el ciudadano y es asunto exclusivo de la aplicación; el emisor no participa en su manejo.

## Verificación de identidad como servicio externo

La capa de identidad debe asumir que cualquier servicio externo de verificación puede registrar información sobre los ciudadanos que se autentican. Por esta razón:

- el emisor solicita únicamente los datos mínimos necesarios para verificar elegibilidad
- los identificadores del proveedor externo no se utilizan como identidad interna
- no se almacenan tokens ni respuestas completas de autenticación (P-0013)
- la identidad anónima se genera de forma independiente del proveedor (P-0015)
- la legitimidad de cada identidad emitida es auditable criptográficamente sin necesidad de confiar en el operador del emisor (P-0014)

## Documentos relacionados

- `AGENTS.md` — principios del proyecto y estructura del repositorio.
- `design/capa_de_identidad/README.md` — contrato entre la capa de identidad y las aplicaciones destino.
- `design/capa_de_identidad/identity_model.md` — modelo detallado de identidad: pseudónimos, `anon_seed`, `anon_id`.
- `design/threat_model.md` — modelo de amenazas del sistema.
- `design/adr/` y subcarpetas — Architecture Decision Records.
- `docs/propuesta/` — documentación conceptual orientada a lectores no técnicos.
