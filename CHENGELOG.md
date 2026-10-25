# Changelog · SnackCheck

Registro de cambios del proyecto. Las versiones siguen el formato `MAYOR.MENOR.PARCHE` de [Semantic Versioning 2.0.0](https://semver.org/lang/es/).

La versión `0.1.0` se asigna como referencia inicial del flujo analizado y de esta documentación. No procede de una etiqueta previa de Git ni implica que se haya publicado una release, desplegado el flujo o superado pruebas. La fecha de esta primera entrada corresponde a la preparación de la documentación; no acredita la fecha de desarrollo original del flujo.

## [Sin publicar]

No hay cambios adicionales documentados sobre la referencia `0.1.0`. Añadir aquí los cambios que se realicen antes de preparar la siguiente versión. Las mejoras pendientes no deben registrarse como implementadas.

## [0.1.0] - 2026-10-25

### Añadido

- README en español con el inventario de los 26 nodos, las conexiones, las reglas de clasificación, las respuestas HTTP, la importación y las limitaciones de la exportación SnackCheck.
- Registro de 35 casos de prueba con identificadores `TC-001` a `TC-035`: 16 funcionales, 6 de integración, 10 de error y 3 de rendimiento. Todos pendientes de ejecución, con resultados esperados y campos para registrar evidencia.
- Este CHANGELOG y una política de versionado para los cambios posteriores.

### Funcionalidad existente documentada

Las siguientes capacidades ya están presentes en el JSON revisado. Esta entrada las establece como referencia inicial; no afirma que se hayan programado o modificado en esta fecha.

- Webhook `POST nutrition-check`, con lectura de `body.barcode` y validación inicial de existencia numérica estricta.
- Consulta de productos mediante la API v2 de Open Food Facts y extracción de azúcares, sal y grasa total por 100 g.
- Semáforo según el número de nutrientes que superan los umbrales: azúcar `> 22.5`, sal `> 1.5` y grasa `> 17.5` g/100 g.
- Conversión de NutriScore: `a`/`b` a `saludable`, `c` a `moderado`, `d`/`e` a `no saludable` y cualquier otro valor normalizado al string `"null"`.
- Combinación de clasificaciones: ambas saludables → saludable; alguna no saludable → no saludable; resto → moderado. Si se alcanza la rama del string `"null"`, se conserva la clasificación por nutrientes.
- Generación de una recomendación mediante Groq con el modelo `openai/gpt-oss-20b`.
- Respuesta final configurada con HTTP 200 y tres respuestas HTTP 400 para las ramas previstas de entrada ausente, producto inexistente y datos insuficientes.

### Limitaciones conocidas

- El ejemplo de barcode del error está escrito como string, mientras que la condición inicial espera un número; no hay validación completa de formato EAN/UPC.
- El control previo exige que exista `nutriscore_grade`; por ello, su ausencia o JSON `null` no equivale a alcanzar la rama posterior del string `"null"`.
- El nodo denominado «¿Ambos datos son NO saludable?» aplica OR: basta con una clasificación no saludable.
- La comprobación de errores de Open Food Facts espera un objeto; otros tipos pueden provocar una excepción. Los fallos de consulta no cuentan con un mensaje específico diferenciado del de producto inexistente.
- Los errores de extracción, del nodo Code y de Groq no disponen de recuperación específica. El texto final no se valida con un esquema estructurado.
- No se han acreditado pruebas funcionales o de integración, mediciones de rendimiento ni compatibilidad con una versión global concreta de n8n.

### Validación

- Documentación basada en la revisión estática del JSON y sus conexiones.
- Sin cambios funcionales realizados al flujo durante la preparación de estos documentos.
- Sin resultados de ejecución inventados: consultar el [registro de pruebas](docs/registro-pruebas.md) y completar sus evidencias al ejecutarlas.

## Política de versionado

El contrato público de referencia se describe en el [README](README.md): endpoint, body, reglas de clasificación y respuestas. La serie `0.y.z` indica desarrollo inicial y no garantiza una API estable. Una versión publicada debe conservar su contenido; los cambios posteriores se registran en una versión nueva. [Especificación SemVer](https://semver.org/lang/es/).

Para este proyecto se adopta la siguiente convención durante el desarrollo inicial:

| Cambio | Criterio de versión |
| --- | --- |
| Corrección compatible o ajuste documental sin cambios de comportamiento | Incrementar el parche: por ejemplo, `0.1.0` → `0.1.1`. |
| Nueva funcionalidad o cambio de contrato durante `0.y.z` | Incrementar la versión menor y detallar expresamente cualquier incompatibilidad: por ejemplo, `0.1.0` → `0.2.0`. |
| Primera API declarada estable | Publicar `1.0.0` cuando el contrato esté definido y su validación esté documentada. |

A partir de `1.0.0`, aplicar los incrementos de SemVer: parche para correcciones compatibles, menor para nuevas funcionalidades compatibles y mayor para cambios incompatibles. Los ejemplos anteriores explican la política; no representan versiones existentes.

## Cómo mantener este archivo

1. Anotar en **Sin publicar** cada cambio realizado, usando categorías como Añadido, Cambiado, Corregido o Eliminado según corresponda.
2. Separar correcciones implementadas de limitaciones pendientes y vincular la evidencia de pruebas disponible.
3. Al preparar una versión, trasladar esos cambios a una entrada con su número SemVer y fecha real. Mantener las entradas más recientes primero.
4. Añadir enlaces a commits, comparaciones o releases únicamente cuando existan. El `versionId` interno de n8n no sustituye al número SemVer del proyecto.
