# Data Lake Setup — MinIO + AWS CLI + NiFi

This `docker-compose.yml` sets up the infrastructure for a data lake built on S3-compatible object storage (MinIO), along with tools for managing data
(AWS CLI) )and orchestrating data flows (Apache NiFi)=.

## Services

- **minio** — S3-compatible object storage server.
  - Console: `http://localhost:9001`
  - API endpoint: `http://localhost:9000`
  - Credentials: `minioadmin` / `minioadmin`

- **aws-cli** — client container used to manage MinIO buckets and objects via standard AWS CLI commands (create bucket, upload, list, metadata).

## Medallion Architecture

The data lake follows a 3-tier medallion architecture, implemented as
separate buckets:
- `bronze` — raw data, unchanged from the source 
- `silver` — transformed data 
- `gold` — aggregated, business-ready data

The buckets are created manually through the `aws-cli` container before running the Kafka Connect S3 sink connector.

---

## Prerequisite: Creating the bucket structure

Before running the S3 sink connector, the buckets used by the data lake must exist in MinIO.

### Step 1 — Start the containers (if not already running)

```bash
docker compose up -d
```

### Step 2 — Enter the `aws-cli` container

Either through a terminal:

```bash
docker exec -it dataLakeMinio-aws-cli-1 bash
```

> The exact container name may vary depending on how Compose generates it.
> Check the exact name with:
> ```bash
> docker ps
> ```

Or through Docker Desktop:

```
Docker Desktop → Containers → dataLakeMinio → aws-cli → Terminal tab
```

### Step 3 — Create the buckets

In line with the medallion architecture, create a separate bucket for each zone — `gold`, `silver`, and `bronze`:

```bash
aws --endpoint-url http://minio:9000 s3 mb s3://gold
aws --endpoint-url http://minio:9000 s3 mb s3://silver
aws --endpoint-url http://minio:9000 s3 mb s3://bronze
```

### Step 4 — Verify

```bash
aws --endpoint-url http://minio:9000 s3 ls
```

---

## Notes on the Kafka Connect S3 sink connector

The Debezium → S3 connector used to populate the bronze zone is configured to write into the `bronze` bucket, consistent with the medallion architecture pattern described above.
