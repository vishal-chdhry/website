---
title: "Replace Image Registry With Harbor"
category: Sample
version: 
subject: Pod
policyType: "mutate"
description: >
    Some registries like Harbor offer pull-through caches for images from certain registries. Images can be re-written to be pulled from the redirected registry instead of the original and the registry will proxy pull the image, adding it to its internal cache. The imageData context variable in this policy provides a normalized view of the container image, allowing the policy to make decisions based on various  "live" image details. As a result, it requires access to the source registry and the existence of the target image to verify those details.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/replace-image-registry-with-harbor/replace-image-registry-with-harbor.yaml" target="-blank">/other/replace-image-registry-with-harbor/replace-image-registry-with-harbor.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: replace-image-registry-with-harbor
  annotations:
    policies.kyverno.io/title: Replace Image Registry With Harbor
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.11.4
    kyverno.io/kubernetes-version: "1.27"
    policies.kyverno.io/description: >-
      Some registries like Harbor offer pull-through caches for images from certain registries.
      Images can be re-written to be pulled from the redirected registry instead of the original and
      the registry will proxy pull the image, adding it to its internal cache.
      The imageData context variable in this policy provides a normalized view
      of the container image, allowing the policy to make decisions based on various 
      "live" image details. As a result, it requires access to the source registry and the existence
      of the target image to verify those details.
spec:
  rules:
    - name: redirect-docker
      match:
        any:
          - resources:
              kinds:
                - Pod
              operations:
                - CREATE
                - UPDATE
      mutate:
        foreach:
          - list: request.object.spec.initContainers[]
            context:
              - name: imageData
                imageRegistry:
                  reference: "{{ element.image }}"
            preconditions:
              any:
                - key: "{{imageData.registry}}"
                  operator: Equals
                  value: index.docker.io
            patchStrategicMerge:
              spec:
                initContainers:
                  - name: "{{ element.name }}"
                    image: harbor.example.com/k8s/{{imageData.repository}}:{{imageData.identifier}}
          - list: request.object.spec.containers[]
            context:
              - name: imageData
                imageRegistry:
                  reference: "{{ element.image }}"
            preconditions:
              any:
                - key: "{{imageData.registry}}"
                  operator: Equals
                  value: index.docker.io
            patchStrategicMerge:
              spec:
                containers:
                  - name: "{{ element.name }}"
                    image: harbor.example.com/k8s/{{imageData.repository}}:{{imageData.identifier}}

```
