# Registro de pruebas · SnackCheck

**Campaña ejecutada el 25 de octubre de 2026.** Este documento recoge resultados observados, entradas utilizadas, criterios de comprobación y evidencias. Cubre pruebas funcionales, de integración, de error y de rendimiento mediante identificadores `TC-XXX`.

La campaña combina la ejecución del JavaScript original de los nodos Code con peticiones reales al flujo publicado en n8n. Las pruebas aisladas no se presentan como ejecuciones del motor n8n. No se ha modificado el flujo durante las pruebas.

## 1. Resumen de resultados

| Categoría | Casos ejecutados | Aprobados en el alcance indicado | Limitación confirmada | Medidos con incidencia |
| --- | ---: | ---: | ---: | ---: |
| Funcional | 12 | 12 | 0 | 0 |
| Integración | 1 | 1 | 0 | 0 |
| Error | 4 | 3 | 1 | 0 |
| Rendimiento | 3 | 1 | 0 | 2 |
| **Total** | **20** | **17** | **1** | **2** |

- **Aprobado:** el resultado observado coincide con el criterio descrito para ese caso y su alcance.
- **Limitación confirmada:** se ha reproducido un comportamiento problemático conocido; no significa que esté corregido.
- **Medido con incidencia:** se ha ejecutado la carga y registrado sus métricas, pero se han observado respuestas de error para una entrada que previamente funcionaba. No hay un objetivo de latencia acordado para declarar aprobado el rendimiento.

Los 13 casos de nodos Code comprenden **36 variantes diferentes, todas satisfactorias**. La suite HTTP principal contiene **71 peticiones**: 8 de integración y error, 3 de calentamiento, 20 secuenciales, 20 con concurrencia máxima 2 y 20 de rechazo temprano. Además, se hicieron dos comprobaciones iniciales de URLs y una repetición del caso de producto inexistente. Las consultas directas a Open Food Facts se conservan como evidencia de los datos externos.

Los identificadores se mantienen respecto del plan inicial; los saltos de numeración corresponden a casos no incluidos en esta campaña. El alcance real de cada caso se precisa aquí, especialmente para las comprobaciones de NutriScore, que se ejecutaron como código aislado.

## 2. Entorno y trazabilidad

| Elemento | Valor observado o utilizado |
| --- | --- |
| Fecha | 2026-10-25 |
| Hora de inicio de la suite HTTP | 17:27:21, Europe/Madrid; 16:27:21 UTC |
| Versión documental | `0.1.1`, véase [CHANGELOG](../CHANGELOG.md) |
| Flujo fuente | Exportación `SnackCheck`, recibida como `Texto pegado.txt` |
| SHA-256 del archivo fuente usado en pruebas de código | `099b1ec7454db41f0df60526e038a4a8bcea26e4bd5c1b50ca20004f0123c71b` |
| Entorno de los nodos Code aislados | Node.js `v24.21.0`, Windows |
| Webhook de pruebas facilitado | `http://localhost:5678/webhook-test/nutrition-check` |
| Respuesta inicial de esa URL | HTTP 404: webhook no registrado; n8n indicaba activar la escucha |
| Webhook usado en la campaña | `http://localhost:5678/webhook/nutrition-check` |
| Servicios del recorrido HTTP | Flujo publicado con consulta a Open Food Facts y generación del mensaje; sin sustituciones de prueba |
| Modelo en la exportación analizada | Groq `openai/gpt-oss-20b` |
| Versión global de n8n e identificadores internos de ejecución | No capturados; las evidencias HTTP no contienen esos datos |

El hash identifica el archivo analizado, no acredita que la versión publicada en n8n sea idéntica byte a byte. Las respuestas de producción son compatibles con los recorridos documentados, pero no se ha exportado de nuevo la instancia ni inspeccionado sus ejecuciones internas. La configuración del modelo se conoce por el JSON, no por una cabecera de la respuesta HTTP.

### Evidencias

| Referencia | Archivo y contenido |
| --- | --- |
| E01 | [Primera ejecución de código y comprobación de Test URL](evidencias/ejecucion-2026-10-25T16-24-57-284Z.json): 36 variantes y respuesta 404 de la URL de pruebas. |
| E02 | [Pruebas de código y primera respuesta de producción](evidencias/ejecucion-2026-10-25T16-25-30-941Z.json): entradas, resultados esperados y observados, hash y respuesta HTTP 200. |
| E03 | [Campaña HTTP y rendimiento](evidencias/http-2026-10-25T16-27-21-234Z.json): cada petición, body, status, cuerpo de respuesta, fecha y duración, más consulta directa a Open Food Facts. |
| E04 | [Verificación y repetición de producto ausente](evidencias/comprobacion-producto-ausente.json): existencia del primer candidato, ausencia verificada del segundo y respuesta del webhook. |

