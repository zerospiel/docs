# Upgrading to {{{ docsVersionInfo.k0rdentName}}} 1.0.0

After upgrading KOF to v1.1.0, please run:

  ```bash
  kubectl apply --server-side --force-conflicts \
    -f https://github.com/grafana/grafana-operator/releases/download/v5.18.0/crds.yaml
  ```

Also run the same for each regional cluster:

  ```bash
  kubectl get secret -n kcm-system $REGIONAL_CLUSTER_NAME-kubeconfig \
    -o=jsonpath={.data.value} | base64 -d > regional-kubeconfig

  KUBECONFIG=regional-kubeconfig kubectl apply --server-side --force-conflicts \
    -f https://github.com/grafana/grafana-operator/releases/download/v5.18.0/crds.yaml
  ```

These steps are required by `grafana-operator`, as noted in its [release notes](https://github.com/grafana/grafana-operator/releases/tag/v5.18.0).
