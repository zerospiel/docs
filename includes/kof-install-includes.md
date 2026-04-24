<!--image-registry-start-->
<!--image-registry-end-->

<!--airgap-start-->
<!--airgap-end-->

<!--install-istio-start-->
    ```bash
    helm upgrade -i --reset-values --wait --create-namespace -n istio-system k0rdent-istio \
      {{{ docsVersionInfo.kofVersions.kofOciRegistryBaseIstio }}}/charts/k0rdent-istio \
      --version 0.4.0 \
      --set cert-manager-service-template.enabled=false \
      --set "istiod.meshConfig.extensionProviders[0].name=otel-tracing" \
      --set "istiod.meshConfig.extensionProviders[0].opentelemetry.port=4317" \
      --set "istiod.meshConfig.extensionProviders[0].opentelemetry.service=kof-collectors-daemon-collector.kof.svc.cluster.local" \
      --set-json 'gateway.resource.spec.servers[0]={"port":{"number":15443,"name":"tls","protocol":"TLS"},"tls":{"mode":"AUTO_PASSTHROUGH"},"hosts":["{clusterName}-vmauth.kof.svc.cluster.local"]}'
    ```
<!--install-istio-end-->

<!--install-kof-start-->
  ```bash
  helm upgrade -i --reset-values \
    --create-namespace -n kof kof \
    -f kof-values.yaml \
    {{{ docsVersionInfo.kofVersions.kofOciRegistryBase }}}/charts/kof \
    --version {{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}
  ```
<!--install-kof-end-->

<!--opencost-openstack-start-->
    ??? note "OpenCost for OpenStack and other unsupported providers"

        Workaround for [opencost/opencost issue #2925](https://github.com/opencost/opencost/issues/2925)
        to avoid `Error getting LoadBalancer cost: strconv.ParseFloat: parsing "": invalid syntax`:

        ```yaml
        k0rdent.mirantis.com/kof-collectors-values: |
          opencost:
            opencost:
              customPricing:
                enabled: true
                costModel:
                  firstFiveForwardingRulesCost: 0
                  additionalForwardingRuleCost: 0
                  LBIngressDataCost: 0
        ```

        Please also consider configuring cost of [default](https://github.com/opencost/opencost-helm-chart/blob/opencost-2.4.0/charts/opencost/values.yaml#L329-L339)
        and [advanced](https://github.com/opencost/opencost/blob/v1.118.0/pkg/cloud/models/models.go#L121-L198) resources.
<!--opencost-openstack-end-->

<!--grafana-intro-start-->
> NOTE:
> Grafana installation and automatic configuration are now disabled in KOF by default.
> Please check the [Using KOF without Grafana](../docs/admin/kof/kof-using.md) and [Grafana in KOF](../docs/admin/kof/kof-grafana.md) guides.
<!--grafana-intro-end-->
