# Phase 04: Object Versioning and Object Lock

## Overview

This phase validates object versioning, configures Object Lock retention, and tests protection against deleting a retained object version in the Silo distributed object storage lab.

## Objectives

* Enable and verify bucket versioning.
* Upload multiple versions of the same object.
* Create a bucket with Object Lock enabled.
* Configure Governance retention.
* Verify retention on a specific object version.
* Test whether direct deletion of a protected version is rejected.

## Environment

* **Storage platform:** Silo
* **Deployment:** Docker Compose
* **Host:** AWS EC2 Ubuntu
* **Storage endpoint:** `http://127.0.0.1:9000`
* **Versioning test bucket:** `ramzan-lab-bucket`
* **Object Lock test bucket:** `ramzan-lock-bucket`

> **Lab limitation:** All storage containers run on a single EC2 instance. This phase does not demonstrate independent host-level or disk-level fault tolerance.

## 1. Verify Object Versioning

The `ramzan-lab-bucket` bucket was configured with versioning enabled.

The object `hello.txt` was uploaded twice with different content. The version listing showed two versions:

| Version |     Size | Description      |
| ------- | -------: | ---------------- |
| v1      | 34 bytes | Original content |
| v2      | 41 bytes | Updated content  |

**Result:** PASS — Multiple versions of the same object were visible.

## 2. Create an Object Lock Bucket

The original bucket did not support Object Lock. A separate bucket was created with both Object Lock and versioning enabled.

```bash
docker compose exec silo1 mcli mb --with-lock --with-versioning lab/ramzan-lock-bucket
```

Versioning was then verified:

```bash
docker compose exec silo1 mcli version info lab/ramzan-lock-bucket
```

**Result:** PASS — The bucket was created successfully, and versioning was enabled.

## 3. Configure Governance Retention

A test object named `object-lock-test.txt` was uploaded to the Object Lock bucket.

Governance retention was configured for one day:

```bash
docker compose exec silo1 mcli retention set governance 1d lab/ramzan-lock-bucket/object-lock-test.txt
```

The retention status was checked for the original object version:

```bash
docker compose exec silo1 mcli retention info lab/ramzan-lock-bucket/object-lock-test.txt --version-id "f51103d2-7dde-46d7-bb3c-03c60d50c658"
```

The result confirmed that Governance retention was active.

**Result:** PASS — Retention was applied to the original object version.

## 4. Verify Object Versions and Delete Marker

The version listing showed the original object version and a delete marker.

| Item            | Version ID                             | Result  |
| --------------- | -------------------------------------- | ------- |
| Original object | `f51103d2-7dde-46d7-bb3c-03c60d50c658` | Present |
| Delete marker   | `fa09675b-afc7-4a6d-b6ed-65c68751e8db` | Created |

Because versioning was enabled, the normal delete operation created a delete marker. The original version remained in the version history.

**Result:** PASS — The original version remained listed after the delete marker was created.

## 5. Test WORM Protection

A direct deletion attempt targeted the original object version without using the Governance bypass option:

```bash
docker compose exec silo1 mcli rm lab/ramzan-lock-bucket/object-lock-test.txt --version-id "f51103d2-7dde-46d7-bb3c-03c60d50c658"
```

The storage system rejected the operation with this error:

```text
Object, 'object-lock-test.txt (Version ID=f51103d2-7dde-46d7-bb3c-03c60d50c658)' is WORM protected and cannot be overwritten
```

**Result:** PASS — Direct deletion of the protected version was rejected.

## 6. Evidence

Test outputs are recorded in:

`evidence/versioning-object-lock.txt`

The evidence covers:

* Object versioning and multiple versions.
* Object Lock bucket creation.
* Governance retention configuration.
* Original version retention verification.
* Delete marker behavior.
* WORM protection during a direct deletion attempt.

## 7. Conclusion

The Silo lab successfully demonstrated object versioning, Governance retention, and WORM protection for a retained object version.

These results validate the tested storage behavior in the current lab environment. They do not establish host-level high availability or independent disk fault tolerance because all storage containers run on one EC2 instance.

**Phase 04 Status: PASS**
