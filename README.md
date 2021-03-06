

# Kafka on Kubernetes

Transparent Kafka setup that you can grow with.
Good for both experiments and production.

How to use:
 * Run a Kubernetes cluster, [minikube](https://github.com/kubernetes/minikube) or real.
 * Quickstart: use the `kubectl apply`s below.
 * Kafka for real: fork and have a look at [addon](https://github.com/Yolean/kubernetes-kafka/labels/addon)s.
 * Join the discussion in issues and PRs.

Why?
See for yourself, but we think this project gives you better adaptability than [helm](https://github.com/kubernetes/helm) [chart](https://github.com/kubernetes/charts/tree/master/incubator/kafka)s. No single readable readme or template can properly introduce both Kafka and Kubernets.
Back when we read [Newman](http://samnewman.io/books/building_microservices/) we were beginners with both.
Now we've read [Kleppmann](http://dataintensive.net/), [Confluent](https://www.confluent.io/blog/) and [SRE](https://landing.google.com/sre/book.html) and enjoy this "Streaming Platform" lock-in :smile:.

## What you get

Keep an eye on `kubectl --namespace kafka get pods -w`.

The goal is to provide [Bootstrap servers](http://kafka.apache.org/documentation/#producerconfigs): `kafka-0.broker.kafka.svc.cluster.local:9092,kafka-1.broker.kafka.svc.cluster.local:9092,kafka-2.broker.kafka.svc.cluster.local:9092`
`

Zookeeper at `zookeeper.kafka.svc.cluster.local:2181`.

## Start Zookeeper

The [Kafka book](https://www.confluent.io/resources/kafka-definitive-guide-preview-edition/) recommends that Kafka has its own Zookeeper cluster with at least 5 instances.

```
kubectl apply -f ./zookeeper/
```

To support automatic migration in the face of availability zone unavailability we mix persistent and ephemeral storage.

## Start Kafka

```
kubectl apply -f ./
```

You might want to verify in logs that Kafka found its own DNS name(s) correctly. Look for records like:
```
kubectl -n kafka logs kafka-0 | grep "Registered broker"
# INFO Registered broker 0 at path /brokers/ids/0 with addresses: PLAINTEXT -> EndPoint(kafka-0.broker.kafka.svc.cluster.local,9092,PLAINTEXT)
```

That's it. Just add business value :wink:.
For clients we tend to use [librdkafka](https://github.com/edenhill/librdkafka)-based drivers like [node-rdkafka](https://github.com/Blizzard/node-rdkafka).
To use [Kafka Connect](http://kafka.apache.org/documentation/#connect) and [Kafka Streams](http://kafka.apache.org/documentation/streams/) you may want to take a look at our [sample](https://github.com/solsson/dockerfiles/tree/master/connect-files) [Dockerfile](https://github.com/solsson/dockerfiles/tree/master/streams-logfilter)s.
Don't forget the [addon](https://github.com/Yolean/kubernetes-kafka/labels/addon)s.

# Tests

```
kubectl apply -f test/
# Anything that isn't READY here is a failed test
kubectl get pods -l test-target=kafka,test-type=readiness -w --all-namespaces
```
