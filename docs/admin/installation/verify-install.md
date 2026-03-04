# Confirming the deployment

> NOTE:
> After running the helm install command, please wait 5 to 10 minutes for the deployment to stabilize.

To understand whether installation is complete, start by making sure all pods are ready in the `kcm-system` namespace. There should be 21 pod entries:

```bash
kubectl get pods -n kcm-system
```

```console { .no-copy }
NAME                                                           READY   STATUS    RESTARTS   AGE
azureserviceoperator-controller-manager-557c4bc8dc-shg4x       1/1     Running   0          2m22s
azureserviceoperator-controller-manager-557c4bc8dc-tfqm6       1/1     Running   0          2m22s
capa-controller-manager-6f4db96656-tzjrp                       1/1     Running   0          2m2s
capd-controller-manager-5977fc677-jql8m                        1/1     Running   0          117s
capg-controller-manager-668955d6f9-h94ff                       1/1     Running   0          110s
capi-controller-manager-66ccd9d8dd-w8bg7                       1/1     Running   0          3m12s
capi-ipam-in-cluster-controller-manager-56c7bd877c-wjc5l       1/1     Running   0          2m50s
capi-ipam-infoblox-controller-manager-548877ddb5-5xcvs         1/1     Running   0          2m48s
capk-controller-manager-769dc57dc8-r8sqk                       1/1     Running   0          104s
capo-controller-manager-7b55784585-fhjpb                       1/1     Running   0          108s
capv-controller-manager-64664f5cf4-s244l                       1/1     Running   0          119s
capz-controller-manager-d849cfbc6-5zkrt                        1/1     Running   0          2m22s
helm-controller-5466948d9f-746fm                               1/1     Running   0          5m40s
k0smotron-controller-manager-bootstrap-8658fbfcb8-t98zt        1/1     Running   0          2m53s
k0smotron-controller-manager-control-plane-595fdc8c84-vh67z    1/1     Running   0          2m47s
k0smotron-controller-manager-infrastructure-5bcdbb4f78-5mhc8   1/1     Running   0          2m8s
kcm-cert-manager-69b946fdb9-2h9bd                              1/1     Running   0          5m40s
kcm-cert-manager-cainjector-757fd85d77-tj54b                   1/1     Running   0          5m40s
kcm-cert-manager-webhook-59f6c49756-5bprs                      1/1     Running   0          5m40s
kcm-cluster-api-operator-bdb86f9fc-d2w2c                       1/1     Running   0          3m49s
kcm-controller-manager-5b6675b94f-x8cwf                        1/1     Running   0          3m49s
kcm-rbac-manager-7579db64d9-tb7gx                              1/1     Running   0          5m40s
kcm-regional-telemetry-69f8ddbc9-mr6sc                         1/1     Running   0          5m40s
kcm-reloader-fc9cbb8cf-rsb98                                   1/1     Running   0          5m40s
source-controller-597784bdbb-rwsqf                             1/1     Running   0          5m40s
velero-b7b646967-zwbxv                                         1/1     Running   0          5m40s
```

```bash
kubectl get pods -n kcm-system --no-headers | wc -l
```

```console { .no-copy }
26
```

State management is handled by Project Sveltos, so you'll want to make sure that all 10 pods are running/completed in the `projectsveltos` namespace:

```bash
kubectl get pods -n projectsveltos
```

```console { .no-copy }
NAME                                      READY   STATUS    RESTARTS   AGE
access-manager-74b7c98d8b-npj6t           1/1     Running   0          20m
addon-controller-6ddb848fdf-5vpnc         1/1     Running   0          20m
classifier-manager-57d5779966-ft9vs       1/1     Running   0          20m
event-manager-5569df975f-4cgrf            1/1     Running   0          20m
hc-manager-66c559fff6-l6xnx               1/1     Running   0          20m
mcp-server-55459fdccf-sg4lj               1/1     Running   0          20m
sc-manager-56ccc48477-lvz66               1/1     Running   0          20m
shard-controller-6b65cd4f8f-kmpps         1/1     Running   0          20m
sveltos-agent-manager-97d78f4bb-hqjm6     1/1     Running   0          19m
techsupport-controller-797459769b-wwg6h   1/1     Running   0          20m
```

