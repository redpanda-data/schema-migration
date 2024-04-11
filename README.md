# Schema Migration

This repo contains two sub-projects related to schema migration: REST Exporter and Connect Transform. Since they each work in different ways, they offer unique advantages and disadvantages: use whichever suits your circumstances.

---

# Projects

## [- Rest Exporter](./batch)

This Python project provides a batch import/export capability of subjects. There are two tools available:

- **exporter**: this reads schemas from a Schema Registry REST endpoint and writes them to a local file
- **importer**: this reads schemas from a local file and writes them to a `_schemas` topic on a target Redpanda cluster

## [- Connect Transform](./streaming)

This Java project contains a Kafka Connect Single Message Transform (SMT) that can enables the use of MirrorMaker 2 for continuous, real-time migration of schemas between clusters.

---

# How to choose?

- If you only have visibility of the REST endpoint for Schema Registry, and not the underlying `_schemas` topic, the REST exporter is the only available choice. 

- If you have visibility of the `_schemas` topic on the source cluster, then either the REST exporter or MirrorMaker 2 (with the transform) can be considered.

The use of MirrorMaker 2 means a more complex deployment, but the advantages of being both realtime and continuous are obvious.

# Caveats

Both approaches provide a unidirectional replication capability. If the target cluster is usable, there is a risk that new subjects / schemas are created, resulting in a schema ID clash.

This should be avoided while replication is ongoing.