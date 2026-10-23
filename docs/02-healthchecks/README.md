# 02 — Silo Health Checks

## Goal

Verify the health and availability of the distributed Silo cluster using native Silo health checks and MinIO-compatible HTTP health endpoints.

The purpose of this phase is to confirm that the cluster is operational before proceeding with S3 operations, failure testing, and monitoring.

The following checks are covered:

* Native Silo health check
* Liveness endpoint
* Cluster readiness/health endpoint
* HTTP response validation
* Evidence collection for health-check results

---

## Environment

| Item              | Details                                             |
| ----------------- | --------------------------------------------------- |
| Storage System    | Silo                                                |
| Deployment        | Docker Compose                                      |
| Operating System  | Ubuntu Linux                                        |
| Container Runtime | Docker                                              |
| Orchestration     | Docker Compose v2                                   |
| Cluster Nodes     | 4                                                   |
| Load Balancer     | Nginx                                               |
| Silo Image        | `docker.io/pgsty/silo:RELEASE.2026-09-03T13-18-01Z` |
| Client            | `mcli` and HTTP client                              |
| Test Data         | Synthetic lab data                                  |

The health checks were performed against the deployed Silo lab.

---

## Steps

### Step 1 — Check Container Status

Run the following command from the project directory:

```bash
docker compose ps
```

**Expected Result**

All four Silo nodes and the Nginx load balancer should be running. Silo containers with configured health checks should report a healthy status.

**Actual Result**

Record the observed container states in the evidence file.

**Result:** PASS / FAIL — confirm using the saved command output.

---

### Step 2 — Run the Native Silo Health Check

Run the native health-check command supported by Silo:

```bash
docker compose exec silo1 silo healthcheck
```

**Expected Result**

The command should complete successfully when the checked Silo service is healthy.

**Actual Result**

Record the command output and exit status. If the command reports an error, preserve the output for troubleshooting.

**Result:** PASS / FAIL — confirm using the saved command output.

---

### Step 3 — Check the Liveness Endpoint

The MinIO-compatible liveness endpoint is:

```text
/minio/health/live
```

Send a request through the configured Silo endpoint:

```bash
curl -i http://127.0.0.1:9000/minio/health/live
```

**Expected Result**

The endpoint should return HTTP `200 OK` when the Silo process is alive and responding to health checks.

**Actual Result**

Record the HTTP status code and response.

**Result:** PASS / FAIL — confirm using the saved command output.

---

### Step 4 — Check the Cluster Health Endpoint

The MinIO-compatible cluster health endpoint is:

```text
/minio/health/cluster
```

Run:

```bash
curl -i http://127.0.0.1:9000/minio/health/cluster
```

**Expected Result**

When the distributed cluster satisfies the endpoint's health requirements, it should return HTTP `200 OK`.

A non-success response should be investigated rather than treated as a healthy result.

**Actual Result**

Record the HTTP status code and response.

**Result:** PASS / FAIL — confirm using the saved command output.

---

### Step 5 — Check the Readiness Endpoint

The readiness endpoint can provide additional information about whether the cluster is ready to serve requests.

Run:

```bash
curl -i http://127.0.0.1:9000/minio/health/ready
```

**Expected Result**

If the endpoint is supported by the deployed Silo release and the cluster is ready, it should return a successful HTTP response.

**Actual Result**

Record the response. If the endpoint is not supported by this release, document the returned status instead of marking the test as passed.

**Result:** PASS / FAIL / NOT SUPPORTED — confirm using the saved command output.

---

## Results Summary

Complete this table using the actual saved outputs.

| Check                   | Expected Result                  | Actual Result         | Status               |
| ----------------------- | -------------------------------- | --------------------- | -------------------- |
| Container status        | Required services running        | Record observed state | Pending verification |
| Native health check     | Successful health check          | Record command output | Pending verification |
| Liveness endpoint       | HTTP 200                         | Record HTTP status    | Pending verification |
| Cluster health endpoint | HTTP 200 when healthy            | Record HTTP status    | Pending verification |
| Readiness endpoint      | Successful response if supported | Record HTTP status    | Pending verification |

> **Important:** Do not mark a test as PASS until its actual command output has been reviewed.

---

## Findings

The health-checking phase provides a baseline for evaluating the Silo cluster before conducting storage operations and failure tests.

The checks serve different purposes:

* **Container status:** Shows whether the Docker services are running.
* **Native health check:** Uses Silo's built-in health-check functionality.
* **Liveness endpoint:** Helps determine whether the Silo service is responding.
* **Cluster health endpoint:** Checks cluster health according to the endpoint's semantics.
* **Readiness endpoint:** Provides an additional readiness signal when supported.

A running container does not, by itself, prove that the distributed storage cluster is healthy. The results should be evaluated together.

---

## How to Reproduce

From the Silo project directory, run:

```bash
docker compose ps
```

Run the native health check:

```bash
docker compose exec silo1 silo healthcheck
```

Check the HTTP endpoints:

```bash
curl -i http://127.0.0.1:9000/minio/health/live
curl -i http://127.0.0.1:9000/minio/health/cluster
curl -i http://127.0.0.1:9000/minio/health/ready
```

These commands assume that the local endpoint is published on `127.0.0.1:9000`. Adjust the endpoint if the Compose configuration uses a different address or port.

---

## Evidence

Store raw command output in:

```text
docs/02-healthchecks/evidence/
```

Suggested evidence files:

```text
docker-compose-ps.txt
silo-healthcheck.txt
health-live.txt
health-cluster.txt
health-ready.txt
```

Capture both the HTTP status and response body where applicable.

Screenshots may be stored in:

```text
docs/02-healthchecks/screenshots/
```

Raw command output is the primary evidence. Screenshots provide supplementary visual confirmation.

---

## Cleanup

Health checks do not require deleting cluster data.

To stop the lab while preserving its volumes:

```bash
docker compose down
```

To stop the lab and remove its volumes:

```bash
docker compose down -v
```

> **Warning:** The `-v` option removes Docker volumes and can permanently delete the stored lab data. Do not use it unless the data is no longer needed.

