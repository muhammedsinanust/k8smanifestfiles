# Admission control for resource limit with OPA

## installing opa gatekeeper

`kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.16/deploy/gatekeeper.yaml`

## resource limit template

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredresources
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredResources
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredresources
 
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits
          msg := sprintf("Container %v must have resource limits", [container.name])
        }
```

constrainttemplate.templates.gatekeeper.sh/k8srequiredresources created

## resource limit constraint

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
```

k8srequiredresources.constraints.gatekeeper.sh/require-resource-limits created

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

`Error from server (Forbidden): error when creating "testconstraint.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [require-resource-limits] Container nginx must have resource limits`

## for latest image tag

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sdisallowlatest
spec:
  crd:
    spec:
      names:
        kind: K8sDisallowLatest
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sdisallowlatest
 
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          endswith(container.image, ":latest")
          msg := "Using latest tag is not allowed"
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sDisallowLatest
metadata:
  name: disallow-latest
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
---
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

`Error from server (Forbidden): error when creating "testconstraint.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [require-resource-limits] Container nginx must have resource limits
[disallow-latest] Using latest tag is not allowed`

## for min replicas

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sminreplicas
spec:
  crd:
    spec:
      names:
        kind: K8sMinReplicas
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sminreplicas

        violation[{"msg": msg}] {
          input.review.object.kind == "Deployment"
          replicas := input.review.object.spec.replicas
          replicas < 2
          msg := sprintf("Deployment must have at least 2 replicas, found %v", [replicas])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sMinReplicas
metadata:
  name: enforce-min-replicas
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fail-both-policies
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: fail-app
  template:
    metadata:
      labels:
        app: fail-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

`Error from server (Forbidden): error when creating "minreplicas.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [enforce-min-replicas] Deployment must have at least 2 replicas, found 1`

## for rollingupdates

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srollingupdate
spec:
  crd:
    spec:
      names:
        kind: K8sRollingUpdate
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srollingupdate

        violation[{"msg": msg}] {
          input.review.object.kind == "Deployment"
          not input.review.object.spec.strategy.type
          msg := "Deployment must define a strategy"
        }

        violation[{"msg": msg}] {
          input.review.object.kind == "Deployment"
          input.review.object.spec.strategy.type != "RollingUpdate"
          msg := "Only RollingUpdate strategy is allowed"
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRollingUpdate
metadata:
  name: enforce-rolling-update
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fail-both-policies
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: fail-app
  template:
    metadata:
      labels:
        app: fail-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
