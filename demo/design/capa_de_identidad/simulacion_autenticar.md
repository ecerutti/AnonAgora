# Simulación del flujo de verificación de identidad

## Propósito

Este documento describe cómo la demo simula el flujo de verificación de identidad que en el sistema en producción se delega a AUTENTICAR, la Plataforma de Autenticación Electrónica Central del Estado argentino.

## Contexto

En el sistema en producción, el ciudadano verifica su identidad a través de AUTENTICAR, que actúa como intermediario entre el emisor y los proveedores de identidad del Estado. AUTENTICAR presenta al ciudadano una pantalla propia donde elegir el proveedor, completa la autenticación contra ese proveedor, y devuelve el resultado al emisor.

AUTENTICAR ofrece múltiples proveedores de identidad, pero el emisor acepta únicamente los que comparten el espacio de identificador CUIT/CUIL por razones de unicidad (ver P-0013):

- ANSES (proveedor de CUIL)
- ARCA / AFIP (proveedor de CUIT)

La demo no puede integrarse con AUTENTICAR ni con estos proveedores reales.

## Decisión de diseño

La demo presenta directamente una pantalla propia con los botones de selección de proveedor, mostrando el logo oficial de cada uno:

- ANSES
- ARCA / AFIP

Esta pantalla reproduce la experiencia que tendría el ciudadano en el sistema en producción, donde AUTENTICAR le presentaría estos mismos proveedores.

Al seleccionar un proveedor, la demo muestra una pantalla que imita fielmente la interfaz de autenticación de ese proveedor.

Todas las pantallas de simulación incluyen un banner prominente, inamovible y claramente visible que indica:

> "⚠️ SIMULACIÓN — Esta es una reproducción de la interfaz real con fines demostrativos. No ingresés datos personales reales."

Una vez que el ciudadano completa el flujo simulado, independientemente del proveedor elegido, la demo considera la verificación exitosa y continúa con la emisión de la identidad anónima.

## Consideraciones de implementación

El banner de simulación debe estar presente en todas las pantallas del flujo simulado y no puede ser ocultado ni cerrado por el usuario.

El diseño visual de cada pantalla simulada debe reproducir fielmente la interfaz real del proveedor correspondiente al momento de la implementación, para que la demo comunique con precisión cómo sería la experiencia real.

Ninguna pantalla simulada debe solicitar ni procesar datos reales. Cualquier dato ingresado por el usuario durante la simulación debe descartarse sin almacenarse.

## Referencias

- P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad
- DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo
