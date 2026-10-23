Silo Distributed Storage Lab

A hands-on distributed object-storage lab built with Silo, a community-maintained MinIO fork by the Pigsty project.

The objective of this project was to build a distributed Silo cluster from scratch, validate S3-compatible storage operations, test failure and healing behaviour, monitor the cluster with Prometheus, run a small benchmark, and prove that the environment can be rebuilt from the repository.

Project Objectives

This lab was designed to verify that a distributed Silo deployment can:

Store and retrieve objects correctly
Provide S3-compatible operations
Maintain data availability during node failures
Recover from drive failure and complete healing
Preserve object integrity through checksum verification
Support versioning and object locking
Expose operational metrics for monitoring
Provide indicative performance measurements
Be destroyed and rebuilt from the repository
Architecture

The lab uses Docker Compose with four Silo nodes behind an Nginx load balancer.

                    Client
                      |
                      v
              +---------------+
              | Nginx / LB    |
              |   :9000       |
              +-------+-------+
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
      +-------+   +-------+   +-------+   +-------+
      | Silo1 |   | Silo2 |   | Silo3 |   | Silo4 |
      +-------+   +-------+   +-------+   +-------+
          |           |           |           |
          +-----------+-----------+-----------+
                      |
               Distributed Pool
                      |
              Docker Network
Components
Component	Purpose
Silo × 4	Distributed object-storage nodes
Nginx	Load balancer / single client entry point
Prometheus	Metrics collection and monitoring
Docker Compose	Local orchestration
mcli	Silo administration and S3 operations
boto3	S3 SDK validation
Warp	Indicative performance benchmark
Environment
Item	Configuration
Deployment	Local Docker Compose
Operating System	Ubuntu Linux
Container Runtime	Docker
Orchestration	Docker Compose v2
Silo Nodes	4
Load Balancer	Nginx
Monitoring	Prometheus
Storage	Docker/local lab storage
Data	Synthetic test data
Silo Image	docker.io/pgsty/silo:RELEASE.2026-09-03T13-18-01Z
Image Digest	Recorded during the lab
Network	Shared Docker Compose network

The Silo image was pinned to an immutable release tag rather than using latest.

Phase Results
Phase 01 — Cluster Build

Status: PASS

The distributed Silo cluster was created using Docker Compose.

Completed:

Docker Compose configuration
Environment configuration
Four Silo nodes
Shared Docker network
Nginx load balancer
Health checks
Pinned Silo image
Cluster startup validation
Administrative cluster information
Console access

All Silo nodes successfully joined the distributed pool.

Phase 02 — Health Checks

Status: PASS

The cluster health mechanisms were validated.

Tested:

Native Silo health check
S3-compatible liveness endpoint
Cluster health endpoint
Node availability

The cluster reported a healthy state before continuing with failure testing.

Phase 03 — Basic S3 Operations

Status: PASS

Basic object-storage functionality was successfully validated.

Tested operations included:

Bucket creation
Object upload
Object listing
Object download
Object deletion
Multipart upload
Presigned URL
Custom metadata
Object tags
Python boto3 access
Integrity Validation

Object integrity was verified by comparing checksums before upload and after download.

The Silo checksum verification functionality was also executed:

mcli checksum verify

No unexplained checksum differences were identified.

Phase 04 — Versioning and Object Lock

Status: PASS

S3 data-protection features were tested.

Versioning

Validated:

Versioning enablement
Object overwrite
Multiple object versions
Identification of previous versions
Restoration of an older version
Object Lock

Validated:

Object-lock-enabled bucket
Retention configuration
Attempted deletion of a protected object

The protected object could not be deleted before its configured retention period expired.

Phase 05 — Failure and Healing

Status: PASS

The distributed cluster was tested under failure conditions.

Single Node Failure

One Silo node was stopped.

Observed:

Cluster remained operational
S3 operations continued according to the cluster's available quorum
Remaining nodes continued serving requests
Two Node Failure

Two nodes were stopped and cluster behaviour was recorded.

The results demonstrated the practical availability limits of the selected distributed topology.

Drive Failure

A drive's data directory was removed and the affected node was restarted.

The cluster detected the missing data and initiated the healing/recovery process.

Healing was monitored until completion.

Integrity After Recovery

After recovery, the objects were checked again using checksums.

Result:

Zero unexplained checksum differences were observed after recovery.

This confirmed that the failure/recovery test did not introduce unexplained data corruption.

