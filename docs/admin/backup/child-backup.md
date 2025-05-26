# Backing Up Child Clusters

When creating backups using {{{ docsVersionInfo.k0rdent}}}, it's important to know what components are backed up by what.  Specifically:

- {{{ docsVersionInfo.k0rdent}}}'s "backup" capabilities use Velero to back up the management cluster.
- The management cluster includes infrastructure and child cluster definitions.
- The Velero CLI backs up workloads within the child cluster

## The different aspects of backing up a child cluster

So when it comes to backing up a child cluster, there are three things to consider:

1. The ability for the management cluster to manage the child cluster
2. The child cluster itself (including the infrastructure on which it runs)
3. Workloads on the child cluster

Let's look at each of these in turn.

### The ability for the management cluster to manage the child cluster

{{{ docsVersionInfo.k0rdentName }}} manages child clusters through the `ClusterDeployment` CRD 
within the management cluster. This object can be created two ways:

1. If the cluster does not yet exist, the creation of a `ClusterDeployment` object within the management cluster causes the 
   child cluster to be created in the "real world". For example, a `ClusterDeployment` based on the 
   `aws-standalone-cp-{{{ docsVersionInfo.providerVersions.dashVersions.awsHostedCpCluster }}}`
   `ClusterTemplate` results in the creation of a cluster on AWS EC2 nodes. The `ClusterDeployment` can then be used
   for management tasks such as adding services or deleting the cluster.
2. If the cluster already exists, the management cluster can "adopt" it by creating a `ClusterDeployment` based on
   the `adopted-cluster-{{{ docsVersionInfo.providerVersions.dashVersions.adoptedCluster }}}` `ClusterTemplate`. Again, 
   the `ClusterDeployment` can then be used for management tasks such as adding services or deleting the cluster.

So backing up the ability for {{{ docsVersionInfo.k0rdentName }}} to manage the child cluster involves
making sure the management cluster exists and that the `ClusterDeployment` object is present.

{{{ docsVersionInfo.k0rdentName }}}'s backup capabilities handle this.

### The child cluster itself (including the infrastructure on which it runs)

The child cluster -- that is, the servers on which Kubernetes runs and the Kubernetes components themselves -- 
can either be created by {{{ docsVersionInfo.k0rdentName }}}, or it can be created independently and adopted by
{{{ docsVersionInfo.k0rdentName }}}.  Because of these two options, there are two options for backing up these
clusters:

1. When the management cluster is restored, the recreation of `ClusterDeployments` results in the recreation of 
   the clusters.
2. A direct Velero backup can be used to recreate the cluster, which can then be adopted by {{{ docsVersionInfo.k0rdentName }}}.

### Workloads on the child cluster

None of {{{ docsVersionInfo.k0rdentName }}}'s backup capabilities operate on the actual child
cluster, so in order to backup the workloads running on the child cluster, Velero must act on 
it directly. You can do this from the command line.

## Backup use cases

When it comes to backing up and restoring child clusters, there are three situations to consider:

1. The management cluster remains in place, but the child cluster goes down and needs to be recreated.
2. The child cluster remains in place, but the management cluster goes down and needs to be recreated.
3. Both the management cluster and the child cluster go down and need to be recreated.

To accomodate these three use cases, we need to make sure to backup:

1. The management cluster itself. You can do this with {{{ docsVersionInfo.k0rdentName }}}'s backup
   capabilities.
2. The `ClusterDeployment` within the management cluster that represents the child cluster. You can
   also do this with {{{ docsVersionInfo.k0rdentName }}}'s backup capabilities.
3. The workloads running on the child cluster. You will need the Velero CLI to back these up.

Now let's look at how to do this.

## Backing up the management cluster

You can find information on [creating a single backup](ondemand-backups.md) or [scheduling backups](scheduled-backups.md)
within the documentation.

## Backing up the ClusterDeployment objects

By default, `ClusterDeployment` objects are backed up, so you don't need to take any special action.

## Backing up the child cluster workloads

