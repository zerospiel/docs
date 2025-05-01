# Backing up {{{ docsVersionInfo.k0rdentName }}} with Velero

Ensuring you can recover your {{{ docsVersionInfo.k0rdentName }}} environment and the clusters it manages is vital for 
operational resilience. This guide provides a comprehensive walkthrough for setting up Velero to back up both your {{{ docsVersionInfo.k0rdentName }}} management cluster and your child/workload clusters.

## Understanding Velero

Velero works by:

1.  Querying the Kubernetes API server to capture the state of cluster resources (like Deployments, Services, Secrets, ConfigMaps, CRDs, etc.).
2.  Optionally backing up data stored in Persistent Volumes (PVs) using either storage platform snapshots (via CSI) or a file-level backup tool (like Kopia or Restic).
3.  Storing both the resource definitions and volume backups in an external object storage location (like AWS S3, Azure Blob Storage, Google Cloud Storage, or any S3-compatible provider).

## {{{ docsVersionInfo.k0rdentName }}} Backup Strategy

The best backup strategy for you depends on your {{{ docsVersionInfo.k0rdentName }}} architecture:

* **Standalone Managed Clusters:** If {{{ docsVersionInfo.k0rdentName }}} manages standard K0s/Kubernetes clusters, you'll need backups for the {{{ docsVersionInfo.k0rdentName }}} management cluster itself *and* separate backups configured *within each* standalone child cluster.
* **Hosted Control Planes (HCP with k0smotron):** This common {{{ docsVersionInfo.k0rdentName }}} setup requires a layered approach:
    * **Management Cluster Backup:** Protects {{{ docsVersionInfo.k0rdentName }}}, K0smotron, and critically, the *control planes* (including their state stored in etcd PVs within the management cluster) of all your workload clusters. **This is essential for infrastructure recovery.**
    * **Workload Cluster Backup (Recommended):** Protects the *application data* stored in PVs attached to the worker nodes of your workload clusters. **This is essential for application data recovery.**

## Prerequisites

Before you start, ensure you have:

* `kubectl` access configured for the cluster(s) you intend to back up (Management cluster first, then Child/Workload clusters).
* An account with a cloud provider (AWS, Azure, GCP) or access to an S3-compatible object storage system (like MinIO).
* A dedicated object storage bucket/container created in your chosen storage system.
* Credentials (for example, access keys, service principal secrets, service account keys) that grant Velero the necessary permissions to access the object storage bucket/container.

## Understanding Backup Targets

It's crucial to understand what needs to be backed up in each location:

### Management Cluster

**What to back up here:**

* {{{ docsVersionInfo.k0rdentName }}} core components (`kcm`) and configuration.
* Management add-ons (for example, `cert-manager`).
* K0smotron controller and Custom Resources (`Cluster`, `MachinePool`, and so on, defining HCPs).
* Cluster API (CAPI) resources.
* Running components of Hosted Control Planes, such as API server and controller-manager pods (if any).
* **Critically:** The Persistent Volumes (PVs) storing the `etcd` data for each Hosted Control Plane (HCP). These PVs physically reside on the management cluster's storage but contain the state of your workload clusters.

**Purpose of this Backup:** Infrastructure disaster recovery. Enables you to restore the entire {{{ docsVersionInfo.k0rdentName }}} management plane, K0smotron, and the control planes of your workload clusters.

**What's NOT Covered:** Application data stored on PVs within the workload clusters.

### Child / Workload Cluster

**What to Back Up Here:**

* **Critically:** Application data stored in **Persistent Volumes (PVs)** attached to the worker nodes of this specific cluster.
* Application-specific API objects (Deployments, Services, ConfigMaps, Secrets, application-specific CRs) deployed within this cluster's namespaces (provides granular restore capability).

**Purpose of this Backup:** Application data recovery and granular application restores. Ensures that if application data PVs are lost, you can restore the application's state.

**What's NOT Covered:** The control plane state (that's stored in the HCP etcd PVs, backed up from the management cluster in an HCP scenario).

## Setting Up Velero for the Management Cluster (HCP Scenario)

