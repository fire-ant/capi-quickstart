allow_k8s_contexts("default")
load('ext://cert_manager', 'deploy_cert_manager')
load('ext://namespace', 'namespace_create')
load('ext://helm_remote', 'helm_remote')

# install cert manager
deploy_cert_manager(
    version='v1.11.0'
)

# capi-operator version
components = {
    'capi-operator': 'v0.1.0',
}
# helm install for oci targets
for k in components:
    cmd="helm upgrade --install {} --version {} --namespace {}-system --create-namespace  oci://ghcr.io/fire-ant/{}".format(k,components[k],k,k,k)
    local_resource(k, cmd=cmd)
    local_resource("capi-op-check",
    cmd="kubectl wait --for=condition=Available deployments {}-controller-manager -n {}-system".format(k,k)
    )

# crds for cluster api components grouped for deps
k8s_yaml("providers/capi-crds.yaml")
k8s_resource(
objects=[
    'cluster-api:CoreProvider:capi-system',
    'kubeadm:ControlPlaneProvider:capi-kubeadm-control-plane-system',
    'kubeadm:BootstrapProvider:capi-kubeadm-bootstrap-system'
    ],
    resource_deps=['capi-op-check'],
new_name='capi',
)

# load infra providers from map
def provider(infra, version):
    cmd="helm upgrade --install {} --version {} --namespace {}-system --create-namespace oci://ghcr.io/fire-ant/{}".format(infra,version,infra,infra)
    local_resource(infra, cmd=cmd, resource_deps=['capi'])

def flux_install(
    base_components="source-controller,kustomize-controller,helm-controller",
    add_components="",
    all_namespaces=True):
    if all_namespaces == "":
        fail("this must be set to a boolean value of true|false")
    if base_components == []:
        fail("this must be a list of strings, accepts comma-separated value")

    cmd = "flux install --components={base_components} --components-extra={add_components} --watch-all-namespaces={all_namespaces} ".format(
        base_components = base_components,
        add_components = add_components,
        all_namespaces = all_namespaces
    )
    local(cmd)
def gitops_ui():
# PASSWORD=WWGit0ps!
    wgvalues=[
            "service.type=NodePort",
            "adminUser.create=true",
            "adminUser.username=admin",
            "adminUser.createClusterRole=true",
            "adminUser.createSecret=true",
            "adminUser.passwordHash=$2a$10$iBSzHfPmZzNYy1WDrC4WNu4OfJDcJfDNh801BwoqA2GgwqjNb9lsu"
            ]
    helm_remote('weave-gitops',
        namespace="flux-system",
        set=wgvalues,
        repo_name='ww-gitops',
        version="4.0.13 ",
        repo_url='https://helm.gitops.weave.works'
        )
    k8s_resource(
    workload='weave-gitops',
    links=[
        'ui',
        link('weave-gitops', 'http://localhost:9001'),
    ],
    port_forwards=[port_forward(9001, 9001)]
    )

config.define_string("provider")
config.define_bool("ui")
ui = config.parse()["ui"]
infra = config.parse()["provider"]
if infra !="":
    if infra == "docker" :
        k8s_yaml("providers/capd-crd.yaml")
        k8s_resource(
            objects=[
                'docker:InfrastructureProvider:capd-system'
                ],
            new_name='capd',
            resource_deps=['capi-operator','capi']
            )
    if infra == "microvm" :
        p = "capmvm"
        v = "v0.8.0"
        # cmd="vagrant up"
        # local_resource("flintlock", cmd=cmd)
        provider(p,v)
    if infra == "vcluster" :
        p = "capvc"
        v = "0.1.3"
        provider(p,v)
if ui == True :
    flux_install()
    gitops_ui()
