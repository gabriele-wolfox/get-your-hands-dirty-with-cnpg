# Slot 3: Apply and Inspect the Cluster CRD

> **Goal:** Create PostgreSQL clusters using the Cluster CRD and explore the
> resources that CNPG provisions automatically.

## Watch resources (separate terminal)

Open a **separate terminal** and watch pods as they are created:

```bash
kubectl get pods -w
```

## Deploy a simple cluster

Review the manifest in [`manifests/cluster-example.yaml`](../manifests/cluster-example.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example
spec:
  instances: 3

  storage:
    size: 1Gi
```

Key points:

- **`instances: 3`**: one primary and two streaming replicas.
- **`storage.size`**: each instance gets its own 1 Gi PVC.

Apply it:

```bash
kubectl apply -f manifests/cluster-example.yaml
```

Pods need to pull the PostgreSQL operand image before they can start. This
could take a few minutes depending on your network. While you wait, use the
time to explore what CNPG is doing behind the scenes.

### While the cluster is coming up

Check the cluster phase as pods progress through their lifecycle:

```bash
kubectl get cluster -o wide
```

You will see the phase change from `Setting up primary` to
`Creating a new replica` and finally `Cluster in healthy state`.

Inspect the full cluster spec to see the defaults CNPG adds on top of your
minimal manifest:

```bash
kubectl get cluster cluster-example -o yaml | less
```

Notice how much the operator filled in. Look for:

- **`spec.postgresql`**: default PostgreSQL configuration parameters
- **`spec.replicationSlots`**: HA replication slots enabled by default
- **`spec.enableSuperuserAccess`**: `false` by default, the `postgres`
  superuser is locked down
- **`spec.enablePDB`**: `true` by default, a PodDisruptionBudget protects
  the primary
- **`spec.switchoverDelay`**: defaults to `3600` seconds (1 hour)
- **`spec.stopDelay`**: defaults to `1800` seconds (30 minutes)
- **`spec.primaryUpdateStrategy`**: defaults to `unsupervised`

Now look at the status section:

```bash
kubectl get cluster cluster-example -o yaml | grep -A 50 "^status:"
```

Interesting fields:

- **`status.conditions`**: standard Kubernetes conditions (Ready, etc.)
- **`status.phase`**: human-readable cluster lifecycle phase
- **`status.currentPrimary`** / **`status.targetPrimary`**: which pod is
  primary and which is the intended primary (they differ during switchover)
- **`status.certificates`**: TLS certificate details including DNS
  alternative names used for server verification
- **`status.topology`**: how instances are distributed across nodes
- **`status.timelineID`**: the PostgreSQL WAL timeline, increments after
  each promotion

Follow the operator logs to see what the controller is doing in real time:

```bash
kubectl logs -n cnpg-system deployment/cnpg-controller-manager -f
```

Once all three pods are `Running` and `1/1` ready, move on.

## Analyze resources

Once the cluster is ready, look at everything CNPG created:

```bash
kubectl get clusters,pods,pvc,svc,ep,secrets
```

Take note of:

- The **Cluster** object and its status
- Three **pods** (one primary, two replicas)
- Three **PVCs**, one per instance, data is not shared
- **Services**: `cluster-example-rw` (primary), `cluster-example-ro`
  (replicas), `cluster-example-r` (any instance)
- **Secrets** with superuser and app credentials, plus TLS certificates

## Deploy a cluster with backup

Now create a cluster with continuous backup to object storage using the
**Barman Cloud Plugin**.

First, create an `ObjectStore` resource that points to the RustFS instance
provided by the playground.

Review [`manifests/objectstore-eu.yaml`](../manifests/objectstore-eu.yaml):

```yaml
apiVersion: barmancloud.cnpg.io/v1
kind: ObjectStore
metadata:
  name: objectstore-eu
spec:
  configuration:
    destinationPath: s3://backups/
    endpointURL: http://objectstore-eu:9000
    s3Credentials:
      accessKeyId:
        name: objectstore-eu
        key: ACCESS_KEY_ID
      secretAccessKey:
        name: objectstore-eu
        key: ACCESS_SECRET_KEY
    wal:
      compression: gzip
```

Apply it:

```bash
kubectl apply -f manifests/objectstore-eu.yaml
```

Now review [`manifests/cluster-example-backup.yaml`](../manifests/cluster-example-backup.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example-backup
spec:
  instances: 3
  storage:
    size: 1Gi

  plugins:
  - name: barman-cloud.cloudnative-pg.io
    isWALArchiver: true
    parameters:
      barmanObjectName: objectstore-eu
      serverName: cluster-example-backup

  externalClusters:
  - name: cluster-example-backup
    plugin:
      name: barman-cloud.cloudnative-pg.io
      parameters:
        barmanObjectName: objectstore-eu
        serverName: cluster-example-backup
```

Key points:

- **`plugins`**: enables the Barman Cloud Plugin for WAL archiving.
- **`externalClusters`**: configures the plugin as the source for recovery.

Apply it:

```bash
kubectl apply -f manifests/cluster-example-backup.yaml
```

Check the cluster status:

```bash
kubectl cnpg status cluster-example-backup
```

## Recap

You have:

- Created a simple 3-instance PostgreSQL cluster
- Explored the resources CNPG provisions (pods, PVCs, services, secrets)
- Deployed a second cluster with continuous backup to object storage