This setup protects {{{ docsVersionInfo.k0rdentName }}}, K0smotron, and the control planes of your workload clusters via their HCPs.

**Goal:** Create reliable, automated backups of the management cluster including HCP etcd PVs.

### Step 1: Choose Backup Storage & Prepare Credentials

1.  **Create a Bucket/Container:** In your chosen cloud provider (such as AWS S3, Azure Blob, GCS) or S3-compatible storage, create a dedicated bucket/container for Velero backups (for example, `{{{ docsVersionInfo.k0rdentName }}}-management-backups`). Note the bucket name and region (if applicable).
2.  **Create Credentials:** Velero needs permissions to write to this bucket. Create credentials according to your provider:

    * **AWS:** Create an IAM user or role with permissions allowing `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket`, `s3:GetBucketLocation`, etc., on the target bucket and objects. (See [Velero AWS Plugin Docs](https://velero.io/docs/main/plugins/aws/) for exact permissions). Save the Access Key ID and Secret Access Key to a local file (e.g., `~/credentials-velero`):
        ```ini
        [default]
        aws_access_key_id=YOUR_ACCESS_KEY_ID
        aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
        ```
    * **Azure:** Create an Azure Service Principal. Grant it the "Storage Blob Data Contributor" role on the storage account or specific container. Get the Tenant ID, Subscription ID, Client ID, and Client Secret. (See [Velero Azure Plugin Docs](https://velero.io/docs/main/plugins/azure/) for details). Save these details to a file (e.g., `~/credentials-velero`).
    * **GCP:** Create a Service Account. Grant it the "Storage Admin" (`roles/storage.admin`) role. Download the service account key file (JSON format, e.g., `~/gcp-key.json`).
    * **S3-Compatible:** Usually uses AWS format credentials file. Check provider docs for specific endpoint/region configuration.

### Step 2: Choose Persistent Volume (PV) Backup Method

This is **critical** for backing up the HCP etcd PVs.

* **Method A: CSI Volume Snapshots (Recommended)**

    * **How it works:** Uses the storage provider's native snapshot capabilities via the Container Storage Interface (CSI). Efficient and fast.
    * **Prerequisites (Check in {{{ docsVersionInfo.k0rdentName }}} Mgmt Cluster):**
        * **CSI Driver:** A CSI driver compatible with your storage must be installed. Check with `kubectl get pods -n kube-system | grep csi`.
        * **VolumeSnapshot CRDs:** Kubernetes Volume Snapshot capabilities must be installed. Check with `kubectl get crd volumesnapshots.snapshot.storage.k8s.io`.
        * **VolumeSnapshotClass:** A `VolumeSnapshotClass` pointing to your CSI driver must exist. Check with `kubectl get volumesnapshotclass`.

* **Method B: File-Based Backup (Kopia Recommended)**

    * **How it works:** Velero integrates with Kopia (or Restic) which runs a helper container (in a DaemonSet) on each node to copy file data directly from the volume into the backup storage location.
    * **Prerequisites:** Generally none, works with most volume types.
    * **Considerations:** Can be slower and consume more CPU/Memory/Network resources on cluster nodes during backup compared to CSI snapshots. Kopia is generally preferred over the older Restic integration.

### Step 3: Install Velero CLI

Install the Velero command-line tool on your local machine where you run `kubectl`. Follow instructions at: [https://velero.io/docs/main/basic-install/#install-the-cli](https://velero.io/docs/main/basic-install/#install-the-cli)
Verify with `velero version`.

### Step 4: Install Velero Server in {{{ docsVersionInfo.k0rdentName }}} Management Cluster

Run `velero install` targeting your {{{ docsVersionInfo.k0rdentName }}} management cluster context. Adapt placeholders (`<...>`) carefully for the appropriate environment.

#### AWS with CSI Snapshots

    ```shell
    velero install \
        --provider aws \
        --plugins velero/velero-plugin-for-aws,velero/velero-plugin-for-csi \
        --bucket <YOUR_S3_BUCKET_NAME> \
        --backup-location-config region=<YOUR_BUCKET_REGION> \
        --snapshot-location-config region=<YOUR_CSI_REGION> \
        --secret-file </path/to/your/credentials-velero> \
        --use-volume-snapshots=true \
        --namespace velero # Default namespace
    ```

#### AWS with Kopia File-Based Backup

    ```shell
    velero install \
        --provider aws \
        --plugins velero/velero-plugin-for-aws \
        --bucket <YOUR_S3_BUCKET_NAME> \
        --backup-location-config region=<YOUR_BUCKET_REGION> \
        --secret-file </path/to/your/credentials-velero> \
        --use-volume-snapshots=false \ # Disable CSI snapshots
        --use-kopia \ # Enable Kopia integration
        --namespace velero
    ```

#### Azure with CSI Snapshots

    ```shell
    # Ensure AZURE_CLIENT_SECRET env var is set or included in the file
    # See Azure plugin docs for credential file details
    velero install \
        --provider azure \
        --plugins velero/velero-plugin-for-azure,velero/velero-plugin-for-csi \
        --bucket <YOUR_BLOB_CONTAINER_NAME> \
        --secret-file </path/to/your/azure-credentials.txt> \
        --backup-location-config resourceGroup=<YOUR_RG_NAME>,storageAccount=<YOUR_STORAGE_ACCOUNT_NAME>,subscriptionId=<YOUR_SUBSCRIPTION_ID> \
        --snapshot-location-config apiTimeout=5m,resourceGroup=<YOUR_RG_NAME>,subscriptionId=<YOUR_SUBSCRIPTION_ID> \
        --use-volume-snapshots=true \
        --namespace velero
    ```

#### GCP with CSI Snapshots

    ```shell
    velero install \
        --provider gcp \
        --plugins velero/velero-plugin-for-gcp,velero/velero-plugin-for-csi \
        --bucket <YOUR_GCS_BUCKET_NAME> \
        --secret-file </path/to/your/gcp-key.json> \
        --use-volume-snapshots=true \
        --namespace velero
    ```

#### Key Flags Explained:
    * `--provider`: Your storage provider (aws, azure, gcp).
    * `--plugins`: Required Velero plugins (provider plugin is always needed, add CSI plugin if using snapshots).
    * `--bucket`: Name of your storage bucket/container.
    * `--secret-file`: Path to the credential file you created.
    * `--backup-location-config`: Provider-specific settings (e.g., region, resource group).
    * `--snapshot-location-config`: Provider-specific settings for volume snapshots (if using CSI).
    * `--use-volume-snapshots`: Set to `true` for CSI, `false` otherwise.
    * `--use-kopia` / `--use-restic`: Enable file-based PV backup.
    * `--namespace`: Kubernetes namespace to install Velero into (default `velero`).

### Step 5: Verify Velero Installation

1.  **Check Pods:** 
   
     Ensure Velero and any helper pods (Kopia/Restic) are running:
     ```bash
     kubectl get pods -n velero
     # Expected: velero-xxxx deployment pod running.
     # If using Kopia/Restic: kopia-maintainer-xxxx or restic-xxxx pods (one per node) running.
     ```

2.  **Check Locations:** Verify Velero can connect to your storage:

     ```bash
     velero backup-location get
     # Should show Phase: Available

     # If using CSI:
     velero volume-snapshot-location get
     # Should show Phase: Available
     ```
3.  **Troubleshooting:** 

     If locations are not available, check Velero pod logs: `kubectl logs deployment/velero -n velero`. Common issues are incorrect credentials or permissions.

### Step 6: Identify HCP etcd PVs (Crucial Sanity Check)

While Velero (especially with CSI) often automatically discovers PVs, it's good practice to confirm the etcd PVs are eligible.

1.  Find the namespaces used by your HCPs (often related to your K0smotron `Cluster` resource names).
2.  List PVCs in those namespaces:

     ```shell
     kubectl get pvc -n <hcp-namespace-1>
     kubectl get pvc -n <hcp-namespace-2>
     # ... etc.
     ```

3.  Look for PVCs associated with etcd StatefulSets (for example, `data-etcd-main-0`).
4.  Check the `STORAGECLASS` used by these PVCs (`kubectl get pvc <pvc-name> -n <hcp-namespace> -o jsonpath='{.spec.storageClassName}'`). Ensure this StorageClass is compatible with your chosen PV backup method (has a `VolumeSnapshotClass` for CSI, or is generally accessible for Kopia/Restic).

### Step 7: Create the Management Backup

Perform an initial backup manually:

```shell
# Generate a unique name with timestamp
BACKUP_NAME="{{{ docsVersionInfo.k0rdentName }}}-mgmt-$(date +%Y%m%d-%H%M%S)"

velero backup create ${BACKUP_NAME} \
    --include-cluster-resources=true \ # MUST INCLUDE: Backs up CRDs, Nodes, ClusterRoles etc.
    --include-crds=true \ # MUST INCLUDE: Ensures CRDs (like K0smotron/CAPI) are backed up
    # Optional: Specify namespaces if needed, otherwise defaults to all
    # --include-namespaces velero,kube-system,<hcp-namespace-1>,...
    # Optional but Recommended if using Kopia/Restic for *all* PVs:
    # --default-volumes-to-kopia=true # Tells Velero to use Kopia for PVs unless opted out
    --wait # Waits for the command to complete and shows final status
```

When the backup is complete, be sure to verify it:

```shell
velero backup describe ${BACKUP_NAME}
# Look for Phase: Completed, check Errors/Warnings are 0.
# Check for 'Persistent Volumes' section (if using Kopia/Restic)
# Check for 'Volume Snapshots' details (if using CSI)

velero backup logs ${BACKUP_NAME}
# Look for errors or details about PV backups.
```

### Step 8: Schedule Regular Backups

1. Automate backups using schedules:

    ```shell
    velero schedule create {{{ docsVersionInfo.k0rdentName }}}-mgmt-daily \
        --schedule="@daily" \ # Or other cron expression e.g., "0 3 * * *" for 3 AM daily
        --include-cluster-resources=true \
        --include-crds=true \
        # --default-volumes-to-kopia=true # If using Kopia for PVs
        --ttl 240h # Example: Retain backups for 10 days (240 hours)
        # Optional: --include-namespaces, --exclude-namespaces
    ```

2. Check schedules: 

    ```shell
    velero schedule get
    ```

3. Check backups created by schedule: 

    ```shell
    velero backup get
    ```

## Setting Up Velero for Child/Workload Clusters

This setup protects application data within standalone or HCP workload clusters.

**Goal:** Create reliable, automated backups of application namespaces and their Persistent Volumes within each child/workload cluster.

**Process:**

The process largely mirrors the Management Cluster setup (Steps 1-5, 7-8), but is performed **inside each child/workload cluster**:

1.  **Target the Child/Workload Cluster:** Ensure your `kubectl` context points to the specific child/workload cluster you want to back up.
2.  **Storage & Credentials:** You *can* use the same backup bucket, but consider using a different prefix or subdirectory for organizational purposes (configure via `--backup-location-config prefix=...`). Use the same or different credentials file as appropriate.
3.  **Choose PV Method:** Determine if CSI snapshots or Kopia/Restic is appropriate based on the storage and CSI setup *within this specific child/workload cluster*.
4.  **Install Velero CLI:** Already done.
5.  **Install Velero Server:** Run `velero install` targeting the child/workload cluster, adapting provider, plugins, bucket, credentials, PV method flags (`--use-volume-snapshots`, `--use-kopia`) based on *this cluster's* environment.
6.  **Verify Installation:** Run `kubectl get pods -n velero`, `velero backup-location get`, etc., targeting the child/workload cluster.
7.  **Create Application Backup:**

        * Focus on application namespaces.
        ```shell
        # Backup specific application namespace(s)
        APP_BACKUP_NAME="<app-name>-$(date +%Y%m%d-%H%M%S)"
        velero backup create ${APP_BACKUP_NAME} \
            --include-namespaces <your-app-namespace-1>,<your-app-namespace-2> \
            # --default-volumes-to-kopia=true # If using Kopia for app PVs
            --wait
        ```
        * Verify the backup (`velero backup describe ...`) ensuring application PVs were captured.

8.  **Schedule Application Backups:**
        ```shell
        velero schedule create <app-name>-daily \
            --schedule="@daily" \
            --include-namespaces <your-app-namespace-1>,<your-app-namespace-2> \
            # --default-volumes-to-kopia=true # If using Kopia
            --ttl 168h # Example: Retain for 7 days
        ```

Repeat this setup process for each child/workload cluster requiring backup.

## What Gets Backed Up (Summary)

* **Management Cluster Backup Includes:**

    * {{{ docsVersionInfo.k0rdentName }}} state (via API objects, etcd PV if applicable).
    * K0smotron controller & CRDs.
    * HCP definitions (CAPI CRs, K0smotron CRs like `Cluster`, `MachinePool`).
    * **HCP etcd Persistent Volumes** (containing workload cluster state).
    * Other management components like parts of `cert-manager`.
    * Potentially objects matching specific labels used by {{{ docsVersionInfo.k0rdentName }}}'s default configuration (see label list below).
    * *Excludes:* Application data PVs attached to workload cluster nodes.

    *Default Labels (for API Object Selection):* {{{ docsVersionInfo.k0rdentName }}}'s configuration might prioritize API objects with these labels during backup:
    ```yaml
    # Identifies objects belonging to a specific HCP managed by Cluster API
    cluster.x-k8s.io/cluster-name: "<cluster-deployment-name>"
    # Identifies objects related to Flux Helm releases for the HCP
    helm.toolkit.fluxcd.io/name: "<cluster-deployment-name>"
    # Identifies CAPI provider components (bootstrap, control plane, infrastructure)
    cluster.x-k8s.io/provider: "bootstrap-<provider>"
    cluster.x-k8s.io/provider: "control-plane-<provider>"
    cluster.x-k8s.io/provider: "infrastructure-<provider>"
    # Identifies core CAPI components
    cluster.x-k8s.io/provider: "cluster-api"
    # Identifies specific cert-manager resources
    controller.cert-manager.io/fao: "true"
    # Identifies {{{ docsVersionInfo.k0rdentName }}} KCM components
    [{{{ docsVersionInfo.k0rdentName }}}.mirantis.com/component](https://{{{ docsVersionInfo.k0rdentName }}}.mirantis.com/component): "kcm"
    ```
    (Remember: Label selection alone is NOT enough; PV backup **must** be configured correctly for HCP etcd PVs).

* **Child/Workload Cluster Backup Includes:**

    * API Objects within specified namespaces (Deployments, Services, ConfigMaps, Secrets, App-specific CRs).
    * **Application Data Persistent Volumes** associated with the included namespaces/pods.
    * *Excludes:* Control plane state (this is in the HCP etcd PVs backed up from the management cluster).

## Important Considerations & Next Steps

* **TEST YOUR RESTORES:** Backups are only useful if they can be restored successfully. Regularly practice restoring the management cluster to a non-production environment and restoring applications in workload clusters. Use `velero restore create --from-backup <backup-name>`.
* **Permissions (RBAC):** Velero's Service Account typically needs cluster-admin-like privileges to read all necessary resources and potentially manage snapshots or exec into pods (for Restic/Kopia). Review permissions if your cluster has strict RBAC.
* **Resource Usage:** Kopia/Restic backups consume node CPU, Memory, and Network I/O. Monitor node resource usage during backups, especially on heavily loaded clusters. CSI snapshots are generally much lighter.
* **Monitoring Backups:** Regularly check backup status using `velero backup get` and `velero schedule get`. Set up alerts if backups fail.
* **Security:** Protect your backup storage credentials securely. Ensure your backup bucket/container has appropriate access controls (e.g., private, limited access).
* **Velero Documentation:** For advanced configuration, troubleshooting, and restore procedures, consult the official Velero documentation: [https://velero.io/docs/main/](https://velero.io/docs/main/)

By following this guide, you can establish a robust backup strategy for your {{{ docsVersionInfo.k0rdentName }}} environment using Velero, significantly improving your ability to recover from failures.
