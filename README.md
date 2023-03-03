### Cluster API quickstart

This is a quickstart collection used as a primer on the [cluster-api](https://cluster-api.sigs.k8s.io) project.

### Installation

to get the necessary tools for the quickstart you can use the accompanying [Brewfile](Brewfile) in the root of this directory with 'brew bundle'.

### Docker

The tools include the Tilt ctlptl cli which can be used to create a local kind cluster with the docker socket mounted to the mgmt control plane node.

Switch to the [docker](docker) directory and start a local kind cluster with:

`ctlptl apply -f capi-mgmt.yaml`

use the included [Tiltfile](docker/Tiltfile) and run `Tilt up` or `Tilt ci` to load the capi components into the control plane.

create the leaf workload cluster with `kubectl apply -f docker-quickstart.yaml` . If you are running a Docker/Podman UI you should be able to observe the cluster get created as containers.

Once you have finished you can delete the workload cluster with `kubectl delete -f docker-quickstart.yaml`.

remove the mgmt cluster with `ctlptl apply -f capi-mgmt.yaml`.

### Vcluster

Follow the same steps as above, replacing 'docker' with vcluster.

KUBECONFIG=k3s/k3s.yaml tilt ci
KUBECONFIG=k3s/k3s.yaml kubectl apply -f manifests/mvm-cluster.yaml
KUBECONFIG=k3s/k3s.yaml kubectl get secret lm-1-kubeconfig -o json -n capmvm-system | jq -r .data.value | base64 -d > k3s/config.yaml
