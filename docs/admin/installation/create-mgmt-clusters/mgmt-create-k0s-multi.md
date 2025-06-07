# Create and prepare a multinode Kubernetes cluster with k0s

Follow these steps to install and prepare a multinode [k0s kubernetes](https://k0sproject.io) management cluster:

Many features of {{{ docsVersionInfo.k0rdentName }}} require a Kubernetes cluster that uses etcd for its persistence layer. While this is the case for many implementations, the single-node k0s cluster uses SQLite instead. Fortunately, multi-node k0s uses etcd by default, so you can solve this problem by creating controller and worker nodes and linking them together. 

## Create the first controller

1. Download k0s:

    Run the k0s download script to download the latest stable version of k0s and make it executable from `/usr/bin/k0s`:

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo sh
    ```

2. Create a configuration file:

    mkdir -p /etc/k0s
    k0s config create > /etc/k0s/k0s.yaml

3. If you need to access the cluster from outside the specific node on which you're installing it, add the external address, as in:

    ```yaml
    spec:
    api:
        externalAddress: <your-public-ip>
        sans:
        - <your-public-ip>
    ```

    You can also make other configuration changes accodring to the [documentation](https://docs.k0sproject.io/stable/configuration/).

4. Install the controller:

    ```shell
    sudo k0s install controller -c /etc/k0s/k0s.yaml
    sudo k0s start
    ```

    The k0s process acts as a "supervisor" for all of the control plane components. In moments the control plane will be up and running.

## Add workers to the cluster

1. Create a join token:

    To join additional nodes to the cluster, uou need a join token, which is a base64-encoded `KUBECONFIG`. The token embeds information that enables mutual trust between the worker and controller(s) and allows the node to join the cluster.

    To get a worker token, run the following command on newly created existing controller node:

    ```shell
    sudo k0s token create --role=worker > token-file
    ```

    The resulting output is a long token string, which we'll use in a moment to add a worker to the cluster.

    For enhanced security, set an expiration time for the token:

    ```shell
    sudo k0s token create --role=worker --expiry=100h > token-file
    ```

2. Add workers to the cluster:

    To join the worker download k0s to the worker node and run it with the join token you created:

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo sh
    sudo k0s install worker --token-file /path/to/token/file/token-file
    sudo k0s start
    ```

## Add controllers to the cluster

To create a join token for the new controller, follow these steps.

1. Run the following command on an existing controller:

    ```shell
    sudo k0s token create --role=controller --expiry=1h > token-file
    ```
2. Copy the `token-file` and the `k0s.yaml` file to the new controller. Copy `k0s.yaml` to  `/etc/k0s/k0s.yaml`. Each controller in the cluster must have this `k0s.yaml` file, or some cluster nodes will use default config values, which will lead to inconsistency behavior. If your configuration file includes IP addresses (node address, sans, etcd peerAddress), remember to update them accordingly for this specific controller node.

3. On the new controller, run:

    ```shell
    sudo k0s install controller --token-file /path/to/token/file -c /etc/k0s/k0s.yaml
    sudo k0s start
    ```

4. Check k0s status by running:

    ```shell
    sudo k0s status
    ```
    ```console
    Version: v1.33.1+k0s.1
    Process ID: 2769
    Parent Process ID: 1
    Role: controller
    Init System: linux-systemd
    Service file: /etc/systemd/system/k0scontroller.service
    ```

## Access your cluster

Use the Kubernetes 'kubectl' command-line tool that comes with k0s binary to deploy your application or check your node status:

```shell
sudo k0s kubectl get nodes
```
```console
NAME   STATUS   ROLES    AGE    VERSION
k0s    Ready    <none>   4m6s   v1.33.1+k0s
```

Alternatively, to use a tool like Lens or to access the cluster from another machine, copy the KUBECONFIG, which is located at:

```console
/var/lib/k0s/pki/admin.conf
```

to your target machine.  Note that to access the cluster from an external machine you must replace `localhost` in the KUBECONFIG with the host IP address for your controller.