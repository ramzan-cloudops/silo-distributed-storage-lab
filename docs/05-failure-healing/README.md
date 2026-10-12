# Phase 05 — Failure Testing and Healing

## 1. Objective

The objective of this phase was to test how the four-node Silo distributed object storage cluster responds to node failures and recovers after the failed nodes are restarted.

The tests focused on cluster health, object availability, and recovery.

## 2. Lab Environment

* **Storage system:** Silo
* **Deployment:** Docker Compose
* **Silo nodes:** `silo1`, `silo2`, `silo3`, `silo4`
* **Reverse proxy:** Nginx
* **S3 API endpoint:** `http://127.0.0.1:9000`
* **Test bucket:** `ramzan-lab-bucket`
* **Test object:** `hello.txt`

Expected object content:

`Silo distributed storage lab - Version 2`

## 3. Test 1 — Baseline Cluster Health

**Action:** Checked cluster health before failure testing.

**Command:**

```bash
curl -i http://127.0.0.1:9000/minio/health/cluster
```

**Observed result:** HTTP 200 OK.

**Status:** PASS

The cluster health endpoint reported that the cluster was healthy before failure testing.

## 4. Test 2 — Single-Node Failure

**Action:** Stopped `silo4` to simulate a single-node failure.

**Command:**

```bash
docker compose stop silo4
```

**Verification commands:**

```bash
curl -i http://127.0.0.1:9000/minio/health/cluster
docker compose exec silo1 mcli ls lab/ramzan-lab-bucket/
docker compose exec silo1 mcli cat lab/ramzan-lab-bucket/hello.txt
```

**Observed results:**

* The cluster health endpoint returned HTTP 200 OK.
* The `hello.txt` object remained accessible.
* The object content matched the expected Version 2 content.

**Status:** PASS

The cluster remained available during the tested single-node outage.

## 5. Test 3 — Single-Node Recovery

**Action:** Restarted `silo4` after the single-node failure test.

**Command:**

```bash
docker compose start silo4
```

**Verification commands:**

```bash
docker compose ps
curl -i http://127.0.0.1:9000/minio/health/cluster
docker compose exec silo1 mcli cat lab/ramzan-lab-bucket/hello.txt
```

**Observed results:**

* The cluster health endpoint returned HTTP 200 OK.
* The object content remained correct after recovery.
* The node returned to service.

**Status:** PASS

## 6. Test 4 — Two-Node Failure

**Action:** Stopped `silo3` and `silo4` to test cluster health with two nodes unavailable.

**Command:**

```bash
docker compose stop silo3 silo4
```

**Verification command:**

```bash
curl -i http://127.0.0.1:9000/minio/health/cluster
```

**Observed results:**

* The cluster health endpoint returned HTTP 503 Service Unavailable.
* The response included `X-Minio-Write-Quorum: 3`.

**Status:** PASS — Expected failure behavior observed.

The cluster reported an unhealthy state when two of its four Silo nodes were stopped. Object read and write operations were not tested during this two-node outage.

## 7. Test 5 — Two-Node Recovery

**Action:** Restarted both stopped nodes.

**Command:**

```bash
docker compose start silo3 silo4
```

**Verification commands:**

```bash
docker compose ps
curl -i http://127.0.0.1:9000/minio/health/cluster
docker compose exec silo1 mcli cat lab/ramzan-lab-bucket/hello.txt
```

**Observed results:**

* All four Silo containers returned to a healthy state.
* The Nginx container was running.
* The cluster health endpoint returned HTTP 200 OK.
* The test object was readable and its content matched the expected value.

**Status:** PASS

The cluster recovered after both stopped nodes were restarted.

## 8. Test Summary

| Test                 | Expected behavior                | Observed result                      | Status |
| -------------------- | -------------------------------- | ------------------------------------ | ------ |
| Baseline health      | Healthy cluster                  | HTTP 200 OK                          | PASS   |
| Single-node failure  | Cluster remains available        | HTTP 200 OK; object readable         | PASS   |
| Single-node recovery | Cluster returns to service       | HTTP 200 OK; object readable         | PASS   |
| Two-node failure     | Cluster reports unhealthy state  | HTTP 503; write quorum reported as 3 | PASS   |
| Two-node recovery    | Cluster returns to healthy state | HTTP 200 OK; object readable         | PASS   |

## 9. Important Limitations

This lab runs all four Silo nodes as Docker containers on a single EC2 instance.

Therefore:

* The tests demonstrate container/node failure behavior within this lab.
* The nodes do not represent independent physical servers.
* Separate container data directories do not necessarily represent separate physical disks.
* These tests do not prove resilience against an EC2 instance failure, an Availability Zone failure, or a physical storage-device failure.
* The two-node outage test verified cluster health behavior; object read/write operations were not tested during that outage.

A production resilience assessment would require testing across independent hosts and storage devices.

## 10. Conclusion

The failure and recovery tests demonstrated that the Silo cluster remained healthy during the tested single-node outage, reported an unhealthy state during the tested two-node outage, and returned to a healthy state after the stopped nodes were restarted.

The test object remained accessible after recovery.

Phase 05 documents the observed failure and recovery behavior of this local distributed storage lab.