Son archivos de evidencia real generados durante la campaña. Las credenciales no se incluyen. En E02, localizar `pruebas` por `id`; en E03, localizar `requests` por `id` y `phase`. Los valores de duración originales están expresados en milisegundos, sin redondear.

## 3. Método

### Pruebas de código

Se extrajo el contenido `jsCode` de los dos nodos originales, sin reescribir sus reglas. El programa de pruebas ejecuta esas funciones en Node.js y compara su salida con valores esperados mediante aserciones. Para NutriScore, se proporciona un contexto controlado que representa la referencia a `Obtener - Valores nutricionales`.

Esto comprueba el JavaScript de clasificación y de validación. No comprueba el enlazado de ítems, los nodos Set/IF/Merge, la importación ni las conversiones previas del motor n8n. Los casos TC-010 a TC-013 verifican `clasificacionNutriScore`; no se atribuye a estas pruebas la ejecución del Set que la copia a `valorNutriScore`.

### Pruebas HTTP

Se enviaron peticiones POST con `Content-Type: application/json` a la ruta de producción local. Se capturaron status, tipo de contenido, cuerpo y duración desde el envío hasta la recepción completa. No se usaron datos fijados ni servicios simulados en estas peticiones.

La respuesta satisfactoria se contrastó con una consulta directa a Open Food Facts. Las causas internas de los errores no se deducen solamente del status HTTP: cuando no hay evidencia de la causa, se declara expresamente.

## 4. Pruebas funcionales ejecutadas

Las ternas representan `(azúcar, sal, grasa)` en g/100 g y se introducen como `sugar`, `salt` y `fat` en el nodo Code.

| ID | Entrada y variantes | Resultado esperado | Resultado observado | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- |
| TC-002 | `(5, 0.5, 5)` | Puntuación 0; saludable; 🟢. | Coinciden puntuación, clasificación, emoji y nutrientes devueltos. | Aprobado | E02 |
| TC-003 | `(22.51, 0.5, 5)` | Puntuación 1; moderado; 🟠. | Coincide. | Aprobado | E02 |
| TC-004 | `(5, 1.51, 5)` | Puntuación 1; moderado; 🟠. | Coincide. | Aprobado | E02 |
| TC-005 | `(5, 0.5, 17.51)` | Puntuación 1; moderado; 🟠. | Coincide. | Aprobado | E02 |
| TC-006 | `(23, 2, 5)`, `(23, 0.5, 18)`, `(5, 2, 18)` | Puntuación 2; no saludable; 🔴 en las tres variantes. | Coincide en 3/3. | Aprobado | E02 |
| TC-007 | `(23, 2, 18)` | Puntuación 3; no saludable; 🔴. | Coincide. | Aprobado | E02 |
| TC-008 | `(22.5, 1.5, 17.5)` y tres variantes elevando solo un valor en `0.01`. | Puntuación 0 en la igualdad; 1 al superar un único umbral. | Coincide en 4/4. Confirma comparaciones estrictamente mayores. | Aprobado | E02 |
| TC-009 | `(0, 0, 0)` | Ceros aceptados; puntuación 0; saludable. | Coincide, sin excepción. | Aprobado | E02 |
| TC-010 | NutriScore `a`, `b`. | `clasificacionNutriScore: saludable`. | Coincide en 2/2; se conserva el campo de entrada usado como marcador. | Aprobado | E02 |
| TC-011 | NutriScore `c`. | `clasificacionNutriScore: moderado`. | Coincide. | Aprobado | E02 |
| TC-012 | NutriScore `d`, `e`. | `clasificacionNutriScore: no saludable`. | Coincide en 2/2. | Aprobado | E02 |
| TC-013 | NutriScore `" A "`, `"C"`, `" e "`. | Saludable, moderado y no saludable, respectivamente. | Coincide en 3/3; normaliza mayúsculas y espacios. | Aprobado | E02 |

**Resultado funcional:** 21 variantes satisfactorias en 12 casos. La matriz completa de combinación de clasificaciones y todas las ramas del Merge no se han ejecutado de forma controlada en esta campaña.

## 5. Prueba de integración ejecutada

### TC-021 · Consulta de producto y respuesta final por HTTP

**Entrada:**

```json
{"barcode":8480000505460}
```

La consulta directa realizada durante la campaña devolvió `status: 1`, producto `Havarti light`, marca `MERCADONA`, y estos datos:

| Dato | Valor observado |
| --- | ---: |
| Azúcares por 100 g | 1.2 |
| Sal por 100 g | 0.0115 |
| Grasa por 100 g | 17 |
| NutriScore | `c` |

