<!-- Scrum Call
----------
OIDC - external id provider, authentication protocol
keycloak - Acts as an OIDC provider
RBAC

role
rolebinding
clusterrole
clusterroleBinding

service accounts

node selector
node affinity
pod affinity
pod anti affinity -->
# Taints and Tolerations

Taints allow a node to repel a set of pods.
Tolerations are applied to pods, allow the scheduler to schedule pods with matching taints.
Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes.

- add taint to node `kubectl taint nodes node1 key1=value1:NoSchedule`
- remove taint `kubectl taint nodes node1 key1=value1:NoSchedule-`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key1"
    operator: "Exists"
    effect: "NoSchedule"
```

or

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

if you manually specify the `.spec.nodeName` for a Pod, that action bypasses the scheduler; the Pod is then bound onto the node where you assigned it, even if there are `NoSchedule` taints on that node that you selected. If this happens and the node also has a `NoExecute` taint set, the kubelet will eject the Pod unless there is an appropriate tolerance set.

The default value for `operator` is `Equal`.

A toleration "matches" a taint if the keys are the same and the effects are the same, and:

- the `operator` is `Exists` (in which case no value should be specified).
- the `operator` is `Equal` and the values should be equal.

values for the effect field are:

**NoExecute** - This affects pods that are already running on the node
**NoSchedule** - Pods currently running on the node are **not** evicted.
**PreferNoSchedule** - a "preference" or "soft" version of NoSchedule.
