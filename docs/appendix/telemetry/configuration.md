# Configuring telemetry

Telemetry can be configured either via the helm values during the installation
via `--set` flag
or via the [Management](../../reference/crds/index.md#management) object as
shown in the [Extended Management Configuration](../appendix-extend-mgmt.md#configuring-telemetry).

## Main settings block

Telemetry is configured under the `telemetry` block:

```yaml
telemetry:
  mode: online            # disabled | local | online (default: online)
  concurrency: 5          # scrape N clusters in parallel (1..100)
  interval: 24h           # scrape cadence
  jitter: 10              # % jitter (1..99)
```

* `mode` controls collection and storage mechanism. `online` sends to Segment;
  `local` writes files; `disabled` turns it off. *(Default: `online`)*
* `concurrency` is how many child clusters are scraped in parallel.
* `interval` is the scrape frequency.
* `jitter` spreads scrapes across time to avoid thundering herds.

## Local mode storage settings

When `telemetry.mode: local`, configure where and how files are stored:

```yaml
telemetry:
  mode: local
  local:
    baseDir: /var/lib/telemetry
    volume:
      source: hostPath     # pvc | existing | hostPath (default)
      # For source: pvc (dynamic provisioning)
      pvc:
        storageClassName: ""        # default StorageClass if empty
        accessModes: [ReadWriteOnce]
        volumeMode: Filesystem
        size: 200Mi
        annotations: {}
      # For source: existing (use pre-created PVC)
      existingClaim: ""
      # For source: hostPath (static PV + matching PVC)
      hostPath:
        name: ""                    # defaults to <fullname>-telemetry-data
        reclaimPolicy: Retain
        storageClassName: "manual"
        accessModes: [ReadWriteOnce]
        volumeMode: Filesystem
        size: 200Mi
        annotations: {}
        labels: {}
        nodeAffinity: {}            # pin PV to a node if needed
```

* `baseDir` is the directory inside the pod where telemetry files appear;
  when `source: hostPath`, the host path **on the node** is the same `baseDir`.
* `volume.source` chooses between:
  
    - `pvc` — the chart **creates a PVC** (dynamic PV provisioning
      **must be configured** prior).
    - `existing` — the chart **uses an existing PVC** you name in `existingClaim`.
    - `hostPath` *(default)* — the chart **creates a static hostPath PV +
      matching PVC** mounted at `baseDir`.
      Default PV settings include `persistentVolumeReclaimPolicy: Retain`
      and `storageClassName: manual`; optional `nodeAffinity` ties
      the PV to a specific node.

> HINT: **Recommendation**
>
> Prefer dynamically-provisioned PVCs (`source: pvc`) or existing PVCs
> (`source: existing`) over `hostPath`.
> HostPath ties data to one node and is generally less stable/portable
> for controller pods that may reschedule.

## How the chart wires this up

* **Only** in `local` mode, an init container (`telemetry-data-permissions`)
  runs as root (only the `CHOWN` capability)
  to `chown -R 65532:65532 <baseDir>` so the main controller
  (running as non-root) can write files.
  The main pod runs with `runAsNonRoot: true`.
* When `source: hostPath`, the chart **creates**:

    - a **PersistentVolume** with `hostPath.path: <baseDir>`,
      `type: DirectoryOrCreate`, reclaim policy **Retain** (*default*),
      optional `nodeAffinity`.
    - a **matching PVC** bound by name.

> NOTE: **Note on a legacy flag**
>
> `controller.enableTelemetry` is **deprecated in favor of the `telemetry`
> block**. It may still appear as a container arg for backward compatibility,
> but you should configure telemetry via the `telemetry` values going forward.

## Configuration examples

> NOTE:
>
> Replace only the snippets you need; all other values keep their defaults.

### Disable telemetry entirely

```yaml
telemetry:
  mode: disabled
```

No data is collected or stored.

### Online telemetry (defaults)

```yaml
telemetry:
  mode: online      # default
  concurrency: 5    # default
  interval: 24h     # default
  jitter: 10        # default
```

Sends data to Segment.io CDP.

### Local telemetry with dynamic provisioning

> WARNING: Ensure dynamic PV provisioning is configured prior.

```yaml
telemetry:
  mode: local
  local:
    baseDir: /var/lib/telemetry
    volume:
      source: pvc
      pvc:
        storageClassName: gp2
        accessModes: [ReadWriteOnce]
        size: 5Gi
```

Chart creates a PVC; files appear under `/var/lib/telemetry`
in the pod, rotated daily.

### Local telemetry with an existing PVC

```yaml
telemetry:
  mode: local
  local:
    baseDir: /var/lib/telemetry
    volume:
      source: existing
      existingClaim: kcm-telemetry-pvc
```

Chart uses your PVC; no PV/PVC objects are created by the chart.

### Local telemetry with hostPath (single-node / pinned-node)

```yaml
telemetry:
  mode: local
  local:
    baseDir: /var/lib/telemetry
    volume:
      source: hostPath
      hostPath:
        name: kcm-telemetry-hostpath
        reclaimPolicy: Retain
        storageClassName: manual
        size: 10Gi
        nodeAffinity:
          required:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values: ["worker-01"]
```

Chart creates a hostPath PV at `/var/lib/telemetry` and a matching PVC;
prefer PVC-backed options in HA clusters.

## Troubleshooting

* **No local files appear**:
  ensure `telemetry.mode: local` and that a volume source is configured.
  Check the init container `telemetry-data-permissions` completed successfully.
* **Pod unschedulable with hostPath**:
  if the pod moves to a different node, the hostPath PV won’t be there;
  either pin the PV with `nodeAffinity` or switch to a PVC-backed option.
* **Conflicting settings**:
  if you previously set `controller.enableTelemetry`, migrate to the
  `telemetry` block; the old flag is deprecated and may be removed in a
  future release.
