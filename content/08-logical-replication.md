# Slot 8: Declarative Logical Replication

> **Goal:** Set up logical replication between two clusters using the
> declarative Database, Publication, and Subscription CRDs.

In this exercise we will replicate data from `cluster-example-backup`
(source) to `cluster-example` (target) using PostgreSQL logical replication
, all managed declaratively through Kubernetes CRDs.

## Create the source database

Use the `Database` CRD to create a `workshop` database on the source
cluster:

Review [`manifests/workshop-db-source.yaml`](../manifests/workshop-db-source.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Database
metadata:
  name: workshop-db-source
spec:
  name: workshop
  owner: app
  cluster:
    name: cluster-example-backup
```

Apply it:

```bash
kubectl apply -f manifests/workshop-db-source.yaml
```

Check the database was created:

```bash
kubectl get databases
```

Verify inside PostgreSQL:

```bash
kubectl cnpg psql cluster-example-backup -- -c "\l"
```

## Populate the source database

Create a table and insert data:

```bash
kubectl cnpg psql cluster-example-backup -- -d workshop -c "
  CREATE TABLE cities (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    country TEXT NOT NULL
  );
  INSERT INTO cities (name, country) VALUES
    ('Rome', 'Italy'),
    ('Berlin', 'Germany'),
    ('Madrid', 'Spain'),
    ('Lisbon', 'Portugal'),
    ('Paris', 'France');
"
```

## Prepare the source for logical replication

The `app` user needs `REPLICATION` privilege and `SELECT` access on the
published tables:

```bash
kubectl cnpg psql cluster-example-backup -- -d workshop -c "
  ALTER ROLE app WITH REPLICATION;
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO app;
"
```

## Create a Publication

A `Publication` defines which tables are available for logical replication.
Create one that publishes all tables in the `workshop` database:

Review [`manifests/pub-cities.yaml`](../manifests/pub-cities.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Publication
metadata:
  name: pub-cities
spec:
  name: pub_cities
  dbname: workshop
  cluster:
    name: cluster-example-backup
  target:
    allTables: true
```

Apply it:

```bash
kubectl apply -f manifests/pub-cities.yaml
```

Verify the publication was created:

```bash
kubectl get publications
```

```bash
kubectl cnpg psql cluster-example-backup -- -d workshop \
  -c "SELECT * FROM pg_publication;"
```

## Prepare the target cluster

The target cluster needs to know how to connect to the source. Add an
`externalClusters` entry to `cluster-example`:

```bash
kubectl patch cluster cluster-example --type merge -p '
{
  "spec": {
    "externalClusters": [
      {
        "name": "cluster-example-backup",
        "connectionParameters": {
          "host": "cluster-example-backup-rw",
          "user": "app",
          "dbname": "workshop"
        },
        "password": {
          "name": "cluster-example-backup-app",
          "key": "password"
        }
      }
    ]
  }
}'
```

## Create the target database

Create the same `workshop` database on the target cluster:

Review [`manifests/workshop-db-target.yaml`](../manifests/workshop-db-target.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Database
metadata:
  name: workshop-db-target
spec:
  name: workshop
  owner: app
  cluster:
    name: cluster-example
```

Apply it:

```bash
kubectl apply -f manifests/workshop-db-target.yaml
```

Logical replication requires the schema to exist on the subscriber. Create
the target table:

```bash
kubectl cnpg psql cluster-example -- -d workshop -c "
  CREATE TABLE cities (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    country TEXT NOT NULL
  );
"
```

## Create a Subscription

A `Subscription` connects to a publication and starts replicating data.
Create one on the target cluster:

Review [`manifests/sub-cities.yaml`](../manifests/sub-cities.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Subscription
metadata:
  name: sub-cities
spec:
  name: sub_cities
  dbname: workshop
  cluster:
    name: cluster-example
  publicationName: pub_cities
  externalClusterName: cluster-example-backup
```

Apply it:

```bash
kubectl apply -f manifests/sub-cities.yaml
```

Verify the subscription:

```bash
kubectl get subscriptions
```

## Verify replication

Check that data has been replicated to the target cluster:

```bash
kubectl cnpg psql cluster-example -- -d workshop \
  -c "SELECT * FROM cities;"
```

You should see the five rows from the source cluster.

Now insert more data on the source and verify it arrives on the target:

```bash
kubectl cnpg psql cluster-example-backup -- -d workshop -c "
  INSERT INTO cities (name, country) VALUES ('Prague', 'Czech Republic');
"
```

```bash
kubectl cnpg psql cluster-example -- -d workshop \
  -c "SELECT * FROM cities;"
```

The new row appears on the target, logical replication is working, all
managed declaratively through Kubernetes CRDs.

## Recap

You have:

- Created databases using the **Database CRD** on two clusters
- Configured **externalClusters** on the target for cross-cluster connectivity
- Set up a **Publication** on the source cluster
- Created a **Subscription** on the target cluster
- Verified **logical replication** works end-to-end
