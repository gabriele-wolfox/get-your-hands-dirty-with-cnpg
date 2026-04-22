# Slot 9: Declarative Extensions

> **Goal:** Use the Database CRD to declaratively enable PostgreSQL extensions
> and explore extended functionality.

## Extensions in CNPG

The CNPG operand images ship with many popular PostgreSQL extensions
pre-built. Instead of running `CREATE EXTENSION` manually, the `Database`
CRD lets you declare which extensions a database needs, the operator
enables them for you.

## Create a database with pgvector

`pgvector` adds vector data types and similarity search operators to
PostgreSQL, useful for AI/ML workloads and semantic search.

Review [`manifests/vectordb.yaml`](../manifests/vectordb.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Database
metadata:
  name: vectordb
spec:
  name: vectordb
  owner: app
  cluster:
    name: cluster-example
  extensions:
    - name: vector
```

Apply it:

```bash
kubectl apply -f manifests/vectordb.yaml
```

Check the database and extension were created:

```bash
kubectl get databases
```

```bash
kubectl cnpg psql cluster-example -- -d vectordb \
  -c "SELECT extname, extversion FROM pg_extension;"
```

You should see the `vector` extension listed alongside the default ones.

## Use pgvector

Create a table with a vector column:

```bash
kubectl cnpg psql cluster-example -- -d vectordb -c "
  CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    embedding vector(3)
  );
  INSERT INTO documents (title, embedding) VALUES
    ('PostgreSQL Intro',   '[0.1, 0.8, 0.3]'),
    ('Kubernetes Basics',  '[0.9, 0.2, 0.1]'),
    ('CNPG Overview',      '[0.5, 0.5, 0.5]'),
    ('SQL Deep Dive',      '[0.2, 0.9, 0.2]'),
    ('Container Security', '[0.8, 0.1, 0.3]');
"
```

Perform a similarity search:

```bash
kubectl cnpg psql cluster-example -- -d vectordb -c "
  SELECT title, embedding <-> '[0.1, 0.9, 0.2]' AS distance
  FROM documents
  ORDER BY distance
  LIMIT 3;
"
```

## Create a database with multiple extensions

The `Database` CRD supports enabling multiple extensions at once. Create a
database with `pg_stat_statements` and `btree_gist`:

Review [`manifests/analytics-db.yaml`](../manifests/analytics-db.yaml):

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Database
metadata:
  name: analytics-db
spec:
  name: analytics
  owner: app
  cluster:
    name: cluster-example
  extensions:
    - name: pg_stat_statements
    - name: btree_gist
```

Apply it:

```bash
kubectl apply -f manifests/analytics-db.yaml
```

Verify the extensions:

```bash
kubectl cnpg psql cluster-example -- -d analytics \
  -c "SELECT extname, extversion FROM pg_extension ORDER BY extname;"
```

## Explore available extensions

List all extensions available in the operand image:

```bash
kubectl cnpg psql cluster-example -- -d analytics \
  -c "SELECT name, default_version, comment FROM pg_available_extensions ORDER BY name;"
```

This shows everything you can declaratively enable through the `Database`
CRD, the list depends on what is compiled into the operand image.

## Recap

You have:

- Enabled **pgvector** declaratively through the Database CRD
- Performed **vector similarity searches** in PostgreSQL
- Created a database with **multiple extensions** in a single manifest
- Listed all **available extensions** in the operand image
