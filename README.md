# kubectl-vscale

Did you know it is possible to change resource allocation of a running pod - on the fly - by messing with the cgroup params? Well, you can! Now in a nice packaged form.

```
kubectl vscale -n <namespace> -c 2 <pod>
```
limits <pod> CPU to 2 cores instantaneously - or a few seconds. Subsequent commands can increace/decrease this value in any order - the only assumption being that the amound being requested is less than the initial limit value specified in the pod yaml.

You can use the `-u` flag to go unlimited and `-r` to reset to the pod default limits specified in the resource.

## Why?

Normally scaling pods up/down requires a change of Deployment resource yaml that ends up destroying the running pod and creating a new one in its place. The best you can do is using rollout - even then a pod has to go away.

For many stateful applications, this ends up messing the full system. A restart assumes ordering between multiple components - Deployments usually, and if this is not done, the system goes into an unusable state. You may say that this is not how applications **should** be designed for Kubernetes, but sometimes this is how they **are**.

## Why not?

Doing it directly from the Cgroup layer means that Kubernetes is no longer aware of the actual resources being used. *This will break general Kubernetes resource accounting etc*. **Don't use in production.**

Great for research and experiments though!!

## How?

The gist of the matter is that we use a privileged pod on the same node as the pod, nsenter into the host namespace and edit the host `/sys/fs/cgroup` hierarchy directly at the correct leaf node that represents the primary container in the target pod.

More details can be found by perusing the code.
