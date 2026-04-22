# Get Your Hands Dirty with CloudNativePG

A hands-on workshop on [CloudNativePG](https://cloudnative-pg.io/).

CloudNativePG is the Kubernetes operator that covers the full lifecycle of a
highly available PostgreSQL database cluster with a primary/standby
architecture, using native streaming replication.

After a brief presentation (~30 min), attendees follow guided exercises on
their own laptop using
[cnpg-playground](https://github.com/cloudnative-pg/cnpg-playground).

## Agenda

The workshop is organized in 10 slots. Slots 1 through 4 should be followed
in order. Slots 5 through 9 are independent and can be done in any order,
depending on your interests. Some steps involve waiting for images to pull
and pods to start, so the overall pace could depend on your environment and
network. Don't worry if you don't finish everything during the session!

| # | Slot | Description | Soundtrack |
|:-:|:-----|:------------|:-----------|
| 1 | [**Introduction to CNPG**](slides/introduction.pdf) | Presentation: the operator CRD, operand images and extensibility, suggested architectures, deployment options, monitoring, and recent features (declarative databases, major PG version upgrade, quorum failover, …) | *Welcome to the Jungle* , Guns N' Roses |
| 2 | [**Deploy and inspect the operator**](content/02-deploy-and-inspect-the-operator.md) | Set up the environment, install the CNPG operator, explore the infrastructure | *Thunderstruck* , AC/DC |
| 3 | [**Apply and inspect the Cluster CRD**](content/03-apply-and-inspect-the-cluster-crd.md) | Create PostgreSQL clusters (simple + with backup) and examine the provisioned resources | *Start Me Up* , The Rolling Stones |
| 4 | [**Work with data**](content/04-work-with-data.md) | Connect to PostgreSQL, insert data, create a backup, and recover a new cluster from it | *Should I Stay or Should I Go* , The Clash |
| 5 | [**Observability**](content/05-observability.md) | Inspect Prometheus metrics and cluster status | *Every Breath You Take* , The Police |
| 6 | [**Resilience**](content/06-resilience.md) | Failover (delete pod/PVC) and planned switchover | *Stayin' Alive* , Bee Gees |
| 7 | [**Explore the cnpg plugin**](content/07-plugin-commands.md) | Logs, reports, hibernate, and other plugin commands | *Plug In Baby* , Muse |
| 8 | [**Declarative logical replication**](content/08-logical-replication.md) | Database, Publication, and Subscription CRDs for logical replication | *The Logical Song* , Supertramp |
| 9 | [**Declarative extensions**](content/09-extensions.md) | Customize PostgreSQL with declarative extensions via the Database CRD | *Another Brick in the Wall* , Pink Floyd |
| 10 | [**Cleanup**](content/10-cleanup.md) | Tear down the playground and wrap up | *The End* , The Doors |

## Prerequisites

Please follow the setup instructions in the
[cnpg-playground](https://github.com/cloudnative-pg/cnpg-playground)
repository **before** the workshop so you can hit the ground running from
the very first exercise.