To back up your child cluster's workloads, follow these steps:

1. Install Velero:

    ```shell
    curl -LO https://github.com/vmware-tanzu/velero/releases/download/v1.14.0/velero-v1.14.0-linux-amd64.tar.gz
    tar -xzf velero-v1.14.0-linux-amd64.tar.gz
    sudo mv velero-v1.14.0-linux-amd64/velero /usr/local/bin/velero
    ```

1. Get the `KUBECONFIG` for the child cluster and activate it:

    ```shell
    kubectl -n kcm-system get secret <CLUSTER-NAME>-kubeconfig -o jsonpath='{.data.value}' | base64 -d > <CLUSTER-NAME>.kubeconfig
    export KUBECONFIG=<CLUSTER-NAME>.kubeconfig
    ```

2. Install Velero into the child cluster:

    ```shell
    velero install \
    --namespace backup-info \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:latest \
    --bucket nick-chase-backup-bucket \
    --prefix child-cluster-backups \
    --secret-file ./credentials.txt \
    --use-volume-snapshots=false
    ```
    ```console
    ...
    Deployment/velero: attempting to create resource client
    Deployment/velero: created
    Velero is installed! ⛵ Use 'kubectl logs deployment/velero -n velero' to view the status.
    ```

3. Verify the Velero installation:

    ```shell
    velero version
    ```
    ```console
    Client:
            Version: v1.14.0
            Git commit: 2fc6300f2239f250b40b0488c35feae59520f2d3
    Server:
            Version: v1.14.0
    ```

4. Create credentials and a `BackupStorageLocation` in the child cluster, similar to those you [created for the management cluster](prepare-backups.md).

    ```yaml
    ---
    apiVersion: v1
    data:
        # base64-encoded credentials for Amazon S3 in the following format:
        # [default]
        # aws_access_key_id = EXAMPLE_ACCESS_KEY_ID
        # aws_secret_access_key = EXAMPLE_SECRET_ACCESS_KEY
        cloud: <BASE64 CREDENTIALS>
    kind: Secret
    metadata:
        name: cloud-credentials
        namespace: backup-info
    type: Opaque
    ---
    apiVersion: velero.io/v1
    kind: BackupStorageLocation
    metadata:
        name: aws-s3
        namespace: backup-info
    spec:
    config:
        region: ca-central-1
    default: true # optional, if not set, then storage location name must always be set in ManagementBackup
    objectStorage:
        bucket: <BUCKET_NAME>
        prefix: child-cluster-backups
    provider: aws
    backupSyncPeriod: 1m
    credential:
        name: cloud-credentials
        key: cloud
    ```

velero client config set namespace=velero

kubectl apply -n velero <<EOF
apiVersion: velero.io/v1
kind: VolumeSnapshotLocation
metadata:
  name: default
spec:
  provider: aws
  config:
    region: ca-central-1
EOF

ubuntu@ip-172-31-1-177:~$ kubectl apply -n velero <<EOF
apiVersion: velero.io/v1
kind: VolumeSnapshotLocation
metadata:
  name: default
spec:
  provider: aws
  config:
    region: ca-central-1
EOF
error: must specify one of -f and -k
ubuntu@ip-172-31-1-177:~$ vi vsl.yaml
ubuntu@ip-172-31-1-177:~$ kubectl apply -n backup-info -f vsl.yaml
Warning: resource volumesnapshotlocations/default is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
volumesnapshotlocation.velero.io/default configured
ubuntu@ip-172-31-1-177:~$ velero backup create child-backup-$(date +%Y%m%d%H%M)   --include-namespaces=*   --storage-location aws-s3  --wait
Backup request "child-backup-202505260353" submitted successfully.
Waiting for backup to complete. You may safely press ctrl-c to stop waiting - your backup will continue in the background.
.....
Backup completed with status: Completed. You may check for more information using the commands `velero backup describe child-backup-202505260353` and `velero backup logs child-backup-202505260353`.
ubuntu@ip-172-31-1-177:~$