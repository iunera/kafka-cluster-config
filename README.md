# kafka-cluster-config

This repo contains the configuration files for the Apache Kafka cluster setup based on kubernetes, [strimzi kafka operator](https://github.com/strimzi/strimzi-kafka-operator) and [fluxcd](https://fluxcd.io/flux/components/kustomize/kustomizations/) as gitops tool. 

The cluster has 3 brokers with fixed binding to the kubernetes hosts and uses local pvc (`storage.type: jbod`) to leverage comodity baremetal hard rather running in a cloud environment. 

Further more the brokers listen on `PLAIN/9092` and `TLS/9093` 

In addition the [CMAK (Cluster Manager for Apache Kafka)](https://github.com/yahoo/CMAK) and [AKHQ](https://github.com/tchiotludo/akhq) is deployed a via Helmcharts for overview and insights.



# Installation
The Repo is included in fluxcd with following setup. This installs

* Strimzi operator via helm
* Zookeeper via Strimzi operator
* Kafka via Strimzi operator
* AKHQ
* CMAK

```
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: kafka-cluster-config
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  timeout: 60s
  url: ssh://git@github.com/iunera/kafka-cluster-config
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: kafka-cluster-config
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./kubernetes/
  prune: true
  sourceRef:
    kind: GitRepository
    name: kafka-cluster-config
```

Or use the fluxcd-cli 

```
flux create source git kafka-cluster-config \
  --url=ssh://git@github.com/iunera/kafka-cluster-config \
  --branch=main \
```

# Configure

After that your are able to setup topics and users as your needs. Small examples here:

Topic:
```
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: kvv.testdaten.delays.v1
  namespace: kafka
  labels:
    strimzi.io/cluster: iunerakafkacluster
spec:
  partitions: 1
  replicas: 3
  config:
    retention.ms: 157680000000 # 5 years
    retention.bytes: -1
```

Or a user incl. kafka rbacs
```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: druid
  labels:
    strimzi.io/cluster: iunerakafkacluster
spec:
  authentication:
    type: tls
  authorization:
    type: simple
    acls:
      - resource:
          type: topic
          name: kvv.testdaten.delays.v1
          patternType: literal
        operations:
          - Describe
          - Read
        host: "*"
      - resource:
          type: group
          name: druidconsumer
          patternType: literal
        operations:
          - Read
        host: "*"
```

# License

We choose fair [code, fair work, fair payment, open  collaboration](https://www.license-token.com)

## [Open Compensation Token License](https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md)

```
Licensed under the OPEN COMPENSATION TOKEN LICENSE (the "License").

You may not use this file except in compliance with the License.

You may obtain a copy of the License at
<https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either expressed or implied.
See the License for the specific language governing permissions and
limitations under the License.

@octl.sid: 1b6f7a5d-8dcf-44f1-b03a-77af04433496
```
* Why did we [choose the OCTL as alternative to the BSD 3-Clause License](https://www.license-token.com/wiki/unveiling-bsd-3-clause-license-summary)?
* Why we [do NOT apply Apache 2.0 License](https://www.license-token.com/wiki/the-downside-of-apache-license-and-why-i-never-would-use-it)?
