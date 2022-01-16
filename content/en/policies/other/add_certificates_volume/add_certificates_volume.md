---
title: "Add Certificates as a Volume"
category: Sample
version: 1.5.0
subject: Pod,Volume
policyType: "mutate"
description: >
    In some cases you would need to trust custom CA certificates for all the containers of a Pod. It makes sense to be in a ConfigMap so that you can automount them by only setting an annotation. This policy adds a volume to all containers in a Pod containing the certificate if the annotation called `inject-certs` with value `enabled` is found.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/add_certificates_volume/add_certificates_volume.yaml" target="-blank">/other/add_certificates_volume/add_certificates_volume.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-certificates-volume
  annotations:
    policies.kyverno.io/title: Add Certificates as a Volume
    policies.kyverno.io/category: Sample
    policies.kyverno.io/subject: Pod,Volume
    kyverno.io/kyverno-version: 1.5.2
    kyverno.io/kubernetes-version: "1.21"
    policies.kyverno.io/minversion: 1.5.0
    pod-policies.kyverno.io/autogen-controllers: DaemonSet,Deployment,Job,StatefulSet
    policies.kyverno.io/description: >-
      In some cases you would need to trust custom CA certificates for all the containers of a Pod.
      It makes sense to be in a ConfigMap so that you can automount them by only setting an annotation.
      This policy adds a volume to all containers in a Pod containing the certificate if the annotation
      called `inject-certs` with value `enabled` is found.
spec:
  background: false
  rules:
  - name: add-ssl-certs
    match:
      resources:
        kinds:
        - Pod
    preconditions:
      all:
      - key: '{{request.object.metadata.annotations."inject-certs"}}'
        operator: Equals
        value: enabled
      - key: "{{request.operation}}"
        operator: In
        value:
          - CREATE
          - UPDATE
    mutate:
      foreach:
      - list: "request.object.spec.containers"
        patchStrategicMerge:
          spec:
            containers:
            - name: "{{ element.name }}"
              volumeMounts:
              - name: etc-ssl-certs
                mountPath: /etc/ssl/certs
            volumes:
            - name: etc-ssl-certs
              configMap:
                name: ca-pemstore

```