### Cluster API quickstart

This is a quickstart collection used as a primer on the [cluster-api](https://cluster-api.sigs.k8s.io) project.

### Installation

to get the necessary tools for the quickstart you can use the accompanying [Brewfile](Brewfile) in the root of this directory with 'brew bundle'.

### Docker

The tools include the Tilt ctlptl cli which can be used to create a local kind cluster with the docker socket mounted to the mgmt control plane node.

From the switch to the [docker](docker) directory start the cluster with:

`ctlptl apply -f capi-mgmt.yaml`

use the included [Tiltfile](Tiltfile) and run `Tilt up` or `Tilt ci` to load the capi components into the control plane.

create the leaf workload cluster with `kubectl apply -f docker-quickstart.yaml` . If you are running a Docker/Podman UI you should be able to observe the cluster get created as containers.

Once you have finished you can delete the workload cluster with `kubectl delete -f docker-quickstart.yaml`.

remove the mgmt cluster with `ctlptl apply -f capi-mgmt.yaml`.

### operator

BLOCKED! - it seems there may be a small issue wit the early release of the operator where it isnt deploying the Bootstrapand controlplane.

For production environments it is more likely that you woll require ongoing management and maintenance of the various Cluster API components and providers. We can take a similar path to the helm based approach above but replace individual components managed via helm with the [Cluster-API operator](https://github.com/kubernetes-sigs/cluster-api-operator).

The operator directory holds a similar example to the docker example but retrieves the CAPI components through the adjoining kustomization directory.

Please see the Cluster-API operator [docs](https://github.com/kubernetes-sigs/cluster-api-operator/blob/main/docs/getting-started.md) to understand the specifics.


