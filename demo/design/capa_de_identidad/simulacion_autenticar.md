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

Esta pantalla reproduce la experiencia que tendría el ciudadano en el sistema en producción, donde AUTENTICAR le presentaría estos mismos proveedores. Como referencia visual sirven los selectores de proveedores de los portales estatales que usan AUTENTICAR, por ejemplo el de la Provincia de Buenos Aires (`https://sso.gba.gob.ar/web/login/W3B1`), que muestra varios proveedores de los cuales la demo solo usa ANSES y ARCA.

Al seleccionar un proveedor, la demo muestra una pantalla que imita la apariencia de la interfaz de autenticación de ese proveedor, con una diferencia deliberada: **el único campo es el CUIL**. No hay campo de clave ni de ningún otro dato. Así, aunque alguien confunda la pantalla con la real, no tiene dónde ingresar su clave.

El campo de CUIL viene precargado con un CUIL ficticio generado al azar con prefijo `99-`, un prefijo que no se asigna a CUIT/CUIL reales. La demo rechaza cualquier CUIL que no tenga ese prefijo, de modo que es imposible ingresar un CUIL real. El ciudadano puede reemplazar el CUIL precargado por uno ficticio que ya haya usado; por ejemplo, para ver el rechazo por cool-down (DP-0002, Decisión 2).

Todas las pantallas de simulación incluyen un banner prominente, inamovible y claramente visible que indica:

> "⚠️ SIMULACIÓN — Esta es una reproducción de la interfaz real con fines demostrativos. No ingresés datos personales reales."

Una vez que el ciudadano completa el flujo simulado, independientemente del proveedor elegido, la demo considera la verificación exitosa y continúa con la emisión de la identidad anónima.

## Consideraciones de implementación

El banner de simulación debe estar presente en todas las pantallas del flujo simulado y no puede ser ocultado ni cerrado por el usuario.

El diseño visual de cada pantalla simulada debe reproducir la apariencia de la interfaz real del proveedor correspondiente al momento de la implementación, para que la demo comunique con precisión cómo sería la experiencia real, con la excepción del campo de clave, que se omite.

El único dato que ingresa al flujo simulado es el CUIL ficticio. El emisor lo usa exactamente como en producción: deriva el `anon_seed` con el salt del sistema, descarta el CUIL y persiste solo `{anon_seed, fecha_emision}` (P-0015). Ningún otro dato se solicita ni se almacena.

## Referencias

- P-0013 — Integración con AUTENTICAR como proveedor de verificación de identidad
- P-0015 — Qué almacena el emisor
- DP-0001 — Omisión de ZK y simulación de AUTENTICAR en la demo
- DP-0002 — Alcance funcional del MVP de la demo
