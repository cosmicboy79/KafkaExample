# Kafka Example

This is a small project that exemplifies the use of Kafka Producer and Consumer in Spring Boot.

## Prerequisites

The components of this project requires a Kubernetes (K8s) cluster. Personally, I am using
[Rancher Desktop](https://rancherdesktop.io/) with `containerd` as the container runtime.

Having installed Rancher Desktop correctly, I suggest to create a separate namespace
for the Kafka components and the Spring Boot applications. The configurations suggested
in this project use the namespace `mykafka`, created as follows:

```bash
kubectl create namespace mykafka
```

## Installing Kafka Broker

Kafka Broker is installed via [Bitnami Kafka Helm Chart](https://github.com/bitnami/charts/tree/main/bitnami/kafka), which must
be first of all added to the Helm repositories, as follows:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

For the purposes of this project, it is enough to install Kafka with one broker only. In order to do so, the following
configuration must be used in the Helm chart's `values.yaml` file:

```yaml
controller:
  overrideConfiguration:
    transaction.state.log.replication.factor: 1
    transaction.state.log.min.isr: 1
    offsets.topic.replication.factor: 1
  replicaCount: 1

listeners:
  client:
    protocol: PLAINTEXT
  controller:
    protocol: PLAINTEXT
  interbroker:
    protocol: PLAINTEXT
  external:
    protocol: PLAINTEXT
```

This configuration is already to be found in file [config/values-kafka-broker.yaml](config/values-kafka-broker.yaml), and by using it
the Kafka Broker can be installed with the following command:

```bash
helm install --namespace mykafka -f config/values-kafka-broker.yaml local-kafka-broker bitnami/kafka
```

This will create a Kafka Broker with the name `local-kafka-broker-controller-0` in the namespace `mykafka`.

## Installing Schema Registry

Schema Registry is installed via [Bitnami Schema Registry Helm Chart](https://github.com/bitnami/charts/tree/main/bitnami/schema-registry) and
and must point to a Kafka Broker, as the following configuration for the Helm chart's `values.yaml` file shows:

```yaml
externalKafka:
    brokers:
      - PLAINTEXT://<kafka broker name>-controller-0.<kafka broker name>-controller-headless.<namespace>.svc.cluster.local:9092
```

For the sake of this project, it must point to the Kafka Broker installed in the previous step. This
configuration is already to be found in file [config/values-schema-registry.yaml](config/values-schema-registry.yaml), and by using it
the Schema Registry can be installed with the following command:

```bash
helm install --namespace mykafka -f config/values-schema-registry.yaml local-schema-registry bitnami/schema-registry
```

This will create a Schema Registry instance with the name `local-schema-registry` in the namespace `mykafka`, exposed
inside the K8s cluster at the address `local-schema-registry.default.svc.cluster.local:8081`.

## Installing Kafka UI

Kafka UI is installed via [Provectus Kafka UI Helm Chart](https://github.com/provectus/kafka-ui)
which first of all must be added to the Helm repositories, as follows:

```bash
helm repo add kafka-ui https://provectus.github.io/kafka-ui-charts
helm repo update
```

The Kafka UI installation must point to a Kafka Broker and Schema Registry, as the following
configuration for the Helm chart's `values.yaml` file shows:

```yaml
yamlApplicationConfig:
  auth:
    type: disabled
  management:
    health:
      ldap:
        enabled: false
  kafka:
    clusters:
      - name: <kafka broker name>-controller-0
        bootstrapServers: <kafka broker name>-controller-0.<kafka broker name>-headless.<namespace>.svc.cluster.local:9092
        schemaRegistry: http://<schema registry name>.<namespace>.svc.cluster.local:8081
```

For the sake of this project, it must use the Kafka Broker and Schema Registry
installed in the previous steps. This configuration is already to be found in file
[config/values-kafka-ui.yaml](config/values-kafka-ui.yaml), and by using it the Kafka UI can be installed with the following command:

```bash
helm install --namespace mykafka -f config/values-kafka-ui.yaml local-kafka-ui kafka-ui/kafka-ui
```

This will create a Kafka UI instance with the name `local-kafka-ui` in the namespace `mykafka`, exposed
inside the K8s cluster at the address `local-kafka-ui.default.svc.cluster.local:8080`.

The Kafka UI can be accessed via Browser at the address `localhost:8080` by port-forwarding the deployment, as follows:

```bash
kubectl port-forward deployment/local-kafka-ui 8080:8080
```
