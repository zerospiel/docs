# Helm Values Overrides

This document describes all Helm values that are automatically configured when deploying a cluster or a provider.
To effectively use these values, the `ClusterTemplate`'s or `ProviderTemplate`'s Helm chart must be designed to consume them.
For complete examples of cluster and provider templates that consume these values, refer to the
[templates](https://github.com/k0rdent/kcm/tree/main/templates) in the `k0rdent/kcm` repository.

## `global`

These values are globally configured during KCM installation or in the `Management` object and are automatically passed
to all cluster and provider values.

Supported global configuration options (example of KCM values configuration):

```yaml
controller:
  globalRegistry: my-registry.example.com # Global registry URL
  globalK0sURL: https://my-k0s-url.example.com # Global k0s binary download URL
  k0sURLCertSecret: my-k0s-url-cert-secret # Name of the Secret containing the k0s URL TLS certificate
  registryCredsSecret: my-registry-creds-secret # Name of the Secret containing registry pull credentials
  registryCertSecret: my-registry-cert-secret # Name of a Secret containing the registry root CA with a ca.crt key
  imagePullSecret: my-image-pull-secret # Name of the Secret containing credentials to pull images from private registries, see [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
proxy:
  secretName: my-proxy-secret # Name of the Secret with proxy settings data, see [Proxy Configuration Support](https://docs.k0rdent.io/latest/appendix/proxy/)
enableProvidersReload: true # Enable automatic reload-triggering patches for CAPI provider Deployments, see [Configuring Automatic Provider Reload Annotations](https://docs.k0rdent.io/latest/appendix/appendix-extend-mgmt/?h=#configuring-automatic-provider-reload-annotations)
```

Provider values overrides:

| Path                            | Description                                                               |
|---------------------------------|---------------------------------------------------------------------------|
| `global.registry`               | Global registry URL                                                       |
| `global.imagePullSecrets`       | Global array of image pull secrets to pull images from private registries  |
| `global.proxy`                  | Global proxy configuration                                                |
| `global.enableProvidersReload`  | Global boolean flag to enable automatic providers reload                  |

Cluster values overrides:

| Path                              | Description                                                |
|-----------------------------------|------------------------------------------------------------|
| `global.registry`                 | Global registry URL                                        |
| `global.k0sURL`                   | Global k0s binary download URL                             |
| `global.registryCertSecret`       | Name of the Secret containing the registry TLS certificate |
| `global.k0sURLCertSecret`         | Name of the Secret containing the k0s URL TLS certificate  |
| `global.registryCredentialSecret` | Name of the Secret containing registry pull credentials    |

#### Example

Refer to the [templates](https://github.com/k0rdent/kcm/tree/main/templates) in the `k0rdent/kcm` repository
for examples of how to use global values in cluster and provider templates.

---

## `clusterIdentity`

A reference to the cluster identity resource containing credentials used to access the infrastructure provider.

Always set. Populated from the `Credential` object's `spec.identityRef` associated with the `ClusterDeployment`.

| Path                         | Description                                     |
|------------------------------|-------------------------------------------------|
| `clusterIdentity.apiVersion` | APIVersion of the credential identity reference |
| `clusterIdentity.kind`       | Kind of the credential identity reference       |
| `clusterIdentity.name`       | Name of the credential identity reference       |
| `clusterIdentity.namespace`  | Namespace of the credential identity reference  |

The `clusterIdentity` values must be passed to infrastructure objects. The exact way to use the `clusterIdentity`
values in a cluster template depends on the provider (see the provider documentation for details).

#### Example

##### Values

```yaml
clusterIdentity:
  apiVersion: v1
  kind: Secret
  name: my-credential-secret
  namespace: kcm-system
```

##### Usage (OpenStack)

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: OpenStackCluster
...
spec:
  identityRef:
    name: {{ .Values.clusterIdentity.name  | quote }}
```

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: OpenStackMachineTemplate
...
spec:
  template:
    spec:
      identityRef:
        name: {{ .Values.clusterIdentity.name  | quote }}
```

---

## `clusterLabels`

Set only if `clusterLabels` is **not** already defined in the user-provided `spec.config` values.
Defaults to the `ClusterDeployment` object's own labels.

| Path            | Description                                     |
|-----------------|-------------------------------------------------|
| `clusterLabels` | Labels to be applied to the CAPI cluster object |

#### Example

##### Values

```yaml
clusterLabels:
  k0rdent: demo
```

##### Usage

```yaml
apiVersion: cluster.x-k8s.io/v1beta2
kind: Cluster
metadata:
  {{- if .Values.clusterLabels }}
  labels: {{- toYaml .Values.clusterLabels | nindent 4 }}
  {{- end }}
```

---

## `auth`

See details in [Identity and Authorization Management](../admin/clusters/cluster-iam-setup.md).

Set only when a `ClusterAuthentication` object is referenced in the `ClusterDeployment` spec and
its `spec.authenticationConfiguration` is non-nil. The `auth.configWithAnon` field is additionally set to `true`
only if anonymous authentication is explicitly configured.


| Path                     | Description                                                                                       |
|--------------------------|---------------------------------------------------------------------------------------------------|
| `auth.configSecret.name` | Name of the automatically generated authentication configuration Secret (`<cd-name>-auth-config`) |
| `auth.configSecret.key`  | Key within the Secret containing the authentication config                                        |
| `auth.configSecret.hash` | Hash of the authentication configuration content (for change detection)                           |
| `auth.configWithAnon`    | Boolean indicating anonymous authentication is configured                                         |

#### Example

##### Values

```yaml
auth:
  configSecret:
    name: my-clusterdeployment-auth-config
    key: auth-config.yaml
    hash: 1234567890abcdef
  configWithAnon: true
```

##### Usage

```yaml
apiVersion: controlplane.cluster.x-k8s.io/v1beta1
kind: K0sControlPlane
spec:
  k0sConfigSpec:
    {{- if .Values.auth.configSecret.name }}
    files:
      - contentFrom:
          secretRef:
            name: {{ .Values.auth.configSecret.name  | quote }}
            key: {{ default "config" .Values.auth.configSecret.key  | quote }}
        permissions: "0644"
        {{- if .Values.auth.configSecret.hash }}
        path: /var/lib/k0s/auth/config-{{ .Values.auth.configSecret.hash }}.yaml
        {{- else }}
        path: /var/lib/k0s/auth/config.yaml
        {{- end }}
    {{- end }}
    k0s:
      spec:
        api:
          extraArgs:
            {{- if .Values.auth.configSecret.name }}
            authentication-config: {{ include "authentication-config.fullpath" . }}
            {{- end }}
```
