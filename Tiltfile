allow_k8s_contexts('default')
load('ext://cert_manager', 'deploy_cert_manager')

# use a local registry cache
default_registry(
    'localhost:5000',
    host_from_cluster='registry:5000'
)

# install cert manager
deploy_cert_manager(
    version='v1.11.0')

# capi version
cv='1.3.3'

# list capi components
components = {
    'capi': cv,
    'capi-kubeadm-bootstrap': cv,
    'capi-kubeadm-control-plane': cv,
}

# helm install for oci targets
for k in components:
    cmd="helm install {} --version {} --namespace {}-system --set providerArgs.clusterTopology=true --set providerArgs.clusterResourceSet=true --create-namespace oci://ghcr.io/fire-ant/{}".format(k,components[k],k,k)
    local_resource(k, cmd=cmd)

# install providers
providers = {
    'capmvm': '0.8.0',
}

# helm install for oci targets
for k in providers:
    cmd="helm install {} --version {} --namespace {}-system --set providerArgs.clusterTopology=true --create-namespace oci://ghcr.io/fire-ant/{}".format(k,providers[k],k,k)
    local_resource(k, cmd=cmd)
