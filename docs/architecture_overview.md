# Architecture Overview

## Propósito

AnonAgora es una plataforma de participación ciudadana con identidad verificada y anonimato persistente.

Su objetivo no es tomar decisiones vinculantes, sino funcionar como un termómetro social que permita observar qué propuestas e ideas comienzan a reunir apoyo dentro de la ciudadanía.

## Principios centrales

- una persona, una única identidad dentro de la plataforma
- separación entre identidad real e identidad anónima
- anonimato persistente
- ausencia de reputación pública visible entre participantes
- evaluación de propuestas por contenido y no por autor
- minimización de correlaciones y metadatos sensibles

## Componentes conceptuales

```text
[ Verificación de identidad ]
        ↓
[ Emisor de identidad anónima ]
        ↓
[ Plataforma participativa ]
```

### 1. Verificación de identidad

Confirma que la persona es un ciudadano real y único usando infraestructura de identidad existente, por ejemplo AUTENTICAR. La identidad real no debe circular dentro de la plataforma participativa.

### 2. Emisor de identidad anónima

Genera una identidad anónima persistente después de la verificación inicial.

Esa identidad:
- no contiene datos personales
- tiene un identificador interno del sistema
- se representa al usuario mediante un pseudónimo amigable
- se recupera exclusivamente mediante una frase secreta definida por el ciudadano durante el registro

### 3. Plataforma participativa

Gestiona:
- exploración de propuestas
- acompañamiento de propuestas
- creación de propuestas
- propuestas derivadas
- ranking por apoyo
- revisión automática del lenguaje
- reglas de participación como límite anual de propuestas

## Identidad visible dentro del sistema

La identidad anónima no es pública frente a otros ciudadanos.

El pseudónimo es visible únicamente para el propio usuario y funciona como representación amigable de su identidad persistente.

## Propuestas

- una propuesta publicada es inmutable
- una mejora o reformulación se expresa mediante una propuesta derivada
- las propuestas compiten por apoyo ciudadano
- el sistema no busca construir reputación pública de autores
