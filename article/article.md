# Scheduling control in Kubernetes

This article is intended to overview the capabilities of Scheduling Kubernetes process management and describe, how
their
usage can improve the quality of system.

## Scheduling overview

Scheduling is the process of distributing [pods](https://kubernetes.io/docs/concepts/workloads/pods/) to
the [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/). In other words,the process of determining, on
which
physical\virtual machines the cluster containers will be located.

Basic Kubernetes scheduler is
the [kube-scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/).
It doesn't require some specific settings for the default behavior. Scheduler will try to place pods uniformly between
nodes,
according to the pods resource requirements. Basic scheduler can be replaced with the custom one. It could
be written manually, the main requirement is compliance to the specific API. You can see the
example [here](https://developer.ibm.com/articles/creating-a-custom-kube-scheduler/).
Otherwise, in this article will be described only capabilities of the kube-scheduler.

## Why it is important?

As it was said before, Kubernetes can perform the scheduling without extra instructions. However, appropriate
configuration
of the scheduling may significantly increase the security, reliance and fault tolerance of the cluster.

## Taints and toleration

A taint is a node setting that allows to block it from adding pods. This mechanism can be used, for example, to reserve
a node for a future cluster extending, if it is planned.

The taint to the node `exNode` with `key=exKey`, `value=exValue` and `exEffect` effect can be added using the following
command:

```
kubectl taint nodes exNode exKey=exValue:exEffect
```

Key and value could be arbitrary. For taint effect, three values are available:

1. `NoSchedule` - new pods that do not match the taint are not scheduled onto the node, existing pods remain
2. `PreferNoSchedule` - new pods that do not match the taint are not scheduled onto the node, if there is another
   free space for it, existing pods remain
3. `NoExecute ` - new pods that do not match the taint are not scheduled onto the node, existing pods will be removed

Following command will remove the taint from node:

```
kubectl taint nodes exNode exKey=exValue:exEffect-
```

Tolerations are applied to pods. Toleration allows a pod to overcome the node taint. It is added to `spec.tolerations`
Pod
manifest part.

Here you can see the example of Tolerations. Both of them "match" to the Taint example:

```yaml
tolerations:
  - key: "exKey"
    operator: "Exists"
    effect: "NoSchedule"
```

```yaml
tolerations:
  - key: "exKey"
    operator: "Equal"
    value: "exValue"
    effect: "NoSchedule"
```

A toleration "matches" a taint if the keys are the same and the effects are the same, and:

+ the operator is Exists (in which case no value should be specified), or
+ the operator is Equal and the values are equal.

## NodeSelector and nodeAffinity

`NodeSelector` is the simplest approach for the allocation management. It is a part of Pod manifest and describes
the labels of nodes, where the Pod can be placed, in key-value mapping format. Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sensitive-storage
  labels:
    env: prod
spec:
  containers:
    - name: sensitive-storage-container
      image: sensitive-storage-image
      imagePullPolicy: IfNotPresent
  nodeSelector:
    security: high
```

The Pod, containing storage for sensitive users' information, should be deployed to a specifically labeled node (with
label `security=high`), which is
physically high secured machine using the given manifest.

`nodeSelector` only selects nodes with all the specified labels For some cases, it is not flexible enough. There is
a more expressive, but complicated feature called `nodeAffinity`. It gives more control on selection logic.

`nodeAffinty` is conceptually similar to `nodeSelector`, it also based on the node labels, but gives opportunity to 
describe an operator between labels keys and labels available values. Also, the `AND`\ `OR` expressions combination is supported.

There are two types of `nodeAffinity`:

+ `requiredDuringSchedulingIgnoredDuringExecution` - The scheduler can't schedule the Pod unless the rule is met.
  This functions like nodeSelector, but with a more expressive syntax. Structure:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: { affinityKey }
                operator: { affinityOperator }
                values:
                  - { affinityValues }
```

+ `preferredDuringSchedulingIgnoredDuringExecution` - The scheduler tries to find a node that meets the rule. If a
  matching node is not found, scheduler will place node in some available. Structure:

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: { affinityWeight }
          preference:
            - matchExpressions:
                - key: { affinityKey }
                  operator: { affinityOperator }
                  values:
                    - { affinityValues }
```

`affinityKey` is the key of label to find matching node, `affinityValues` is the set of label values. `affinityOperator`
is the operator to find the compliance between defined in `affinityKey`\ `affinityValues` and key\value of node labels.

The following values are supported for `affinityOperator`:
 + `In` -  any of the listed values are relevant
 + `NotIn` - opposite to `In`
 + `Exists` - the label exists, value is ignored
 + `DoesNotExists` - opposite to `Exists`
 + `Gt` - the label value is greater than the value in policy. Applicable for numeric label values only.
 + `Lt` - opposite to `Gt`

The multiple terms under `nodeSelectorTerms` are combined within `OR` logical operator (if only one of the terms is satisfied, 
the Pod can be placed on the Node). The multiple expressions under `matchExpressions` are combined within logical `AND`.

An `affinityWeight` can be specified for `preferredDuringSchedulingIgnoredDuringExecution` nodeAffinity and applicable for
cases with several. It's value can be from 1 to 100 and applicable for node. When scheduler find suitable for pod node candidate,
it iterates over the rules and summarizes the weights of rules, which the node candidate is matching. Then, scheduler chose
the node candidate with the biggest weight and places the pod there.

Example of `nodeAffinity` usage can be grading the nodes by their performance, stability and reliability. Modern and powerful
devices may be used for the important high-loaded component, outdated - for rarely-used legacy services.

##  PodAffinitty and PodAntiAffinitty

Working principle of `podAffinity` and `podAntiAffinity` are the similar to `nodeAffinity`, but describe the rules
based on the labels of Pods, already existing in the cluster