Con las reglas exportadas, ningún nutriente supera su umbral: el semáforo por nutrientes es saludable. NutriScore `c` se clasifica como moderado. La combinación esperada es **moderado**.

**Resultado observado:** HTTP **200**, `Content-Type: text/html; charset=utf-8`, duración **1.226,28 ms**, cuerpo:

```text
🟠 **NARANJA — moderado**

**Recomendación:** Puedes comprarlo, pero consúmelo con moderación para mantener un equilibrio nutricional.
```

**Estado: Aprobado.** La clasificación, color y recomendación recibidos son coherentes con los datos externos capturados. El alcance es una comprobación de caja negra del recorrido HTTP para ese producto; no se registró la salida interna de cada nodo. Evidencia: E03, solicitud `TC-021` y campo `openfoodfacts`.

La primera comprobación de producción, conservada en E02, también devolvió 200 y moderado, con una redacción distinta y **1.156,60 ms**. No se exige igualdad literal entre textos del modelo.

## 6. Pruebas de error ejecutadas

| ID | Entrada y procedimiento | Resultado esperado o criterio | Resultado observado | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- |
| TC-023 | POST con `{}` y con `{"barcode":null}`. | HTTP 400, JSON con código `BARCODE_NO_EXIST`, mensaje obligatorio y timestamp. | 2/2 respuestas 400 con el JSON previsto. | Aprobado | E03 |
| TC-024 | POST con barcode string numérico, `"abc"`, `true` y `{}`. | Caracterizar el fallo de tipo del validador estricto y comprobar si existe una respuesta controlada. | 4/4 respuestas HTTP 500 con `{"message":"Error in workflow"}`; no se obtiene el error 400 personalizado. | Limitación confirmada | E03 |
| TC-025 | Verificar ausencia de `987654321012345` en Open Food Facts y enviar ese número al webhook. | Tras confirmar `status: 0`, respuesta de producto inexistente HTTP 400. | Fuente: HTTP 404 y `status: 0`. Webhook: HTTP 400 con el mensaje previsto. | Aprobado, tras corregir el dato de prueba | E04 |
| TC-028 | En cada nutriente, probar negativo, `null`, cadena vacía, campo omitido y `"abc"`; mantener los otros dos válidos. Ejecutar el Code aislado. | Excepción `Faltan nutrientes o sus valores no son válidos`. | La excepción coincide en 15/15 variantes. No se mide HTTP en este caso. | Aprobado | E02 |

### Trazabilidad de la corrección de TC-025

El primer candidato fue `9999999999999`, que devolvió HTTP 200 y moderado. El comprobador automático de E03 lo marcó como fallido porque lo había supuesto inexistente. La consulta independiente posterior de E04 demostró que **sí existe** en Open Food Facts (`Salatgurke`, `status: 1`, NutriScore `unknown`). Por tanto, aquella expectativa era incorrecta y no demuestra un defecto del flujo.

Se conserva el resultado original sin alterarlo. Se repitió la prueba con `987654321012345`, cuya ausencia se verificó antes de invocar el webhook. El estado final de TC-025 corresponde a esa repetición válida. El fallo automático inicial se excluye del recuento de defectos por ser un error en la preparación del dato de prueba.

## 7. Pruebas de rendimiento ejecutadas

Las mediciones usan servicios reales y el producto `8480000505460`. Son una observación de una campaña breve, no un ensayo de capacidad máxima. No se ha acordado un SLA ni se ha medido CPU, memoria o latencia de cada servicio por separado.

| ID | Escenario | Respuestas observadas | Mediana | p95 | Resultado |
| --- | --- | --- | ---: | ---: | --- |
| TC-033 | 3 peticiones de calentamiento excluidas; después 20 secuenciales, concurrencia 1. | 8 HTTP 200 y 12 HTTP 400; 40 % de éxito en las 20 medidas. | 826,30 ms | 1.426,69 ms | Medido con incidencia |
| TC-034 | 20 peticiones en 10 parejas, concurrencia máxima 2. | 1 HTTP 200 y 19 HTTP 400; 5 % de éxito. | 988,78 ms | 988,78 ms | Medido con incidencia |
| TC-035 | 20 peticiones secuenciales con body `{}`. | 20 HTTP 400 con `BARCODE_NO_EXIST`, como se esperaba. | 88,35 ms | 137,29 ms | Aprobado en rechazo funcional; latencia registrada sin SLA |

Para TC-033 y TC-034, las latencias de la tabla corresponden **solo a las respuestas HTTP 200**: 8 muestras y 1 muestra, respectivamente. El p95 con una única muestra no permite caracterizar la distribución. Para TC-035 se usan las 20 respuestas 400 esperadas. Los errores no se ocultan: se contabilizan por separado y sus duraciones completas permanecen en E03.

