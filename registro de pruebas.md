# Registro de pruebas · SnackCheck

Este documento reúne los casos de prueba del flujo n8n SnackCheck y el espacio para registrar sus ejecuciones. Cubre pruebas **funcionales, de integración, de error y de rendimiento**, con identificadores `TC-XXX`.

**Estado inicial: todos los casos están pendientes de ejecución.** Los resultados esperados se deducen de la exportación revisada y del [README](../README.md); no son resultados observados. No se han realizado llamadas al webhook ni a los servicios externos para preparar este registro.

## Control del documento

| Campo | Valor |
| --- | --- |
| Fecha de preparación | 2026-10-25 |
| Versión de referencia | `0.1.0`, asignada como base documental en el [CHANGELOG](../CHANGELOG.md) |
| Flujo de referencia | Exportación `SnackCheck`, recibida como `Texto pegado.txt` |
| Commit o hash del JSON ejecutado | Pendiente de registrar |
| Responsable de ejecución | Pendiente de registrar |
| Versión de n8n | Pendiente de registrar |
| Entorno y URL del webhook | Pendiente de registrar |
| Modelo configurado | Groq `openai/gpt-oss-20b` |

## Resumen de ejecución

| Categoría | Casos definidos | Ejecutados | Aprobados | Fallidos | Bloqueados | Pendientes |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Funcional | 16 | 0 | 0 | 0 | 0 | 16 |
| Integración | 6 | 0 | 0 | 0 | 0 | 6 |
| Error | 10 | 0 | 0 | 0 | 0 | 10 |
| Rendimiento | 3 | 0 | 0 | 0 | 0 | 3 |
| **Total** | **35** | **0** | **0** | **0** | **0** | **35** |

Algunos casos incluyen varias variantes. El resumen cuenta identificadores, no peticiones ni variantes. Un caso parametrizado solo se aprueba cuando se han ejecutado y aprobado todas sus variantes; conserva el detalle de cada una.

Estados permitidos:

- **Pendiente:** aún no ejecutado.
- **Aprobado:** ejecución respaldada por evidencia y conforme al resultado esperado.
- **Fallido:** ejecución completada con una diferencia frente al resultado esperado.
- **Bloqueado:** no se puede completar por falta de entorno, credenciales u otra dependencia; registrar el motivo.

Una prueba de caracterización puede aprobarse al confirmar una limitación conocida. Eso **no significa que el defecto esté corregido**. Los casos exploratorios sin criterio de aceptación deben conservar sus mediciones y quedar pendientes de evaluación, sin presentarse como aprobados.

## Preparación y procedimiento

1. Importa una copia del flujo y configura su credencial Groq. Registra la versión de n8n, el commit o hash del JSON y la URL de pruebas.
2. Para pruebas por HTTP, pon el Webhook a la escucha y envía un POST con `Content-Type: application/json`. Guarda código HTTP, cabeceras, cuerpo, duración e identificador de ejecución de n8n.
3. Para pruebas de reglas, utiliza datos controlados en esa copia. Puedes fijar la salida de `API - Consultamos información de barcode` para ejecutar manualmente el tramo posterior. Mantén el nodo `Obtener - Valores nutricionales` y su nombre: el cálculo de NutriScore lo referencia directamente.
4. Las pruebas aisladas de un nodo Code o IF no acreditan una respuesta HTTP. Identifícalas como pruebas de nodo y registra su entrada y salida, sin atribuirles un estado HTTP que no se haya observado.
5. Las pruebas con datos fijados no acreditan una integración real. Para probar el recorrido completo por HTTP con datos simulados, prepara una copia con una fuente de respuestas controladas y registra la modificación. No des por hecho que los datos fijados del editor se aplican a las ejecuciones de producción.
6. Para las integraciones reales, retira los datos fijados y las sustituciones de prueba. Conserva la respuesta de Open Food Facts observada: sus datos pueden cambiar.
7. Rellena la ficha de ejecución al final del documento, actualiza el estado de la tabla y el resumen. Guarda evidencias sin API keys, cabeceras de autorización ni otros secretos.

### Datos controlados de referencia

Respuesta simulada de Open Food Facts, creada exclusivamente para las pruebas:

```json
{
  "status": 1,
  "product": {
    "product_name": "Producto sintético de prueba",
    "brands": "Marca de prueba",
    "nutriscore_grade": "a",
    "nutriments": {
      "sugars_100g": 5,
      "salt_100g": 0.5,
      "fat_100g": 5
    }
  }
}
```

