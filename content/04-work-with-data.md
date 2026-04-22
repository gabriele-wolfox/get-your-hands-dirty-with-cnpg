# Slot 4: Work with Data

> **Goal:** Connect to PostgreSQL, create a table, insert data, create a
> backup, and recover a new cluster from it.

## Connect to the database

Use the cnpg plugin to open a `psql` session on the `app` database of the
backup-enabled cluster:

```bash
kubectl cnpg psql cluster-example-backup -- app
```

## Create a table and insert data

Run the following SQL:

```sql
CREATE TABLE numbers(x int);
INSERT INTO numbers (SELECT generate_series(1,10000));
\q
```

## Create a backup

Trigger an on-demand backup using the Barman Cloud Plugin:

```bash
kubectl cnpg backup cluster-example-backup \
  --method plugin \
  --plugin-name barman-cloud.cloudnative-pg.io
```

## Verify the backup

Check the backup object:

```bash
kubectl get backup
```

For more details:

```bash
kubectl get backup -o yaml
```

Verify the cluster status reflects the backup:

```bash
kubectl cnpg status cluster-example-backup
```

You should see the backup listed and WAL archiving active.

## Recover a new cluster from the backup

Before starting the recovery, ensure all WAL files are archived by forcing
a WAL switch:

```bash
kubectl cnpg psql cluster-example-backup -- -c "SELECT pg_switch_wal();"
```

Create a new cluster that bootstraps from the backup you just took.

Review [`manifests/cluster-recovery.yaml`](../manifests/cluster-recovery.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-recovery
spec:
  instances: 3
  storage:
    size: 1Gi

  bootstrap:
    recovery:
      source: cluster-example-backup

  externalClusters:
  - name: cluster-example-backup
    plugin:
      name: barman-cloud.cloudnative-pg.io
      parameters:
        barmanObjectName: objectstore-eu
        serverName: cluster-example-backup
```

Key points:

- **`bootstrap.recovery`**: tells CNPG to initialize this cluster from a
  backup rather than from scratch.
- **`externalClusters`**: uses the Barman Cloud Plugin to locate the backup
  and WAL files in the object store.

In a **separate terminal**, watch the pods:

```bash
kubectl get pods -w
```

Apply the manifest:

```bash
kubectl apply -f manifests/cluster-recovery.yaml
```

Wait for the cluster to become ready:

```bash
kubectl cnpg status cluster-recovery
```

Verify the data is there:

```bash
kubectl cnpg psql cluster-recovery -- app
```

```sql
SELECT COUNT(*) FROM numbers;
\q
```

You should see 10,000 rows, the data was fully recovered from the backup.

## Recap

You have:

- Connected to PostgreSQL using `kubectl cnpg psql`
- Created a table and inserted data
- Triggered an on-demand backup via the Barman Cloud Plugin
- Verified the backup completed successfully
- Recovered a brand-new cluster from the object store backup
