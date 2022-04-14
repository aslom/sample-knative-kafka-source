# sample-knative-kafka-source

## Hello display to show events

https://knative.dev/docs/eventing/getting-started/#creating-event-consumers

```
kubectl apply -f hello-display.yaml
```


## Sending test events to event displat

```
kubectl apply -f curl.yaml
```


```
kubectl attach curl -it
```


```
curl -v "http://hello-display" \
-X POST \
-H "Ce-Id: say-hello" \
-H "Ce-Specversion: 1.0" \
-H "Ce-Type: greeting" \
-H "Ce-Source: not-sendoff" \
-H "Content-Type: application/json" \
-d '{"msg":"Hello Knative!"}'
```

Check hello-display logs:

```
kubectl logs -l app=hello-display --tail=100
```

Expected output:

```
2022/04/14 19:20:26 Failed to read tracing config, using the no-op default: empty json tracing config
☁️  cloudevents.Event
Context Attributes,
  specversion: 1.0
  type: greeting
  source: not-sendoff
  id: say-hello
  datacontenttype: application/json
Data,
  {
    "msg": "Hello Knative!"
  }
  ```

## Create test topic with strimzi

Install strimzi https://strimzi.io/


```
kubectl apply -f knative-demo-topic.yaml
```

## Kafka source

https://knative.dev/docs/eventing/sources/kafka-source/#kafka-event-source

```
kubectl apply -f kafka-source.yaml
```


```
kubectl get kafkasource kafka-source
```

Expacted output:

```
NAME           TOPICS                   BOOTSTRAPSERVERS                            READY   REASON   AGE
kafka-source   ["knative-demo-topic"]   ["my-cluster-kafka-bootstrap.kafka:9092"]   True
```



## Sending test event to Kafka topic

https://knative.dev/docs/eventing/sources/kafka-source/#verify

```
kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.14.0-kafka-2.3.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic knative-demo-topic
```

```
If you don't see a command prompt, try pressing enter.
>Alek1
>
```

Expected output from hello-display

```
kubectl logs -l app=hello-display --tail=100
```


```
☁️  cloudevents.Event
Context Attributes,
  specversion: 1.0
  type: dev.knative.kafka.event
  source: /apis/v1/namespaces/alek/kafkasources/kafka-source#knative-demo-topic
  subject: partition:0#0
  id: partition:0/offset:0
  time: 2022-04-14T19:31:34.395Z
Data,
  Alek1
  ```