No representa un producto real ni una consulta ejecutada. En las tablas, `(azúcar, sal, grasa)` expresa gramos por 100 g. Para probar el nodo de semáforo de forma aislada, utiliza las claves `sugar`, `salt` y `fat` con esos valores.

## Pruebas funcionales

Para estos casos se verifican reglas deterministas. La redacción del modelo se comprueba por separado en integración.

| ID | Caso y entrada | Procedimiento | Resultado esperado según el flujo | Resultado real / evidencia | Estado |
| --- | --- | --- | --- | --- | --- |
| TC-001 | Barcode numérico: `{"barcode":8480000505460}`. | Ejecutar Webhook, primer IF y nodo que guarda barcode. | IF true y `barcode` almacenado como string. No se presupone que el producto exista. | Sin ejecutar / — | Pendiente |
| TC-002 | Ningún nutriente alto: `(5, 0.5, 5)`. | Ejecutar `Calcular - Semáforo saludable`. | `puntuacion: 0`, `clasificacion: "saludable"`, `semaforo: "🟢"`. | Sin ejecutar / — | Pendiente |
| TC-003 | Solo azúcar alto: `(22.51, 0.5, 5)`. | Ejecutar el cálculo de semáforo. | Puntuación 1, `moderado`, 🟠. | Sin ejecutar / — | Pendiente |
| TC-004 | Solo sal alta: `(5, 1.51, 5)`. | Ejecutar el cálculo de semáforo. | Puntuación 1, `moderado`, 🟠. | Sin ejecutar / — | Pendiente |
| TC-005 | Solo grasa alta: `(5, 0.5, 17.51)`. | Ejecutar el cálculo de semáforo. | Puntuación 1, `moderado`, 🟠. | Sin ejecutar / — | Pendiente |
| TC-006 | Dos altos: `(23, 2, 5)`, `(23, 0.5, 18)` y `(5, 2, 18)`. | Ejecutar las tres variantes. | Puntuación 2, `no saludable`, 🔴 en las tres. | Sin ejecutar / — | Pendiente |
| TC-007 | Tres altos: `(23, 2, 18)`. | Ejecutar el cálculo de semáforo. | Puntuación 3, `no saludable`, 🔴. | Sin ejecutar / — | Pendiente |
| TC-008 | Fronteras: `(22.5, 1.5, 17.5)` y tres variantes aumentando solo uno de los valores en `0.01`. | Ejecutar las cuatro variantes. | Puntuación 0 en la igualdad exacta; puntuación 1 cuando solo uno supera su umbral. | Sin ejecutar / — | Pendiente |
| TC-009 | Ceros: `(0, 0, 0)`. | Ejecutar el cálculo de semáforo. | Valores aceptados, puntuación 0 y `saludable`. | Sin ejecutar / — | Pendiente |
| TC-010 | NutriScore `a` y `b`. | Modificar el grado en la respuesta simulada y ejecutar desde la extracción hasta `Obtener - Dato NutriScore`. | `clasificacionNutriScore` y `valorNutriScore` iguales a `saludable`. | Sin ejecutar / — | Pendiente |
| TC-011 | NutriScore `c`. | Ejecutar el mismo tramo. | Ambos campos iguales a `moderado`. | Sin ejecutar / — | Pendiente |
| TC-012 | NutriScore `d` y `e`. | Ejecutar el mismo tramo para ambos grados. | Ambos campos iguales a `no saludable`. | Sin ejecutar / — | Pendiente |
| TC-013 | Normalización: grados `" A "`, `"C"` y `" e "`. | Ejecutar el tramo de NutriScore con cada variante. | Respectivamente `saludable`, `moderado` y `no saludable`. | Sin ejecutar / — | Pendiente |
| TC-014 | Grados textuales no clasificables: `"unknown"`, `"null"` y `""`; repetir con semáforos saludable, moderado y no saludable. | Ejecutar desde la extracción hasta el nodo que establece `respuesta`. | `valorNutriScore` es el string `"null"`; el IF correspondiente sale por true y `respuesta` conserva la etiqueta del semáforo. Verificar las nueve variantes. | Sin ejecutar / — | Pendiente |
| TC-015 | Nueve combinaciones válidas de semáforo y NutriScore. | Usar los tres conjuntos de nutrientes indicados debajo y los grados `a`, `c`, `e`; ejecutar hasta establecer `respuesta`. | Resultados de la matriz de combinación. Basta con un `no saludable` para obtener esa respuesta. | Sin ejecutar / — | Pendiente |
| TC-016 | Barcode `0`, `-1` y `12.5`. Caracterización de validación incompleta. | Ejecutar solo el primer IF y el guardado del código con cada número. | La comprobación de existencia numérica no rechaza estos formatos; no hay validación de longitud, positividad ni entero. No implica que Open Food Facts encuentre un producto. | Sin ejecutar / — | Pendiente |

