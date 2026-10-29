# analisis-control-plane.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 1.

## TODO 1: Análisis del Plano de Control

### 1. ¿Qué tipo de objeto de Kubernetes es el `dapr-sidecar-injector`? ¿Por qué necesita ser un **MutatingWebhookConfiguration**?

> _Es un deployment de kubernetes, el cual administra un conjunto de pods que ejecutan el servicio del inyector. Para que este Deployment pueda realizar su función, se registra en el clúster a través de un objeto de tipo MutatingWebhookConfiguration, el cual desvía el tráfico de la API de Kubernetes hacia los Pods de este Deployment cuando se crea un nuevo recurso._

### 2. ¿Cuántas réplicas del `dapr-operator` están corriendo? ¿Por qué el plano de control de Dapr es más liviano que el de Istio?

> _Está corriendo 1 réplica del dapr-operator (dapr-operator-7c8bb589b7-7l2np), porque Dapr 1.19.0 se instaló con Helm sin el modo de alta disponibilidad, que desplegaría 3 réplicas. El único componente con 3 réplicas es dapr-scheduler-server, porque usa un etcd embebido que necesita un número impar de nodos para mantener el quórum. El plano de control de Dapr es más liviano que el de Istio porque no intercepta el tráfico de red ni distribuye configuración de enrutamiento a toda la malla._

### 3. Describe con tus palabras el flujo de inyección del sidecar: ¿qué ocurre desde que `kubectl apply` un Deployment con `dapr.io/enabled: "true"` hasta que el sidecar está corriendo?

> _Al hacer kubectl apply, el API Server guarda el Deployment, que crea un ReplicaSet, y este a su vez pide crear los Pods. Antes de guardar cada Pod, el API Server llama al webhook dapr-sidecar-injector. Si el Pod tiene la anotación dapr.io/enabled: “true”, el injector responde con un parche que agrega el contenedor daprd con sus puertos y configuración. Luego el kubelet arranca la aplicación y el sidecar. El sidecar obtiene su certificado de dapr-sentry y carga sus componentes desde el dapr-operator, y el Pod queda en 2/2 Running._

---

## TODO 2: Exploración de los CRDs de Dapr

### 4. ¿Qué CRDs instala Dapr? Lista al menos 4 y describe brevemente para qué sirve cada uno.

> _Dapr instaló 7 CRDs: Components.dapr.io: define qué tecnología concreta implementa cada building block, por ejemplo Redis como broker de Pub/Sub o como state store. Subscriptions.dapr.io: declara de forma declarativa a qué tópico se suscribe una aplicación y a qué ruta deben llegar los mensajes. Configurations.dapr.io: configura el comportamiento de los sidecars, como el tracing, las métricas, mTLS y el control de acceso. Resiliencies.dapr.io: define políticas de reintentos, timeouts y circuit breakers para las llamadas entre servicios y hacia los componentes._

### 5. ¿Cómo se relaciona el CRD `components.dapr.io` con el concepto de "Building Block"? ¿Es el componente la abstracción o la implementación concreta?

> _El building block es la abstracción: una API genérica que la aplicación consume, como Pub/Sub en /v1.0/publish, sin saber qué hay detrás. Component es la implementación concreta: indica qué tecnología respalda esa API y con qué parámetros, por ejemplo type: pubsub.redis con su host y contraseña. Por eso se puede cambiar Redis por Kafka modificando solo el YAML del componente, sin tocar el código de la aplicación._
