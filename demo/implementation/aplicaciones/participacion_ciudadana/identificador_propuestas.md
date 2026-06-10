# Identificador de propuestas — demo

Para la demo, el `id` de las propuestas (P-0018) se genera en **Crockford Base32**, con **4 caracteres siempre en mayúsculas** y prefijo `#`. Ejemplo: `#8KF2`.

El formato responde al alcance demostrativo: es corto, legible y fácil de dictar o de escribir a mano al declarar vínculos entre propuestas. Crockford Base32 excluye los caracteres ambiguos (I, L, O, U), reduciendo errores de lectura y tipeo, y 32⁴ = 1.048.576 identificadores posibles son holgados para una demo.

Esta es una decisión de implementación específica de la demo; un despliegue productivo deberá evaluar su propio esquema de identificadores. Si la decisión se revisara formalmente contra alternativas, correspondería un ADR `DI-XXXX` en esta carpeta.