### Matriz esperada de TC-015

Usa `(5, 0.5, 5)` para semáforo saludable, `(23, 0.5, 5)` para moderado y `(23, 2, 5)` para no saludable.

| Semáforo por nutrientes | NutriScore `a` | NutriScore `c` | NutriScore `e` |
| --- | --- | --- | --- |
| Saludable | saludable | moderado | no saludable |
| Moderado | moderado | moderado | no saludable |
| No saludable | no saludable | no saludable | no saludable |

## Pruebas de integración

Los casos que dependen de servicios reales requieren credenciales y conectividad. No se debe elegir un resultado esperado a partir de una respuesta que aún no se ha obtenido.

| ID | Caso y preparación | Procedimiento | Resultado esperado / criterio | Resultado real / evidencia | Estado |
| --- | --- | --- | --- | --- | --- |
| TC-017 | Importación y configuración. | Importar el JSON en la instancia destino, revisar los 26 nodos, las conexiones y la asignación de Groq. Registrar `typeVersion` y versión de n8n. | Sin nodos desconocidos; expresiones y conexiones conservadas; credencial asignada. No acredita aún una ejecución completa. | Sin ejecutar / — | Pendiente |
| TC-018 | Consulta real a Open Food Facts. Preparar un barcode numérico de producto con datos, verificado en el momento de la prueba. | Invocar el webhook y conservar URL y salida del nodo HTTP. Comparar la extracción con la respuesta original. | Consulta al endpoint v2 con los cuatro campos solicitados; con `status` numérico 1 y datos completos, extracción correcta de azúcares, sal y grasa por 100 g. | Sin ejecutar / — | Pendiente |
| TC-019 | Convergencia de las cuatro ramas en `Obtener - Respuesta`. | En una copia controlada, ejecutar los recorridos: grado `unknown` con nutrientes bajos; grado `a` con nutrientes bajos; grado `e` con nutrientes bajos; grado `c` con nutrientes bajos. | Cada recorrido llega al Merge por la entrada prevista (0, 1, 2, 3), continúa a la cadena LLM y no mezcla clasificaciones ni genera respuestas duplicadas. | Sin ejecutar / — | Pendiente |
| TC-020 | Groq con las tres clasificaciones. | Ejecutar la cadena con `respuesta` saludable, moderado y no saludable. Conservar texto y clasificación enviados. | Texto no vacío; emoji, color y clasificación coherentes; recomendación pertinente, máximo dos frases, sin nutrientes o ingredientes inventados. No se exige una frase literal idéntica entre ejecuciones. | Sin ejecutar / — | Pendiente |
| TC-021 | Recorrido completo satisfactorio. Fuente real y producto completo verificado. | Enviar POST; conservar ejecución, clasificaciones intermedias, salida LLM, código HTTP, cabeceras y cuerpo. | HTTP 200 al alcanzar `Responder - Respuesta final`; clasificación conforme a los datos observados. Verificar cómo se entrega `$json.text` y cualquier tratamiento HTML del entorno, sin asumir `text/plain` ni JSON. | Sin ejecutar / — | Pendiente |
| TC-022 | Webhook de pruebas y de producción. | En la instancia de pruebas, escuchar en Test URL y enviar una petición. Después publicar/activar esa copia y enviar otra a Production URL, sin datos fijados. | Ambas URLs funcionan en su modo correspondiente; el path es `nutrition-check` y el método POST. Guardar una ejecución de cada modo. | Sin ejecutar / — | Pendiente |

## Pruebas de error

Estas pruebas distinguen las respuestas 400 previstas de las excepciones que el flujo no captura. Para una excepción sin respuesta propia, se espera el comportamiento de error del motor de n8n; hay que registrar el código y cuerpo reales del entorno, no inventar un JSON de error de SnackCheck.

