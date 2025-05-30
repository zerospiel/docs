## Extended Management Configuration

{{{ docsVersionInfo.k0rdentName }}} is deployed with the following default configuration, which may vary
depending on the release version:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: Management
metadata:
  name: kcm
spec:
  core:
    capi: {}
    kcm: {}
  providers:
  - name: k0smotron
  - name: cluster-api-provider-aws
  - name: cluster-api-provider-azure
  - name: cluster-api-provider-openstack
  - name: cluster-api-provider-vsphere
  - name: projectsveltos
  release: kcm-0-0-7
```

As you can see, the `Management` object defines the providers that are available from within {{{ docsVersionInfo.k0rdentName }}}. Some of these are
providers directly used by the user, such as aws, azure, and so on, and others are used internally
by {{{ docsVersionInfo.k0rdentName }}}, such as Sveltos.

To see what is included in a specific release, look at the `release.yaml` file in the tagged release.
For example, here is the [v0.0.7 release.yaml](https://github.com/k0rdent/kcm/releases/download/v0.0.7/release.yaml).


{{{ docsVersionInfo.k0rdentName }}} allows you to customize its default configuration by modifying the spec of the `Management` object.
This enables you to manage the list of providers to deploy and adjust the default settings for core components.

For detailed examples and use cases, refer to [Examples and Use Cases](#examples-and-use-cases)

## Configuration Guide

There are two options to override the default management configuration of {{{ docsVersionInfo.k0rdentName }}}:

1. Update the `Management` object after the {{{ docsVersionInfo.k0rdentName }}} installation using `kubectl`:

    `kubectl --kubeconfig <path-to-management-kubeconfig> edit management`

2. Deploy {{{ docsVersionInfo.k0rdentName }}} skipping the default `Management` object creation and provide your
   own `Management` configuration:

    - Create `management.yaml` file and configure core components and providers.
      For example:

           ```yaml
           apiVersion: k0rdent.mirantis.com/v1beta1
           kind: Management
           metadata:
             name: kcm
           spec:
             core:
               capi: {}
               kcm:
                 config:
                   controller:
                     defaultRegistryURL: "oci://ghcr.io/my-oci-registry-name/kcm/charts"
             providers:
             - name: k0smotron
             - name: cluster-api-provider-aws
             - name: projectsveltos
             release: kcm-0-0-7
           ```
      In the example above, the `Management` object is configured with custom registry settings for the KCM controller
      and a reduced list of providers.

    - Specify `--create-management=false` controller argument and install {{{ docsVersionInfo.k0rdentName }}}:
      If installing using `helm` add the following parameter to the `helm
      install` command:

        ```shell
        --set="controller.createManagement=false"
        ```

    - Create `kcm` `Management` object after {{{ docsVersionInfo.k0rdentName }}} installation:

           ```bash
           kubectl --kubeconfig <path-to-management-kubeconfig> create -f management.yaml
           ```

You can customize the default configuration options for core components by updating the
`.spec.core.<core-component-name>.config` section in the `Management` object. For example, to override the default
settings for the KCM component, modify the `spec.core.kcm.config` section. To view the complete list of configuration
options available for kcm, refer to:
[KCM Configuration Options for {{{ docsVersionInfo.k0rdentName }}} v0.0.7](https://github.com/k0rdent/kcm/blob/v0.0.7/templates/provider/kcm/values.yaml)
(Replace v0.0.7 with the relevant release tag for other {{{ docsVersionInfo.k0rdentName }}} versions).

To customize the list of providers to deploy, update the `.spec.providers` section. You can add or remove providers
and configure custom templates for each provider. Each provider in the list must include the `name` field
and may include the `template` and `config` fields:

```yaml
- name: <provider-name> 
  template: <provider-template> # optional. If omitted, the default template from the `Release` object will be used
  config: {} # optional provider configuration containing provider Helm Chart values in YAML format
```
## Examples and Use Cases
### Configuring a Custom OCI Registry for KCM components
You can override the default registry settings in {{{ docsVersionInfo.k0rdentName }}} by specifying the `defaultRegistryURL`, `insecureRegistry`,
and `registryCredsSecret` parameters under `spec.core.kcm.config.controller`:

* `defaultRegistryURL`: Specifies the registry URL for downloading Helm charts representing templates. 
Use the `oci://` prefix for OCI registries. Default: `oci://ghcr.io/k0rdent/kcm/charts`.
* `insecureRegistry`: Allows connecting to an HTTP registry. Default: `false`.
* `registryCredsSecret`: Specifies the name of a Kubernetes Secret containing authentication credentials for the 
registry (optional). This Secret should exist in the system namespace (default: `kcm-system`).

Example Configuration:

```yaml
spec:
  core:
    kcm:
      config:
        controller:
          defaultRegistryURL: "oci://ghcr.io/my-private-oci-registry-name/kcm/charts"
          insecureRegistry: true
          registryCredsSecret: my-private-oci-registry-creds
```

Example of a Secret with Registry Credentials:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-private-oci-registry-creds
  namespace: kcm-system
stringData:
  username: "my-user-123"
  password: "my-password-123"
```

The KCM controller will create the default [HelmRepository](https://fluxcd.io/flux/components/source/helmrepositories/)
using the provided configuration and fetch KCM components from this repository. For the example above,
the following `HelmRepository` will be created:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  labels:
    k0rdent.mirantis.com/managed: "true"
  name: my-private-oci-registry-name
  namespace: kcm-system
spec:
  insecure: true
  interval: 10m0s
  provider: generic
  type: oci
  url: oci://ghcr.io/my-private-oci-registry-name/kcm/charts
  secretRef:
    name: my-private-oci-registry-creds
```

### Configuring a Custom Image for KCM controllers

You can override the default image for the KCM controllers by specifying the `repository`, `tag` and `pullPolicy`
parameters under `spec.core.kcm.config.image`: 

Example Configuration:

```yaml
spec:
  core:
    kcm:
      config:
        image:
          repository: ghcr.io/my-custom-repo/kcm/controller
          tag: v0.0.7
          pullPolicy: IfNotPresent
```

### Configuring manager settings for CAPI providers

Starting from k0rdent `v0.3.0`, configuring manager settings for CAPI providers is supported. You can override
these settings by defining the `spec.providers[*].config.manager` section. The values under the `manager` section should
follow the format described here:
https://github.com/kubernetes-sigs/cluster-api-operator/blob/v0.18.1/api/v1alpha2/provider_types.go#L126.

> WARNING: This is not supported for the `k0sproject-k0smotron` provider due to a bug in the CAPI Operator:
> [CAPI operator incorrectly finds the manager container if the number of containers is >1](https://github.com/kubernetes-sigs/cluster-api-operator/issues/787).

For example, to override feature gates for the Cluster API Provider AWS, configure the following:

```yaml
spec:
  providers:
  - name: cluster-api-provider-aws
    config:
      manager:
        featureGates:
          MachinePool: true
          EKSEnableIAM: true
          EKSAllowAddRoles: true
```
