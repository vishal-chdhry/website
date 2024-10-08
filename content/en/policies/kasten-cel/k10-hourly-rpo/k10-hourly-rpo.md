---
title: "Check Hourly RPO in CEL expressions"
category: Kasten K10 by Veeam in CEL
version: 1.11.0
subject: Policy
policyType: "validate"
description: >
    K10 Policy resources can be educated to adhere to common Recovery Point Objective (RPO) best practices.  This policy is advising to use an RPO frequency that with hourly granularity if it has the appPriority: Mission Critical
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//kasten-cel/k10-hourly-rpo/k10-hourly-rpo.yaml" target="-blank">/kasten-cel/k10-hourly-rpo/k10-hourly-rpo.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: k10-policy-hourly-rpo
  annotations:
    policies.kyverno.io/title: Check Hourly RPO in CEL expressions
    policies.kyverno.io/category: Kasten K10 by Veeam in CEL 
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Policy
    policies.kyverno.io/description: >-
      K10 Policy resources can be educated to adhere to common Recovery Point Objective (RPO) best practices. 
      This policy is advising to use an RPO frequency that with hourly granularity if it has the appPriority: Mission Critical
spec:
  validationFailureAction: Audit  
  rules:
  - name: k10-policy-hourly-rpo
    match:
      any:
      - resources:
          kinds:
          - config.kio.kasten.io/v1alpha1/Policy
          operations:
          - CREATE
          - UPDATE
          selector:
            matchLabels:
              appPriority: Mission-Critical
    validate:
      cel:
        expressions:
          - expression: "has(object.spec.frequency) && object.spec.frequency == '@hourly'"
            message: "Mission Critical RPO frequency should use no shorter than @hourly frequency"


```
