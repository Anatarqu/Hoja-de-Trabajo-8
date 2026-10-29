# analisis-cloudevents.md
# Completa este archivo con tus respuestas al TODO 4 de la Fase 3.

## TODO 4: Análisis de Trazabilidad (CloudEvents)

### 1. **`id`**: ¿Quién genera este UUID? ¿Por qué es importante para la idempotencia?

> _El UUID lo genera el sidecar de Dapr del publisher, no el código Python. Cuando la aplicación llama a la API de publicación, daprd envuelve el mensaje en un CloudEvent 1.0 y le asigna un id único. Es importante para la idempotencia porque Dapr garantiza entrega “al menos una vez”: si el subscriber no responde 200 o hay un fallo de red, el mensaje se reintenta y puede llegar duplicado, pero con el mismo id. Así, el subscriber puede guardar los ids ya procesados e ignorar los repetidos, evitando, por ejemplo, despachar dos veces la misma orden._

### 2. **`source`**: ¿Qué valor tiene? ¿De dónde lo toma Dapr?

> _El campo source tiene el valor order-publisher. Dapr lo toma del app-id de la aplicación que publicó el mensaje, es decir, de la anotación dapr.io/app-id: "order-publisher" del Deployment del publisher. El sidecar lo agrega automáticamente al construir el CloudEvent, sin que el código Python tenga que indicarlo._

### 3. **`traceparent`**: ¿Qué estándar implementa este atributo? ¿Cómo se relaciona con OpenTelemetry?

> _El atributo traceparent implementa el estándar W3C Trace Context, que define cómo propagar el contexto de una traza distribuida entre servicios. Su formato es version-traceid-spanid-flags. Dapr agrega este atributo al CloudEvent para que una herramienta compatible con OpenTelemetry, como Jaeger o Zipkin, pueda unir la publicación y el consumo del mensaje en una sola traza de extremo a extremo. En los logs aparece vacío porque en el laboratorio no se configuró el tracing de Dapr, es decir, no hay un recurso Configuration con muestreo habilitado ni un colector de trazas._

### 4. **Valor de negocio**: Un equipo de auditoría necesita rastrear todos los eventos que pasaron por el sistema en las últimas 24h. ¿Qué atributos del sobre CloudEvents les serían más útiles y por qué?

> _Los atributos más útiles para la auditoría serían id, time, source, type y traceparent. El id identifica cada evento de forma única, lo que permite contarlos sin duplicados y detectar reintentos. El time indica cuándo se generó el evento, así que sirve para filtrar exactamente las últimas 24 horas. El source dice qué aplicación lo emitió (por ejemplo order-publisher), lo que permite atribuir responsabilidades. El type clasifica la clase de evento. El traceparent permite reconstruir todo el recorrido del evento entre servicios cuando el tracing está habilitado._
