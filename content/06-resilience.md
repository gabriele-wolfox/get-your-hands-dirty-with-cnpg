# Slot 6: Resilience

> **Goal:** Test CNPG high-availability by triggering an unplanned failover
> and a planned switchover.

## Identify the current primary

```bash
kubectl get cluster cluster-example-backup
```

You can also check via labels:

```bash
kubectl get pods -l cnpg.io/cluster=cluster-example-backup -L role
```

## Failover: delete the primary pod

In a **separate terminal**, watch the pods:

```bash
kubectl get pods -w
```

Delete the primary pod to simulate an unplanned failure:

```bash
kubectl delete pod cluster-example-backup-<N>
```

> **Note:** Replace `<N>` with the primary pod number.

Watch the cluster react:

1. CNPG detects the primary is gone
2. The most advanced replica is promoted to primary
3. The deleted pod is recreated as a new replica

Once all pods are `Running` and `1/1`, check the new topology:

```bash
kubectl cnpg status cluster-example-backup
```

The primary has changed. Verify your data is still there:

```bash
kubectl cnpg psql cluster-example-backup -- app -c "SELECT COUNT(*) FROM numbers;"
```

## Incident simulation: delete pod and PVC

This simulates a more severe failure where both the pod and its storage are
lost.

Identify the current primary:

```bash
kubectl get cluster cluster-example-backup
```

In a **separate terminal**, watch the pods:

```bash
kubectl get pods -w
```

Delete both the pod and its PVC:

```bash
kubectl delete pod,pvc cluster-example-backup-<N>
```

Watch the recovery, CNPG will create a new pod, provision new storage, and
rebuild the instance from the remaining cluster members.

Check the cluster status:

```bash
kubectl cnpg status cluster-example-backup
```

## Switchover: planned promotion

A **switchover** is a graceful, zero-data-loss promotion. Use the cnpg
plugin:

```bash
kubectl cnpg promote cluster-example-backup <replica-pod-name>
```

> **Note:** Replace `<replica-pod-name>` with one of the current replicas
> (check with `kubectl cnpg status cluster-example-backup`).

Watch the switchover:

```bash
kubectl get pods -l cnpg.io/cluster=cluster-example-backup -L role -w
```

Verify:

```bash
kubectl cnpg status cluster-example-backup
```

The roles have swapped with minimal disruption.

## Recap

You have:

- Triggered an **unplanned failover** by deleting the primary pod
- Simulated a **severe failure** by deleting both pod and PVC
- Performed a **planned switchover** using `kubectl cnpg promote`
