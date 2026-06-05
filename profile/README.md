<p align="center">
  <img src="/profile/lattice_logo.png" alt="Lattice Logo" width="500" />
</p>
# Lattice

Lattice is an open-source framework designed for fast and consistent data propagation in your CDC pipelines. Built to handle real-time data, you can have infrastructure running in a matter of minutes.

For more information, check out our case study: [Lattice Platform Case Study](https://latticeplatform.io)

---

## Architecture

Lattice runs on a cluster of ECS instances in AWS and is composed of the following components:

- **Apache Kafka** — High-throughput distributed event streaming.
- **Kafka Connect with Debezium** — Change data capture from your source databases.
- **Apicurio Schema Registry** — Manages and stores schemas for Kafka messages, ensuring efficient data serialization and schema evolution.
- **Lattice UI** — Web interface for managing and monitoring your pipelines.
- **OpenTofu Automation with CLI Wrapper** — Infrastructure-as-code deployment tooling.

---

## Getting Started

### 1. Configure Your Source Databases

#### PostgreSQL

**Local instance:** Ensure that replication and the write-ahead log are accessible. In your `postgresql.conf` file, add or update the following settings:

```
wal_level = logical
max_replication_slots = 5
max_wal_senders = 5
```

Then, in your `pg_hba.conf` file, allow replication connections:

```
host replication all 0.0.0.0/0 md5
```

**RDS instance:** Enable logical replication via the RDS console by setting the following parameter:

```
name         = "rds.logical_replication"
value        = "1"
apply_method = "pending-reboot"
```

A reboot of your instance is required after making these changes.

**Permissions:** Inside `psql`, grant the appropriate permissions to your database user. The example below grants access to a user named `inventory` on the `inventory` database:

```sql
GRANT CONNECT ON DATABASE inventory TO inventory;
GRANT USAGE ON SCHEMA public TO inventory;
GRANT rds_replication TO inventory;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO inventory;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO inventory;
```

#### MongoDB

Run MongoDB with the `--replSet` flag and call `rs.initiate()` to initialize the replica set.

---

### 2. Configure Sink Connectors

Sink connectors write data to its destination after it travels through the Kafka pipeline. Clickhouse is our primary tested and fully supported sink, though any JDBC-compatible connector can be used.

#### ClickHouse

Tables in ClickHouse must follow a specific naming convention to map correctly from Kafka topics. For example, to sync the `products` table from PostgreSQL, create a table named `public.products` in ClickHouse. The Kafka topic will be mapped to the matching table name.

#### JDBC Connectors

Other JDBC-compatible databases are supported. Refer to the JDBC documentation for your specific database to confirm setup requirements and replication support.

---

### 3. Set Up AWS Credentials

Ensure your AWS credentials have the permissions necessary to create ECS instances. You can use an admin account or a role with at minimum the following permissions:

```
iam:PassRole
iam:PutUserPolicy
iam:AttachUserPolicy
iam:PutRolePolicy
```

Contact your AWS administrator if you need these permissions granted.

Once permissions are in place:

1. Install and configure the [AWS CLI](https://aws.amazon.com/cli/).
2. Verify it's working by running:
   ```
   aws sts get-caller-identity
   ```

---

### 4. Install OpenTofu

Follow the [official OpenTofu installation guide](https://opentofu.org/docs/intro/install/). After installation, verify it's working:

```
tofu --help
```

---

### 5. Deploy with `lattice_deploy`

Once the AWS CLI is configured and OpenTofu is installed, run `npx lattice-platform-cli setup` to provision your infrastructure.
To teardown its as simple as `npx lattice-platform-cli teardown`

---

### 6. Access the Lattice UI

After deployment, locate the UI through the AWS console:

1. Navigate to your ECS cluster.
2. Find the service named `lattice-ui`.
3. Under **Tasks**, click the specific Task ID for `lattice-ui`.
4. Go to the **Network** tab to find the service's IP address.
5. Open that IP address in your browser to access the Lattice UI.

Alternatively, if you have access to your private VPC, navigate to `lattice.local` in your browser.

Full UI documentation is available in the [how-to section of our case study](https://latticeplatform.io/docs/lattice-ui/).

---

## Manual Deployment (for Testing)

Instructions for local/manual deployment can be found in the [lattice-local repository](https://github.com/latticeplatform/lattice-local).

---

## Enjoy 🥳

---

## The Team

[Joshua Crane](https://github.com/joshbintree) |
[Kessler Mentele](https://github.com/KesslerMentele) |
[William Seipp](https://github.com/williamseipp)