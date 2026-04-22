# Slot 5: Observability

> **Goal:** Explore CNPG Prometheus metrics and cluster status.

## Inspect metrics

CNPG exposes a Prometheus-compatible `/metrics` endpoint on each PostgreSQL
pod (port 9187). Let's look at the raw metrics.

Find the primary pod:

```bash
kubectl get cluster cluster-example-backup
```

Port-forward the metrics endpoint:

```bash
kubectl port-forward cluster-example-backup-1 9187:9187 &
```

> **Note:** Replace `cluster-example-backup-1` with the actual primary pod
> name.

Fetch the metrics with curl:

```bash
curl -s localhost:9187/metrics | head -80
```

You will see Prometheus-format metrics covering connections, replication,
WAL, transactions, and more.

Filter for specific metrics:

```bash
curl -s localhost:9187/metrics | grep cnpg_pg_replication
curl -s localhost:9187/metrics | grep cnpg_pg_stat_archiver
```

Stop the port-forward:

```bash
kill %1
```

## Inspect cluster status

The cnpg plugin provides rich status information:

```bash
kubectl cnpg status cluster-example-backup
```

This shows the primary, replicas, timeline, replication lag, backup status,
and more.

For even more detail:

```bash
kubectl cnpg status cluster-example-backup --verbose
```

## Recap

You have:

- Inspected raw Prometheus metrics from a PostgreSQL pod via port-forward
- Used `kubectl cnpg status` for a high-level cluster health overview