Se calcula p95 ordenando las duraciones y tomando la posición `ceil(0.95 × N)`, contando desde 1. Las tres peticiones de calentamiento de TC-033 devolvieron 200 y no se incorporan a sus métricas.

### Incidencia observada durante las repeticiones

Las **31 respuestas HTTP 400** de TC-033 y TC-034 contenían:

```text
Upps! Parece que el  producto no existe en nuestra base de datos.
Comprueba el código de barras o inténtalo con otro producto
```

Se trata del mismo barcode para el que se había obtenido información de Open Food Facts y respuestas 200 del webhook. La conclusión sustentada es que **el recorrido no mantuvo una respuesta satisfactoria al repetir las consultas bajo las condiciones de esta campaña**.

No se puede atribuir la causa a un límite de llamadas, a Groq, a Open Food Facts o a un nodo concreto sin inspeccionar las ejecuciones de n8n. El mensaje recibido no distingue un producto inexistente de otros fallos de consulta. Las dos cargas se ejecutaron consecutivamente y comparten el estado de las dependencias; por ello, tampoco se puede atribuir la caída de éxito exclusivamente a la concurrencia.

No se hicieron más ensayos de carga después de estas mediciones. Para una repetición, primero deben revisarse las ejecuciones fallidas y los límites de los servicios, y espaciar las peticiones según esos límites.

## 8. Incidencias y acciones derivadas

| ID | Evidencia observada | Impacto | Acción propuesta, no implementada |
| --- | --- | --- | --- |
| INC-001 | TC-024: cuatro entradas de tipo incorrecto producen HTTP 500. | El cliente recibe un fallo interno ante una entrada inválida; además, el ejemplo string del error contradice el validador. | Validar el formato explícitamente y devolver un 400 controlado; alinear el ejemplo con el contrato elegido. |
| INC-002 | TC-033/034: 31 rechazos de un barcode que había funcionado en la misma campaña. | Respuesta inestable y mensaje de producto inexistente que no permite identificar la causa. | Revisar ejecuciones internas; distinguir producto ausente de error externo; ajustar tratamiento de errores y frecuencia de llamadas según el diagnóstico. |
| OBS-001 | La URL `/webhook-test/` devolvió 404 sin escucha registrada. | No se podía ejecutar repetidamente por esa URL. | Se usó la ruta de producción local ya activa; no fue necesario modificar o activar el flujo. |

## 9. Alcance no acreditado

Esta campaña no demuestra cobertura exhaustiva. No se han probado de forma controlada todos los casos del plan original: importación en otra instancia, todos los campos ausentes desde la API, todas las variantes de `"null"`, las nueve combinaciones de clasificaciones, las cuatro entradas del Merge, fallos de credencial Groq ni interrupciones de red provocadas.

Tampoco se ha comprobado la ausencia de llamadas externas en TC-035 mediante trazas internas, aunque la exportación sitúa la respuesta de entrada ausente antes de esos nodos. Los casos no ejecutados no forman parte del resumen de 20 casos y no se presentan como aprobados.

## 10. Reproducción

Se adjuntan dos comprobadores:

- [verificar-snackcheck.cjs](../tests/verificar-snackcheck.cjs): ejecuta las 36 variantes sobre el código de la exportación indicada; no necesita n8n si se omite la URL opcional.
- [verificar-http.cjs](../tests/verificar-http.cjs): ejecuta la campaña HTTP y guarda las respuestas. Su ejecución consume llamadas a los servicios del flujo; revisar primero las incidencias detectadas y los límites disponibles.

Desde la raíz del repositorio, con Node.js disponible:

```bash
node tests/verificar-snackcheck.cjs workflows/SnackCheck.json
node tests/verificar-http.cjs http://localhost:5678/webhook/nutrition-check
```

Los datos externos y la redacción del modelo pueden cambiar. Cada repetición genera evidencia con fecha nueva; debe interpretarse sin sobrescribir ni reinterpretar como nueva la evidencia de esta campaña.

## 11. Conclusión de la campaña

Las reglas de los dos nodos Code superaron las 36 variantes ejecutadas. El flujo publicado completó una consulta real con una recomendación coherente, y se verificaron las respuestas de entrada ausente y de producto inexistente.

La campaña también confirmó el HTTP 500 para tipos de barcode incompatibles y detectó errores frecuentes al repetir consultas. Por tanto, queda acreditado el funcionamiento de los casos indicados, pero **no la robustez bajo carga ni una validación completa del flujo**. Las incidencias se mantienen abiertas y deben tratarse como resultados del ensayo, no como pruebas pendientes ni como correcciones ya realizadas.

