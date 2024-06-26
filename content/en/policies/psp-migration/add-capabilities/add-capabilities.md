---
title: "Add Capabilities"
category: PSP Migration
version: 
subject: Pod
policyType: "mutate"
description: >
    In the earlier Pod Security Policy controller, it was possible to configure a policy to add capabilities to containers within a Pod. This made it easier to assign some basic defaults rather than blocking Pods or to simply provide capabilities for certain workloads if not specified. This policy mutates Pods to add the capabilities SETFCAP and SETUID so long as they are not listed as dropped capabilities first.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//psp-migration/add-capabilities/add-capabilities.yaml" target="-blank">/psp-migration/add-capabilities/add-capabilities.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-capabilities
  annotations:
    policies.kyverno.io/title: Add Capabilities
    policies.kyverno.io/category: PSP Migration
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.10.0
    kyverno.io/kubernetes-version: "1.24"
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/description: >-
      In the earlier Pod Security Policy controller, it was possible to configure a policy
      to add capabilities to containers within a Pod. This made it easier to assign some basic defaults
      rather than blocking Pods or to simply provide capabilities for certain workloads if not specified.
      This policy mutates Pods to add the capabilities SETFCAP and SETUID so long as they are not listed
      as dropped capabilities first.
spec:
  rules:
  - name: add-setfcap-setuid
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
          - CREATE
          - UPDATE
    mutate:
      foreach:
      - list: request.object.spec.[ephemeralContainers, initContainers, containers][]
        preconditions:
          all:
          - key: SETFCAP
            operator: AnyNotIn
            value: "{{ element.securityContext.capabilities.drop[] || `[]` }}"
        patchesJson6902: |-
          - path: /spec/containers/{{elementIndex}}/securityContext/capabilities/add/-
            op: add
            value: SETFCAP
      - list: request.object.spec.[ephemeralContainers, initContainers, containers][]
        preconditions:
          all:
          - key: SETUID
            operator: AnyNotIn
            value: "{{ element.securityContext.capabilities.drop[] || `[]` }}"
        patchesJson6902: |-
          - path: /spec/containers/{{elementIndex}}/securityContext/capabilities/add/-
            op: add
            value: SETUID

```
