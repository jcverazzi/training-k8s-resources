## Création du pod Nginx

`kubectl run mynginx --image=nginx:1.15 --port=80`

## Liste des status possibles pour un pod

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase

```
A Pod’s status field is a PodStatus object, which has a phase field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of Container or Pod state, nor is it intended to be a comprehensive state machine.

The number and meanings of Pod phase values are tightly guarded. Other than what is documented here, nothing should be assumed about Pods that have a given phase value.

Here are the possible values for phase:

- Pending: The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.

- Running: The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.

- Succeeded: All Containers in the Pod have terminated in success, and will not be restarted.

- Failed: All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.

- Unknown: For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.
```