| ID | Entrada o fallo provocado | Procedimiento | Resultado esperado / caracterización | Resultado real / evidencia | Estado |
| --- | --- | --- | --- | --- | --- |
| TC-023 | Body `{}` y `{"barcode":null}`. | Enviar ambos POST al webhook. | Rama false inicial; HTTP 400 JSON, `status: "error"`, `code: "BARCODE_NO_EXIST"`, mensaje obligatorio y timestamp ISO. No se consulta Open Food Facts. | Sin ejecutar / — | Pendiente |
| TC-024 | Barcode `"8480000505460"`, `"abc"`, `true` y `{}`. | Enviar cada variante y revisar el primer IF. | El tipo estricto puede detener el nodo antes de su salida false. Confirmar la incompatibilidad del ejemplo string incluido en el error y registrar excepción y HTTP reales; no presuponer el 400 personalizado. | Sin ejecutar / — | Pendiente |
| TC-025 | Respuesta de Open Food Facts `{"status":0}` sin `error`. | Inyectar la respuesta en una copia controlada. Para medir HTTP, recorrer el webhook de esa copia. | Pasa la ausencia de `error`, falla `status == 1` y alcanza el mensaje de producto inexistente; HTTP 400 si se ejecuta por webhook. | Sin ejecutar / — | Pendiente |
| TC-026 | Salida del nodo HTTP con `error` objeto y, en otra variante, con `error` string. | Simular `{"error":{"message":"Fallo simulado"}}` y `{"error":"Fallo simulado"}`. | Objeto: rama de producto inexistente. String: posible excepción del IF estricto de tipo objeto. Registrar la diferencia, sin considerar ambos casos equivalentes. | Sin ejecutar / — | Pendiente |
| TC-027 | Falta de cada nutriente y objetos `nutriments` o `product` ausentes. | Con `status: 1`, quitar cada campo por separado y luego cada objeto en la respuesta simulada. Ejecutar desde la extracción. | Si la extracción produce valores ausentes, el control posterior rechaza los datos. Si falla antes, registrar el nodo y la excepción. No se garantiza que todas las variantes lleguen al 400 de datos insuficientes. | Sin ejecutar / — | Pendiente |
| TC-028 | Valores inválidos en el Code: negativo, `null`, `""`, campo omitido y `"abc"`, aplicados a cada nutriente; los demás en valores bajos. | Inyectar los campos directamente en `Calcular - Semáforo saludable`. | Excepción `Faltan nutrientes o sus valores no son válidos`; no se calcula clasificación. Prueba aislada: no atribuir una respuesta HTTP. | Sin ejecutar / — | Pendiente |
| TC-029 | NutriScore ausente y NutriScore JSON `null`, con nutrientes completos. | Ejecutar desde la extracción para ambas respuestas simuladas. | El control conjunto rechaza el producto por datos insuficientes; no se alcanza la rama posterior del string `"null"`. Si se recorre por webhook, HTTP 400 de datos insuficientes. | Sin ejecutar / — | Pendiente |
| TC-030 | Credencial Groq inválida o fallo simulado del proveedor. | En una copia, usar una credencial de prueba inválida o un fallo controlado; ejecutar el recorrido hasta Groq. | La cadena falla y no alcanza la respuesta final prevista. No hay plantilla de respaldo ni respuesta personalizada para este fallo. Registrar código del proveedor y HTTP del webhook sin exponer la clave. | Sin ejecutar / — | Pendiente |
| TC-031 | `status` de API como string `"1"`, con producto completo. | Simular la respuesta y ejecutar la comprobación de status. | La validación numérica estricta puede lanzar error de tipo; no se debe asumir la misma ruta que con `status: 1`. Registrar el comportamiento en la versión instalada. | Sin ejecutar / — | Pendiente |
| TC-032 | Fallo real de transporte hacia Open Food Facts en entorno controlado. | En una copia, simular conexión rechazada o timeout mediante una dependencia de prueba. Ejecutar el nodo HTTP con su política de error original. | Observar la forma real de la salida de `continueRegularOutput` y su paso por el IF de error. Confirmar si termina en 400 de producto inexistente o en excepción. No dar por implementada una respuesta específica de indisponibilidad. | Sin ejecutar / — | Pendiente |

En los casos TC-024, TC-026, TC-027, TC-031 y TC-032, el objetivo es caracterizar los puntos que dependen del tipo de dato o del motor. Una vez observado el comportamiento en una versión concreta, fija el resultado esperado para futuras regresiones y conserva la evidencia original.

## Pruebas de rendimiento

**No existen métricas medidas ni un objetivo de latencia acordado.** Los volúmenes siguientes son propuestas para una primera medición, no capacidad garantizada. Utiliza una instancia de pruebas y ajusta el volumen a los límites de los servicios y de la cuenta.

