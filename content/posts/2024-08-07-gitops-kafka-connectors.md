+++
title = 'Gitops Kafka Connectors'
date = 2024-08-07T11:13:19+02:00
draft = false
tags = ['kubernetes', 'kafka', 'gitops']
+++

Hey there, developers!


I want to share a simple proof of concept on GitHub that showcases how to use the Operator SDK to create an operator to configure connectors for Kafka Connect.

Working with a colleague, we tried to figure out if there is any option to configure connectors on a Kafka Connect cluster, outside Kubernetes, using a GitOps approach. 

We know the Strimzi project uses a declarative approach to manage connectors for Kafka Connect clusters, but it's limited to clusters managed by Strimzi itself. To achieve a similar behavior, we explored the possibility of implementing a custom operator and came up with this proof of concept.

The project demonstrates how to use the Ansible SDK to create an operator that configures Kafka Connect connectors via its REST API. It's a straightforward example to show how these technologies can work together to manage a Kafka Connect cluster that's external to your Kubernetes environment.


{{< notice info >}}
The operator is not production-ready and is only intended to inspire you on how to implement your own operator for similar problems.
{{< /notice >}}


## Custom Resource Definition 

Below an example of the `KafkaConnector` custom resource. 

To keep things simple, labels have been used to provide information about the external connect clusters(a better approach would be to externalize such informations. i.e. in a secret.) while the spec part, have been dedicated to the connector config emulating the strimzi behaviour.

```
apiVersion: external.saiello.github.io/v1alpha1
kind: KafkaConnector
metadata:
  name: kafkaconnector-file-source-sample

  labels:
    external.saiello.github.io/api-schema: 'http'
    external.saiello.github.io/api-host: '<REMOTE_ADDRESS_OF_CONNECT_CLUSTER>'
    external.saiello.github.io/api-port: '8083'

spec:

  class: org.apache.kafka.connect.file.FileStreamSourceConnector

  config: 
    topic: connect-test

  ... omit ...
```


## Internals 

The Ansible role within the operator uses builtin modules to interact with the REST api and create or update the connector according to the desired state. 

```
- name: Update connector's config 
  ansible.builtin.uri:
    url: "{{ _api_url }}/connectors/{{ ansible_operator_meta.name }}/config" 
    method: PUT
    body_format: json
    body: "{{ all_configs }}"
  when: 
    - state == 'running'
    - connector_exists
```


## Conclusion

I hope you can find inspiration from the poc and that the resources provided help you dive deeper into the possibilities of using the Ansible SDK and Kubernetes operators to manage Kafka Connect clusters. Your feedback are welcome, and I'd love to hear how you plan to use or extend this approach in your projects.


## Resources

- [External Kafka Connect operator](https://github.com/saiello/external-kafka-connect-operator)
- [Gitops Principles](https://opengitops.dev/)
- [Operator SDK](https://sdk.operatorframework.io/)
- [Strimzi](https://strimzi.io/)