Phase 06 — Monitoring

Status: PASS

A local Prometheus instance was configured to scrape Silo's Prometheus-compatible minio_* metrics.

The monitoring setup provided visibility into:

Storage capacity
Used/free capacity
Nodes
Drives
Request activity
Request errors
Healing activity
Recommended Alerts
Alert Area	Reason
Node unavailable	Detect node failure
Drive unavailable	Detect storage failure
Low free capacity	Prevent storage exhaustion
High error rate	Detect service/client problems
Healing activity/failure	Detect recovery problems
Cluster health	Detect overall availability issues
Request latency	Detect performance degradation

Monitoring provides the operational visibility required to identify storage and availability problems before they become larger incidents.

Phase 07 — Benchmark

Status: PASS

MinIO Warp was used for a small performance benchmark.

The benchmark covered:

Small-object PUT
Small-object GET
Large-object PUT
Large-object GET

Multiple runs were performed for each workload.

Machine resource usage was observed during the benchmark.

Benchmark Interpretation

The benchmark results are indicative and lab-scale.

They should not be treated as production performance numbers because the environment uses a local Docker-based topology rather than multiple physical storage servers.

The results are primarily useful for:

Comparing future configuration changes
Understanding workload behaviour
Establishing a baseline
Practising performance measurement
Phase 08 — Rebuild From Scratch

Status: PASS

The lab was validated as a reproducible environment.

The cluster can be destroyed and recreated from the repository using Docker Compose.

Example:

docker compose up -d

The rebuild process and timing were recorded as part of the final validation.

This confirms that the environment is not dependent on undocumented manual configuration.

Overall Results
Phase	Area	Result
01	Cluster Build	PASS
02	Health Checks	PASS
03	S3 Operations & Integrity	PASS
04	Versioning & Object Lock	PASS
05	Failure & Healing	PASS
06	Monitoring	PASS
07	Benchmark	PASS
08	Rebuild	PASS
Overall Status

8 / 8 phases completed successfully.

Key Findings
1. Distributed storage works as expected

The four-node Silo cluster successfully operated as a distributed object-storage system behind a load balancer.

2. S3 compatibility was validated

Both CLI-based operations and Python boto3 access were successfully tested.

3. Data integrity was explicitly verified

Checksums were compared before and after storage operations, and Silo's checksum verification capability was used.

4. Failure recovery was validated

Node and drive failure scenarios were tested, including the healing process.

5. Monitoring is available

Silo's Prometheus-compatible metrics can be collected using a local Prometheus deployment.

6. The environment is reproducible

The cluster can be rebuilt from the repository using Docker Compose rather than relying on undocumented manual steps.

7. Lab-scale benchmarking is useful as a baseline

Warp provides a repeatable way to compare performance across future configuration changes.

Known Limitations

This project intentionally represents a local lab environment, not a production deployment.

Limitations include:

Single physical/local machine
Docker-based storage
No multiple physical failure domains
No production TLS/CA setup
No Kubernetes or Nomad orchestration
No production-scale performance testing
Synthetic test data only

Therefore, the observed availability and performance characteristics should not be directly interpreted as production capacity or SLA guarantees.

Security and Repository Hygiene

The repository is intended to be public.

Therefore:

No real credentials are stored
Secrets are represented using placeholders
Access keys and secret keys are excluded
Internal hostnames/IPs are not committed
Screenshots are reviewed before committing
Secret scanning is used to reduce the risk of credential exposure
Reproduce the Lab

Clone the repository and start the environment:

git clone <REPOSITORY_URL>
cd silo-cluster-lab
docker compose up -d

Check the running services:

docker compose ps

View logs:

docker compose logs -f

When finished with the lab:

docker compose down

Warning: docker compose down -v removes Docker volumes and therefore deletes stored lab data. Use it only when intentionally destroying the environment.

Recommendation

The tested Silo topology is sound for a controlled distributed-storage lab and development environment.

The project successfully demonstrated the core capabilities required by this task: distributed storage, S3 operations, integrity verification, failure handling, healing, monitoring, benchmarking, and reproducibility.

For production adoption, additional validation would be required across multiple physical hosts, independent failure domains, production networking, TLS, backup/recovery procedures, capacity planning, security hardening, and larger-scale performance testing.

Recommendation to the DevOps team: the lab provides a solid technical foundation for further Silo evaluation and can be used as a reproducible environment for future work such as migration testing, feature comparison, and storage-backend integration.
