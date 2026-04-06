# kyverno

## install kyverno

`kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.16.2/install.yaml`

`kubectl get pods -n kyverno`

```yaml
apiVersion: policies.kyverno.io/v1alpha1
kind: ValidatingPolicy
metadata:
  name: require-labels
spec:
  validationActions:
    - Deny
  matchConstraints:
    resourceRules:
      - apiGroups: ['']
        apiVersions: ['v1']
        operations: ['CREATE', 'UPDATE']
        resources: ['pods']
  validations:
    - message: "label 'team' is required"
      expression: "has(object.metadata.labels) && has(object.metadata.labels.team) && object.metadata.labels.team != ''"
```

`kubectl create deployment nginx --image=nginx`
