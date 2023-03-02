### Cluster API quickstart

This is a quickstart collection used as a primer on the [cluster-api](https://cluster-api.sigs.k8s.io) project.

### Installation

to get the necessary tools for the quickstart you can use the accompanying [Brewfile](Brewfile) in the root of this directory with 'brew bundle'.

### get a local cluster

I have included ctlptl and local [kind](capi-mgmt.yaml) cluster file which will also add a local registry to your docker instance but you can use whatever you like.

NOTE: If you would like to experiment with the microvm provider you will need to use the included Vagrant file and run:

```
Vagrant up
export KUBECONFIG=kc/k3s.yaml
```

use the included [Tiltfile](docker/Tiltfile) and run `Tilt up` or `Tilt ci` to load the capi components into the control plane.

### Providers

the following providers have been tested:

- docker (CAPD)
- vcluster (CAPVC)
- microvm (CAPMVM)

### Vcluster

Once your cluster is ready and the kube-context is set, make sure Tilt is installed and then run:
```
tilt up -- --provider=${PROVIDER} --ui=False
```
where provider is one of the named options above ie. `tilt up -- --provider=docker --ui=False`

This will:
- install cert-manager
- install cluster-api-operator and cluster-api components
- install the infrastucture provider you have listed.

Once ready you can then install capi predicates such as kubeadm and workload templates for docker/microvm providers with:
```
kubectl apply -f manifests/${PROVIDER}/
```
*NOTE* you can skip this step for capvc.

Now you are read to spin up a workload cluster with `kubectl apply -f manifests/clusters/${PROVIDER}/cluster.yaml`

you can run ` kubectl get clusters -A --watch` to observe the cluster creation process or follow the controller logs with `kubectl logs deployment/capmvm-controller-manager -n capmvm-system` for whichever respective provider is in use.

you can use the name of the cluster to obtain its kubeconfig once it is provisioned, i.e `mkdir -p kc && kubectl get secret capvc-workload-kubeconfig -o json -n capvc-system | jq -r .data.value | base64 -d > kcconfig.yaml`

to delete a cluster use `kubectl delete -f manifests/clusters/${PROVIDER}/cluster.yaml`

sometimes clusters can get stuck (especially if you delete things in the wrong order). you can use `kubectl patch cluster lm-1 -p '{"metadata":{"finalizers":null}}' --type=merge`
to remove the finalizer on a cluster stuck in the deletion phase.

### GitOps

To enable some of the GitOps features you can enable Flux and the Weave GitOps UI by setting the UI to true ie `tilt up -- --provider=docker --ui=True`

now you can take the same paths and reconcile the clusters with `gitops beta run ./manifests/clusters/${PROVIDER}/ --no-session --skip-dashboard-install --no-bootstrap --insecure-skip-tls-verify`

where GitOps is able to apply the cluster components and spin up clusters with the configurations