```bash
kubectl get pods -n projectsveltos --no-headers | wc -l
```

```console { .no-copy }
10
```

If any of these pods are missing, simply give {{{ docsVersionInfo.k0rdentName }}} more time. If there's a problem, you'll see pods crashing and restarting, and you can see what's happening by describing the pod, as in:

```bash
kubectl describe pod classifieclassifier-manager-5b47b66fc9-5mtwl -n projectsveltos
```

As long as you're not seeing pod restarts, you just need to wait a few minutes.

## Verify the templates

Next verify whether the KCM templates have been successfully installed and reconciled. Start with the `ProviderTemplate` objects:

```bash
kubectl get providertemplate -n kcm-system
```
```console { .no-copy }
NAME                                   VALID
cluster-api-{{{ extra.docsVersionInfo.providerVersions.dashVersions.clusterApi }}}                                 true
cluster-api-provider-aws-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderAws }}}                   true
cluster-api-provider-azure-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderAzure }}}                 true
cluster-api-provider-docker-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderDocker }}}                true
cluster-api-provider-gcp-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderGcp }}}                   true
cluster-api-provider-infoblox-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderInfoblox }}}               true
cluster-api-provider-ipam-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderIpam }}}                   true
cluster-api-provider-k0sproject-k0smotron-{{{ docsVersionInfo.providerVersions.dashVersions.k0smotron }}}  true
cluster-api-provider-openstack-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderOpenstack }}}             true
cluster-api-provider-vsphere-{{{ docsVersionInfo.providerVersions.dashVersions.clusterApiProviderVsphere }}}               true
kcm-{{{ docsVersionInfo.k0rdentVersion }}}                                         true
kcm-regional-{{{ docsVersionInfo.providerVersions.dashVersions.regional }}}                                true
projectsveltos-{{{ docsVersionInfo.providerVersions.dashVersions.sveltosProvider }}}                              true
```

Make sure that all templates are not just installed, but valid. Again, this may take a few minutes.

You'll also want to make sure the `ClusterTemplate` objects are installed and valid:

```bash
kubectl get clustertemplate -n kcm-system
```
```console { .no-copy }
NAME                             VALID
adopted-cluster-{{{ docsVersionInfo.providerVersions.dashVersions.adoptedCluster }}}            true
aws-eks-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsEksCluster }}}                    true
aws-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsHostedCpCluster }}}             true
aws-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsStandaloneCpCluster }}}         true
azure-aks-{{{ extra.docsVersionInfo.providerVersions.dashVersions.azureAksCluster }}}                  true
azure-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.azureHostedCpCluster }}}           true
azure-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.azureStandaloneCpCluster }}}       true
docker-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.azureHostedCpCluster }}}          true
gcp-gke-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsEksCluster }}}                    true
gcp-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsHostedCpCluster }}}             true
gcp-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsStandaloneCpCluster }}}         true
openstack-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.openstackStandaloneCpCluster }}}       true
openstack-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.openstackStandaloneCpCluster }}}   true
remote-cluster-{{{ extra.docsVersionInfo.providerVersions.dashVersions.remoteCluster }}}            true
vsphere-hosted-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.vsphereHostedCpCluster }}}         true
vsphere-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.vsphereStandaloneCpCluster }}}     true
```

## Verify {{{ docsVersionInfo.k0rdentName }}} status

The final test of whether {{{ docsVersionInfo.k0rdentName }}} installation is installed is making sure the
status of the `Management` object itself is `True`:

```bash
kubectl get management -n kcm-system
```
```console { .no-copy }
NAME   READY   RELEASE         AGE
kcm    True    kcm-{{{ docsVersionInfo.k0rdentVersion}}}   18m
```