| ID | Escenario y carga propuesta | Procedimiento | Métricas y criterio de evaluación | Resultado real / evidencia | Estado |
| --- | --- | --- | --- | --- | --- |
| TC-033 | Latencia secuencial del recorrido completo: 3 peticiones de calentamiento y 20 peticiones medidas, concurrencia 1. | Usar un producto completo verificado y servicios reales. Separar calentamiento; conservar una fila por petición y la duración de los nodos HTTP y LLM. | Mínimo, mediana, p95 y máximo; proporción de respuestas satisfactorias, errores y timeouts. Objetivo de latencia pendiente de acordar: no declarar aprobado solo por obtener mediciones. | Sin medir / — | Pendiente |
| TC-034 | Concurrencia del recorrido: 20 peticiones con un máximo de 2 simultáneas. | Ejecutar en pruebas con dependencias controladas. Registrar cola, inicio, fin, respuesta y correlación de cada solicitud. Si se repite con servicios reales, tratarlo como una ejecución distinta. | Latencia, solicitudes completadas por segundo, errores y consumo de recursos si es accesible. Comprobar ausencia de respuestas cruzadas. No extrapolar la capacidad de los mocks a Open Food Facts o Groq; objetivo de rendimiento pendiente. | Sin medir / — | Pendiente |
| TC-035 | Coste de rechazo temprano: 20 peticiones secuenciales con body `{}`. | Medir duración completa y revisar las ejecuciones. | Las 20 respuestas deben ser 400 con `BARCODE_NO_EXIST` y no ejecutar Open Food Facts ni Groq. Registrar mediana y p95; este criterio funcional no fija por sí mismo un umbral de velocidad aceptable. | Sin medir / — | Pendiente |

### Cómo registrar las mediciones

- Duración de cada petición: tiempo desde el envío hasta recibir el cuerpo completo, en milisegundos. Usar el mismo cliente y método de medición.
- p95: ordenar las duraciones y tomar la posición `ceil(0.95 × N)`, contando desde 1. Con 20 muestras corresponde a la posición 19; es una estimación inicial con muestra pequeña.
- No ocultar errores ni timeouts: contar todos los intentos. Calcular latencia de respuestas satisfactorias por separado e indicar cuántas muestras se excluyen de ese cálculo.
- Rendimiento efectivo: peticiones completadas divididas por los segundos de la ventana total medida.
- Registrar carga, concurrencia, entorno, tamaño de muestra, servicios reales o simulados, límites aplicados y hora de la prueba.
- Antes de usar estos casos como criterio de aceptación de rendimiento, definir valores objetivo y repetir la medición bajo esas condiciones. No se inventan límites de segundos ni porcentajes de éxito.

## Ficha de ejecución

Copia este bloque por ejecución y variante. Sustituye `TC-XXX` por uno de los identificadores existentes; no reutilices un ID para un caso diferente. Para repetir un caso, conserva el ID y registra un número de ejecución nuevo.

```text
Caso: TC-XXX
Variante:
Número de ejecución:
Fecha y hora, con zona horaria:
Responsable:
Versión del proyecto / commit o hash del JSON:
Versión de n8n:
Entorno y URL:
Alcance: nodo / tramo / webhook completo
Dependencias: reales / simuladas (indicar cuáles)
Preparación y cambios temporales:
Entrada exacta o referencia al archivo:
Resultado esperado:
Pasos realizados:
Resultado observado:
HTTP, cabeceras y cuerpo (si aplica):
Salida o error del nodo (si aplica):
Duración y métricas (si aplica):
ID de ejecución de n8n:
Evidencia: ruta relativa, captura o enlace accesible
Estado: Pendiente / Aprobado / Fallido / Bloqueado
Motivo del estado o desviación:
Incidencia asociada y siguiente acción:
```

Ubicación sugerida para futuras evidencias: `docs/evidencias/TC-001/ejecucion-01/`, adaptando el ID y el número. Esta ruta es una propuesta; no contiene evidencias aportadas con este documento.

## Incidencias conocidas pendientes de validar

Son hallazgos de revisión estática, no fallos observados en ejecuciones:

| Hallazgo | Casos relacionados |
| --- | --- |
| El ejemplo de error usa barcode string y el primer IF espera número. | TC-001, TC-024 |
| No hay validación completa de formato de barcode. | TC-016 |
| Se exige NutriScore antes de permitir el fallback del string `"null"`. | TC-014, TC-029 |
| La condición titulada «Ambos datos son NO saludable» usa OR. | TC-015 |
| El tipo del campo `error` puede impedir alcanzar el mensaje 400. | TC-026, TC-032 |
| Los fallos de extracción y del Code no tienen recuperación específica. | TC-027, TC-028, TC-031 |
| No existe respuesta alternativa si falla Groq. | TC-030 |
