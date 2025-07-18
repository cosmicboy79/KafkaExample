# KafkaExample

kubectl create namespace mykafka

## Installing Kafka Broker

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```bash
helm install --namespace mykafka -f config/values-kafka-broker.yaml local-kafka-broker bitnami/kafka
```

```yaml
externalKafka:
    brokers:
      - PLAINTEXT://<kafka broker name>-controller-0.<kafka broker name>-controller-headless.<namespace>.svc.cluster.local:9092
```

```bash
helm install --namespace mykafka -f config//values-schema-registry.yaml local-schema-registry bitnami/schema-registry
```

local-schema-registry.default.svc.cluster.local:8081

```bash
helm repo add kafka-ui https://provectus.github.io/kafka-ui-charts
helm repo update
```

```bash
helm install --namespace mykafka -f config/values-kafka-ui.yaml local-kafka-ui kafka-ui/kafka-ui
```

```bash
kubectl port-forward deployment/local-kafka-ui 8080:8080
```