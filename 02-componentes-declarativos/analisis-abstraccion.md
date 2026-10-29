# analisis-abstraccion.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 2.

## TODO 2: Análisis de la Abstracción

### 1. **Migración de broker:** Si la empresa decide migrar de Redis a Apache Kafka mañana, ¿qué archivos del repositorio habría que modificar? ¿Cambiaría el código Python del subscriber o del publisher?

> _Solo habría que modificar el archivo 02-componentes-declarativos/pubsub-redis.yaml. En él se cambia type: pubsub.redis por type: pubsub.kafka y se reemplazan los datos de conexión de Redis por los de Kafka, manteniendo el nombre order-pubsub. El código Python del subscriber y del publisher no cambiaría, porque ambos hablan con el sidecar de Dapr en localhost usando solo el nombre del componente y el tópico, sin saber qué broker hay detrás._

### 2. **Inventario de brokers soportados:** Consulta la [documentación de componentes Pub/Sub de Dapr](https://docs.dapr.io/reference/components-reference/supported-pubsub/). Lista **5 brokers distintos** que Dapr soporta como implementación. ¿Qué campo en el YAML cambia para apuntar a cada uno?

| Broker | `spec.type` en el YAML |
|---|---|
| Redis | `pubsub.redis` |
| Apache Kafka | `pubsub.kafka` |
| RabbitMQ | `pubsub.rabbitmq` |
| Apache Pulsar | `pubsub.pulsar` |
| AWS SNS/SQS | `pubsub.aws.snssqs` |

### 3. **Secretos en producción:** El valor `redisPassword: ""` funciona para el lab, pero en producción nunca se deben poner contraseñas en texto plano en un YAML de Git. ¿Qué mecanismo ofrece Dapr para referenciar secretos de forma segura? (Pista: busca "Dapr Secret Store component reference")

> _Dapr permite referenciar secretos con secretKeyRef, en lugar de escribir la contraseña directamente en el YAML. El componente apunta al nombre y la clave de un secreto, y Dapr lo resuelve al arrancar el sidecar. En Kubernetes usa por defecto los Secrets nativos del clúster. Así, en Git solo queda la referencia y nunca el valor real_

name: redisPassword
      secretKeyRef:
        name: redis
        key: redis-password
auth:
  secretStore: kubernetes
