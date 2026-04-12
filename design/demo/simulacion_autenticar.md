# Simulación del flujo de verificación de identidad

## Propósito

Este documento describe cómo la demo simula el flujo de verificación
de identidad que en la plataforma real se delega a AUTENTICAR, la
Plataforma de Autenticación Electrónica Central del Estado argentino.

## Contexto

En la plataforma real, el ciudadano verifica su identidad a través de
AUTENTICAR, que actúa como nexo transparente entre la plataforma y los
proveedores de identidad del Estado. AUTENTICAR no tiene pantalla propia
visible para el ciudadano: recibe la solicitud de la plataforma, redirige
al proveedor elegido, y devuelve el resultado de la autenticación.

Los proveedores disponibles a través de AUTENTICAR son:

- ANSES (proveedor de CUIL)
- ARCA / AFIP (proveedor de CUIT)
- MiArgentina
- ReNaPer (proveedor de DNI)

La demo no puede integrarse con AUTENTICAR ni con ninguno de estos
proveedores reales.

## Decisión de diseño

La demo presenta directamente una pantalla propia con los botones de
selección de proveedor, mostrando el logo oficial de cada uno:

- ANSES
- ARCA / AFIP
- MiArgentina
- ReNaPer

Esta pantalla reproduce la experiencia que tendría el ciudadano en la
plataforma real, donde AUTENTICAR le presentaría los proveedores
disponibles para esa aplicación cliente.

Al seleccionar un proveedor, la demo muestra una pantalla que imita
fielmente la interfaz de autenticación de ese proveedor.

Todas las pantallas de simulación incluyen un banner prominente,
inamovible y claramente visible que indica:

> "⚠️ SIMULACIÓN — Esta es una reproducción de la interfaz real con
> fines demostrativos. No ingresés datos personales reales."

Una vez que el ciudadano completa el flujo simulado, independientemente
del proveedor elegido, la demo considera la verificación exitosa y
continúa con la emisión de la identidad anónima.

## Consideraciones de implementación

El banner de simulación debe estar presente en todas las pantallas del
flujo simulado y no puede ser ocultado ni cerrado por el usuario.

El diseño visual de cada pantalla simulada debe reproducir fielmente
la interfaz real del proveedor correspondiente al momento de la
implementación, para que la demo comunique con precisión cómo sería
la experiencia real.

Ninguna pantalla simulada debe solicitar ni procesar datos reales.
Cualquier dato ingresado por el usuario durante la simulación debe
descartarse sin almacenarse.
