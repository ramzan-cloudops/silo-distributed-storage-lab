# 01 — Cluster Build

## Goal

Build a distributed Silo cluster from scratch using Docker Compose.

The objective of this phase is to deploy four Silo nodes with a load balancer, verify that the nodes form a distributed cluster, and confirm that the cluster is accessible and healthy.

---

## Environment

| Item              | Details                                             |
| ----------------- | --------------------------------------------------- |
| Deployment        | Local Docker Compose                                |
| Operating System  | Ubuntu Linux                                        |
| Container Runtime | Docker                                              |
| Orchestration     | Docker Compose v2                                   |
| Silo Nodes        | 4                                                   |
| Load Balancer     | Nginx                                               |
| Silo Image        | `docker.io/pgsty/silo:RELEASE.2026-09-03T13-18-01Z` |
| Storage           | Docker/local lab storage                            |
| Network           | Shared Docker Compose network                       |
| Data              | Synthetic test data                                 |

### Cluster Topology

```text
                    Client
                      |
                      v
                +-----------+
                |   Nginx   |
                | Load Balancer
                +-----+-----+
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
    +-------+     +-------+     +-------+
    | Silo1 |     | Silo2 |     | Silo3 |
    +-------+     +-------+     +-------+
         \             |             /
          \            |            /
           +-----------+-----------+
                       |
                   +-------+
                   | Silo4 |
                   +-------+

              Distributed Silo Pool
```

---

## Steps

### Step 1 — Validate the Compose Configuration

Run:

```bash
docker compose config
```

**Expected result:**
Docker Compose parses the configuration successfully without errors.

**Actual result:**
The Compose configuration was validated successfully.

**Result:** PASS

---

### Step 2 — Start the Silo Cluster

Run:

```bash
docker compose up -d
```

Then check the services:

```bash
docker compose ps
```

**Expected result:**
Four Silo nodes and the Nginx load balancer should start successfully.

**Actual result:**
The Silo cluster and Nginx load balancer started successfully.

**Result:** PASS

---

### Step 3 — Verify Container Health

Run:

```bash
docker compose ps
```

Verify that the following services are running:

```text
silo1
silo2
silo3
silo4
nginx
```

**Expected result:**
All required services should be running/healthy.

**Actual result:**
All four Silo nodes and Nginx were running successfully.

**Result:** PASS

---

### Step 4 — Verify Silo Cluster Formation

Check the Silo logs:

```bash
docker compose logs --tail=100 silo1
```

Repeat for the remaining nodes if required:

```bash
docker compose logs --tail=100 silo2
docker compose logs --tail=100 silo3
docker compose logs --tail=100 silo4
```

**Expected result:**
The Silo nodes should discover their peers and form the distributed storage pool.

**Actual result:**
The Silo nodes successfully formed the distributed cluster.

**Result:** PASS

---

### Step 5 — Verify Cluster Administration

Run:

```bash
mcli admin info
```

**Expected result:**
The administrative output should show the distributed Silo nodes and their storage drives as online/available.

**Actual result:**
The cluster status was verified using `mcli admin info`.

**Result:** PASS

---

### Step 6 — Verify the Load Balancer

Check Nginx:

```bash
docker compose ps nginx
```

Validate its configuration:

```bash
docker compose exec nginx nginx -t
```

**Expected result:**
Nginx should be running and its configuration should pass the syntax check.

**Actual result:**
Nginx was running and the configuration passed validation.

**Result:** PASS

---

### Step 7 — Verify Console Access

Access the Silo Console through the configured local endpoint.

**Expected result:**
The Silo Console should be reachable and allow administrative access.

**Actual result:**
The Silo Console was successfully accessed.

**Result:** PASS

---

## Result Summary

| Test                       | Result |
| -------------------------- | ------ |
| Compose configuration      | PASS   |
| Cluster startup            | PASS   |
| Four Silo nodes running    | PASS   |
| Distributed pool formation | PASS   |
| `mcli admin info`          | PASS   |
| Nginx load balancer        | PASS   |
| Silo Console access        | PASS   |

### Overall Result

**PASS**

The four-node distributed Silo cluster was successfully deployed using Docker Compose and made accessible through the Nginx load balancer.

---

## Findings

* Four Silo nodes successfully operated as a distributed cluster.
* All nodes used the same pinned Silo release.
* Nginx successfully provided the client-facing entry point.
* `mcli admin info` was used to verify the cluster state.
* The Silo Console was accessible after the cluster was deployed.
* Docker Compose provided a reproducible way to create the local Silo environment.

---

## Reproduce

From the project directory:

```bash
docker compose up -d
```

Verify the deployment:

```bash
docker compose ps
```

Check the cluster:

```bash
mcli admin info
```

View logs when troubleshooting:

```bash
docker compose logs --tail=100
```

---

## Cleanup

Stop the cluster while preserving its volumes:

```bash
docker compose down
```

To completely destroy the lab and its stored data:

```bash
docker compose down -v
```

> **Warning:** `docker compose down -v` deletes the Docker volumes and all data stored in them. Use it only when intentionally destroying the lab.

---

## Evidence

Raw command outputs for this phase are stored under:

```text
docs/01-cluster/evidence/
```

Screenshots, where applicable, are stored under:

```text
docs/01-cluster/screenshots/
```

Raw command output is treated as the primary evidence, while screenshots provide additional visual confirmation.

