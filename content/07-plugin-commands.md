# Slot 7: Explore the cnpg Plugin

> **Goal:** Discover the most useful commands provided by the `kubectl cnpg`
> plugin.

## Overview

List all available plugin commands:

```bash
kubectl cnpg --help
```

## Cluster status

You have already used `status` throughout the workshop. Let's look at the
full output:

```bash
kubectl cnpg status cluster-example-backup --verbose
```

The verbose flag shows additional details including instance manager
versions, disk usage, and configuration.

## Cluster logs

Check the logs from a specific pod:

```bash
kubectl logs cluster-example-backup-1
```

The cnpg plugin can aggregate and pretty-print logs from the entire cluster:

```bash
kubectl cnpg logs cluster cluster-example-backup \
  | kubectl cnpg logs pretty
```

This is especially useful during incident investigations, as it collects
logs from all instances and formats them for readability.

## Diagnostic report

Generate a comprehensive diagnostic report:

```bash
kubectl cnpg report cluster cluster-example-backup
```

This collects:

- Cluster and pod manifests
- Events
- Logs from all instances
- Operator logs
- PVC and secret metadata

The output can be saved and shared for support or troubleshooting.

## Hibernate and resume

CNPG can **hibernate** a cluster, scaling it to zero pods while preserving
PVCs. This saves compute resources when a cluster is not in use:

```bash
kubectl cnpg hibernate on cluster-example
```

Check that all pods are gone:

```bash
kubectl get pods -l cnpg.io/cluster=cluster-example
```

The PVCs are still there:

```bash
kubectl get pvc -l cnpg.io/cluster=cluster-example
```

Resume the cluster:

```bash
kubectl cnpg hibernate off cluster-example
```

Watch it come back up:

```bash
kubectl get pods -l cnpg.io/cluster=cluster-example -w
```

## Plugin install and upgrade

You can check the installed operator version:

```bash
kubectl cnpg version
```

And preview what an operator install or upgrade would apply, without
actually applying it:

```bash
kubectl cnpg install generate --control-plane | head -50
```

## Summary of key commands

| Command | Description |
|:--------|:------------|
| `kubectl cnpg status` | Cluster health, replication, backups |
| `kubectl cnpg promote` | Trigger a planned switchover |
| `kubectl cnpg backup` | Create an on-demand backup |
| `kubectl cnpg psql` | Open a psql session |
| `kubectl cnpg logs cluster` | Aggregate cluster logs |
| `kubectl cnpg logs pretty` | Pretty-print JSON logs |
| `kubectl cnpg report cluster` | Generate a diagnostic report |
| `kubectl cnpg hibernate on/off` | Hibernate or resume a cluster |
| `kubectl cnpg install generate` | Generate operator install manifests |
| `kubectl cnpg version` | Show plugin and operator version |

## Recap

You have:

- Explored the full `kubectl cnpg` plugin command set
- Used `logs` and `report` for observability and troubleshooting
- **Hibernated** and **resumed** a cluster to save compute resources
