# Silo Distributed Object Storage Lab

**A reproducible distributed object-storage lab demonstrating S3-compatible operations, failure recovery, data integrity, monitoring, and performance testing.**

## Overview

This project implements and validates a distributed object-storage cluster using [Silo](https://github.com/pgsty/silo), a community-maintained MinIO fork maintained by the Pigsty project.

The environment was deployed using Docker Compose and consists of four Silo nodes behind an Nginx load balancer. Prometheus was used to collect storage and operational metrics, while `mcli`, Python's `boto3` SDK, and MinIO Warp supported functional validation and performance testing.

The primary goal was to build a distributed storage environment, deliberately test its behaviour under failure conditions, verify object integrity, and demonstrate that the lab could be rebuilt from its repository configuration.

**Project status: 8 of 8 validation phases completed.**

## Key Highlights

* Deployed a four-node Silo cluster using Docker Compose.
* Configured Nginx as a single entry point for client requests.
* Validated S3-compatible bucket and object operations using `mcli` and Python `boto3`.
* Tested object versioning, restoration of previous versions, and Object Lock retention.
* Simulated node and drive failures and observed recovery and healing behaviour.
* Verified object integrity using checksums before and after relevant operations.
* Configured Prometheus to collect Silo's Prometheus-compatible metrics.
* Benchmarked small- and large-object workloads using MinIO Warp.
* Documented the process for rebuilding the environment from the repository.

## Architecture

The lab uses Docker Compose to orchestrate four Silo nodes, with Nginx providing a single client-facing endpoint.

```text
                      S3 Client
                          |
                          v
                 +------------------+
                 | Nginx Load       |
                 | Balancer :9000   |
                 +--------+---------+
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
          +--------+  +--------+  +--------+  +--------+
          | Silo 1 |  | Silo 2 |  | Silo 3 |  | Silo 4 |
          +--------+  +--------+  +--------+  +--------+
              |           |           |           |
              +-----------+-----------+-----------+
                          |
                 Distributed Storage
                          |
                   Docker Network

             Prometheus --> Silo Metrics
```

### Components

| Component         | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| Silo × 4          | Distributed object-storage nodes                    |
| Nginx             | Single client-facing endpoint and load balancer     |
| Docker Compose v2 | Container orchestration and reproducible deployment |
| Prometheus        | Collection and inspection of operational metrics    |
| `mcli`            | Silo administration and S3-compatible operations    |
| Python `boto3`    | Programmatic S3 API validation                      |
| MinIO Warp        | Object-storage performance benchmarking             |

## Environment

| Item               | Configuration                                       |
| ------------------ | --------------------------------------------------- |
| Deployment         | Docker Compose lab                                  |
| Operating system   | Ubuntu Linux                                        |
| Container runtime  | Docker                                              |
| Orchestration      | Docker Compose v2                                   |
| Storage nodes      | 4                                                   |
| Client entry point | Nginx                                               |
| Monitoring         | Prometheus                                          |
| Storage            | Docker-managed local lab storage                    |
| Test data          | Synthetic data                                      |
| Silo image         | `docker.io/pgsty/silo:RELEASE.2026-09-03T13-18-01Z` |
| Image digest       | Recorded during the lab                             |
| Network            | Shared Docker Compose network                       |

The Silo image uses a pinned release tag rather than the floating `latest` tag to make the lab configuration more predictable. For stronger supply-chain reproducibility, the exact image digest can also be pinned in the Compose configuration.

## Implementation and Validation

The project was completed through eight validation phases.

### Phase 01 — Cluster Deployment

**Result: PASS**

Built the initial distributed storage environment using Docker Compose.

Implemented and validated:

* Four Silo service definitions.
* Shared Docker networking.
* Nginx load-balancer configuration.
* Environment-based configuration.
* Container health checks.
* Pinned Silo image.
* Cluster startup and administrative status checks.

**Outcome:** All four Silo nodes successfully joined the distributed storage pool.

### Phase 02 — Health and Availability Checks

**Result: PASS**

Validated the cluster's health and availability mechanisms before running failure tests.

Checks included:

* Native Silo health checks.
* S3-compatible liveness endpoint.
* Cluster health endpoint.
* Node availability.
* Administrative cluster status.

**Outcome:** The cluster reported a healthy state before failure testing began.

### Phase 03 — S3 Operations and Data Integrity

**Result: PASS**

Validated common object-storage operations through the S3-compatible interface.

Operations included:

* Bucket creation and listing.
* Object upload, listing, download, and deletion.
* Multipart uploads.
* Presigned URL generation and access.
* Custom object metadata.
* Object tags.
* Python `boto3` integration.

#### Integrity validation

Compared checksums before upload and after download to verify that the retrieved test objects matched their original contents.

Also executed Silo's checksum verification command:

```bash
mcli checksum verify
```

**Outcome:** No unexplained checksum differences were identified during the recorded tests.

### Phase 04 — Versioning and Object Lock

**Result: PASS**

Tested object-version management and retention controls.

**Versioning tests:**

* Enabled bucket versioning.
* Overwrote an existing object.
* Inspected multiple object versions.
* Identified previous versions.
* Restored an older object version.

**Object Lock tests:**

* Created an Object-Lock-enabled bucket.
* Configured retention.
* Attempted to delete an object protected by retention.

**Outcome:** The protected object could not be deleted before its configured retention period expired in the tested scenario.

### Phase 05 — Failure, Recovery, and Healing

**Result: PASS**

Simulated storage-node and drive failures to observe the cluster's operational behaviour.

#### Single-node failure

Stopped one Silo node and checked whether the remaining cluster continued serving requests.

Observed that the cluster remained operational and S3 operations continued under the tested conditions.

#### Two-node failure

Stopped two Silo nodes and recorded the resulting behaviour to understand the availability limits of the selected topology.

The results were used to evaluate how node availability affected cluster operations.

#### Drive failure and recovery

Removed a drive's data directory and restarted the affected node to simulate missing storage data.

Monitored the resulting recovery and healing process.

#### Post-recovery integrity

Repeated relevant object-integrity checks after recovery.

**Outcome:** No unexplained checksum differences were observed in the recorded post-recovery checks.

These tests demonstrate the behaviour observed in this lab topology; they do not establish a production availability guarantee.

### Phase 06 — Prometheus Monitoring

**Result: PASS**

Configured Prometheus to scrape Silo's Prometheus-compatible `minio_*` metrics.

Validated the monitoring configuration and checked that the configured scrape targets were available.

The monitoring setup provided visibility into relevant operational areas, including:

* Storage capacity and usage.
* Node and drive status.
* Request activity and errors.
* Healing activity.
* Operational health indicators.

#### Recommended alerting areas

| Alert area                  | Operational purpose                        |
| --------------------------- | ------------------------------------------ |
| Node unavailable            | Detect storage-node failures               |
| Drive unavailable           | Identify storage-device problems           |
| Low free capacity           | Identify potential storage exhaustion      |
| High request-error rate     | Detect client or service problems          |
| Healing activity or failure | Monitor recovery behaviour                 |
| Cluster health              | Track overall storage availability         |
| Request latency             | Identify potential performance degradation |

These are recommended alerting areas, not a claim that every alert rule was deployed and tested.

### Phase 07 — Performance Benchmarking

**Result: PASS**

Used MinIO Warp to run small-scale object-storage benchmarks.

Workloads included:

* Small-object PUT.
* Small-object GET.
* Large-object PUT.
* Large-object GET.

Multiple runs were performed for each workload, and machine resource usage was observed during testing.

#### Benchmark interpretation

The measurements provide a baseline for comparing future configuration changes and workload behaviour.

They should not be interpreted as production performance figures because the lab runs on a local Docker-based topology rather than multiple independent physical storage servers.

Detailed benchmark outputs and supporting evidence are maintained in the project repository where available.

### Phase 08 — Reproducibility and Rebuild Validation

**Result: PASS**

Validated the process for recreating the lab using the repository's Docker Compose configuration.

The environment was rebuilt and its startup behaviour was checked as part of the final validation.

The basic deployment command is:

```bash
docker compose up -d
```

Inspect service status:

```bash
docker compose ps
```

Inspect service logs:

```bash
docker compose logs -f
```

Stop the environment:

```bash
docker compose down
```

**Warning:** Use `docker compose down -v` only when intentionally deleting the associated Docker volumes and stored lab data.

**Outcome:** The lab's rebuild process was validated using the repository configuration, reducing reliance on undocumented manual setup steps.

## Overall Results

| Phase | Area                           | Result |
| ----- | ------------------------------ | ------ |
| 01    | Cluster deployment             | PASS   |
| 02    | Health and availability checks | PASS   |
| 03    | S3 operations and integrity    | PASS   |
| 04    | Versioning and Object Lock     | PASS   |
| 05    | Failure, recovery, and healing | PASS   |
| 06    | Prometheus monitoring          | PASS   |
| 07    | Performance benchmarking       | PASS   |
| 08    | Rebuild validation             | PASS   |

**Overall: 8/8 phases completed successfully in the lab environment.**

## Key Technical Takeaways

This project provided hands-on experience with several operational aspects of distributed object storage:

1. **Distributed deployment:** Configured and operated multiple storage nodes behind a shared client-facing endpoint.
2. **S3-compatible access:** Validated object-storage operations through both a CLI and a Python SDK.
3. **Data integrity:** Used checksums to detect unexplained differences during upload, download, and recovery validation.
4. **Failure analysis:** Observed how node and drive failures affected service availability and recovery.
5. **Monitoring:** Integrated Prometheus metrics collection to improve operational visibility.
6. **Performance testing:** Established a repeatable approach for measuring object-storage workloads.
7. **Reproducibility:** Validated deployment reconstruction from repository configuration.

## Security and Repository Hygiene

The repository is intended for public access. The project follows these documentation and repository practices:

* Use placeholders instead of real credentials in committed examples.
* Exclude environment files containing secrets.
* Avoid committing access keys, secret keys, or passwords.
* Review screenshots and logs before publishing them.
* Run secret scanning to check for accidental credential exposure.

Gitleaks was run against the repository and scanned 14 commits. The reported result was **`no leaks found`**.

Secret scanning reduces the risk of accidental exposure but does not guarantee that every possible secret or security issue has been identified.

## Limitations and Scope

This project is a controlled, single-machine lab rather than a production deployment.

Known limitations include:

* A single physical or virtual host for the storage topology.
* Local Docker-managed storage rather than independent physical storage systems.
* No independent physical failure domains.
* No production TLS and certificate-authority configuration.
* No Kubernetes or Nomad orchestration.
* No production-scale performance evaluation.
* Synthetic test data rather than real workloads.
* No production availability or durability SLA established by these tests.

Consequently, the observed failure behaviour and benchmark results should not be treated as production guarantees.

## Future Improvements

Potential next steps include:

* Deploying storage nodes across independent hosts and failure domains.
* Implementing production-grade TLS and certificate management.
* Adding and testing Prometheus alert rules.
* Establishing automated backup and restore procedures.
* Performing larger-scale performance and endurance testing.
* Evaluating orchestration through Kubernetes or Nomad.
* Expanding automated recovery and integrity checks.

## Recommendation

The lab provides a reproducible foundation for evaluating Silo's distributed object-storage capabilities in a controlled development environment.

It demonstrates practical work across container orchestration, S3-compatible interfaces, object integrity, failure testing, recovery monitoring, benchmarking, and infrastructure reproducibility.

The results support continued technical evaluation and further experimentation. Production adoption would require additional testing across independent hosts, security hardening, backup and recovery validation, capacity planning, and production-scale performance assessment.

## Repository

The source configuration, phase documentation, and available test evidence are maintained in this repository.

To reproduce the lab, clone the repository and follow the deployment instructions and configuration files provided in the project.

---

**Project:** Silo Distributed Object Storage Lab
**Technology areas:** Docker Compose · Distributed Storage · S3 API · Nginx · Prometheus · Python `boto3` · MinIO Warp · Gitleaks
