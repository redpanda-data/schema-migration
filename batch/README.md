# Schema Migration Tool

This tool helps migrate schemas from one Schema Registry instance to another. It exports schemas from a source registry by reading them through the REST API.
It imports schemas into a target instance by writing the schemas as messages to the \_schemas topic. This process ensures we have direct control of the version numbers and schema IDs.

# Dependencies

This tool has the following dependencies:

### Linux
- Python 3.9: `apt install python3`

### Python Libraries
- fastavro: `pip3 install fastavro`
- requests: `pip3 install requests`
- pyyaml: `pip3 install pyyaml`
- confluent_kafka: `pip3 install confluent_kafka`

# Schema Export

`exporter.py` connects to the source Schema Registry instance and uses the REST API to extract all schemas and subjects.
These are written out to an intermediate file, allowing the schemas to be stored in version control, transferred
remotely, etc.

###  Usage:
```shell
usage: python3 exporter.py --config conf/config.yaml
```

### Configuration

`exporter.py` uses YAML for configuration - examples are available in the `conf/` directory.

An example config is below:

```yaml
exporter:
  source:
    url: https://localhost:8081/ # Mandatory
    ssl.key.location: client.key
    ssl.certificate.location: client.cert
    ssl.ca.location: ca.cert
  options:
    exclude.deleted.versions: false # Mandatory
    exclude.deleted.subjects: false # Mandatory
    logfile: export.log # Mandatory

schemas: exported-schemas.txt # Mandatory
```

Under `source:`, the URL of the schema registry is mandatory. The SSL parameters can be removed when connecting to an insecure Schema Registry
instance. If you utilize basic auth, you can add `basic.auth.user.info` under source with the user and password values. i.e. `basic.auth.user.info: "<user>:<password>"`

By default, `exporter.py` will extract all subjects and versions, *including* those which have been *soft-deleted*.
Soft-deleted items can be excluded by setting the appropriate option.

**Hard-deleted schema versions and subjects are permanently removed by Schema Registry. The migration tool can not
retrieve these.**

The `schemas:` configuration defines the local file where the exported schemas will be stored. This configuration is also used by 
`importer.py`.

# Schema Import

`importer.py` reads messages from the intermediate file and writes them to the specified schemas topic on the target Redpanda
cluster. These messages are then read by Schema Registry and made available via REST.

### Usage

```shell
usage: python3 importer.py --config conf/config.yaml
```

### Configuration

`importer.py` uses YAML for configuration - examples are available in the `conf/` directory.

An example config is below:

```yaml
schemas: exported-schemas.txt # Mandatory

importer:
  target:
    bootstrap.servers: localhost:9092 # Mandatory
    security.protocol: SASL_SSL
    sasl.mechanism: SCRAM-SHA-256
    sasl.username: username
    sasl.password: password
  options:
    topic: _schemas # Mandatory
    logfile: import.log # Mandatory
```

As with `exporter.py`, the input file (containing schemas to import) is specified using the `schemas:` property and is
mandatory.

The Kafka producer configuration uses a minimum of `bootstrap_servers` in order to connect. For secure Redpanda
clusters, the remaining security properties are also necessary.

#### Producer Configuration

The importer writes using a Producer


# Considerations

## Idempotency

From limited testing, it appears that Redpanda Schema Registry will accept duplicate messages without issue, so running the migration
import tool multiple times should work.

## Migration Process

Once a Schema Registry instance begins becomes active and begins to assign new schema IDs for itself, further migrations would be inadvisable.

If further export / import processes are performed, there is a risk that the import would write a schema with an ID that
has already been used on the target registry. If there is an ID clash, existing message data may become unreadable.

You have been warned!