# 03 — S3 Basic Operations

## Goal

Validate the basic S3-compatible object-storage capabilities of the distributed Silo cluster using the Silo client, `mcli`.

This phase verifies bucket management, object upload and download, data integrity, object metadata, object tags, and temporary access through a presigned URL.

The tests use synthetic data and are intended to establish a basic functional baseline for the Silo deployment.

---

## Environment

| Item                | Details                                             |
| ------------------- | --------------------------------------------------- |
| Storage System      | Silo                                                |
| Deployment          | Docker Compose                                      |
| Operating System    | Ubuntu Linux                                        |
| Container Runtime   | Docker                                              |
| Silo Image          | `docker.io/pgsty/silo:RELEASE.2026-09-03T13-18-01Z` |
| Client              | `mcli`                                              |
| Test File           | `benchmarks/test-data/hello.txt`                    |
| Test Object         | `ramzan-lab-bucket/hello.txt`                       |
| Object Size         | 34 bytes                                            |
| Integrity Algorithm | SHA-256                                             |
| Test Data           | Synthetic text file                                 |
| Evidence Date       | 2026-10-09                                          |

---

## Step 1 — Create a Bucket

### Command

```bash
docker compose exec silo1 mcli mb lab/ramzan-lab-bucket
```

### Expected Result

The command should create the `ramzan-lab-bucket` bucket successfully.

### Actual Result

The bucket was created successfully.

**Result: PASS**

---

## Step 2 — Verify the Bucket

### Command

```bash
docker compose exec silo1 mcli ls lab
```

### Expected Result

The bucket listing should contain `ramzan-lab-bucket`.

### Actual Result

The newly created bucket appeared in the listing.

**Result: PASS**

---

## Step 3 — Prepare the Test Object

The synthetic test file was prepared at:

```text
benchmarks/test-data/hello.txt
```

### File Content

```text
Silo distributed storage lab test
```

The test file was used to validate object upload, download, metadata, tags, and checksum integrity.

---

## Step 4 — Upload an Object

### Command

```bash
docker compose exec silo1 mcli cp /tmp/hello.txt lab/ramzan-lab-bucket/hello.txt
```

### Expected Result

The client should upload the local test file to the specified bucket.

### Actual Result

The `hello.txt` object was uploaded successfully and subsequently verified in the bucket.

| Property    | Value               |
| ----------- | ------------------- |
| Object Name | `hello.txt`         |
| Bucket      | `ramzan-lab-bucket` |
| Object Size | 34 bytes            |

**Result: PASS**

---

## Step 5 — Download an Object

### Command

```bash
docker compose exec silo1 mcli cp lab/ramzan-lab-bucket/hello.txt /tmp/downloaded-hello.txt
```

### Expected Result

The object should be downloaded to the specified local path.

### Actual Result

The object was downloaded successfully.

**Result: PASS**

---

## Step 6 — Verify Data Integrity Using SHA-256

The original file checksum was compared with the downloaded file checksum.

### Original File SHA-256

```text
02b810feffefcc128e191ec7d74a8db206e75d02eebefc2d14a2de989158434a
```

### Downloaded File SHA-256

```text
02b810feffefcc128e191ec7d74a8db206e75d02eebefc2d14a2de989158434a
```

### Expected Result

Both SHA-256 checksums should match.

### Actual Result

The original and downloaded files produced identical SHA-256 checksums.

This confirms that the downloaded file's contents match the original test file.

**Result: PASS**

---

## Step 7 — Retrieve Object Metadata

### Command

```bash
docker compose exec silo1 mcli stat lab/ramzan-lab-bucket/hello.txt
```

### Actual Result

The object metadata was retrieved successfully.

| Property     | Value                              |
| ------------ | ---------------------------------- |
| Name         | `hello.txt`                        |
| Size         | 34 B                               |
| ETag         | `d4646d3a533a4cae25067339ce6471b8` |
| Type         | `file`                             |
| Content-Type | `text/plain`                       |

### Expected Result

The client should return the object's metadata.

### Actual Result

The object name, size, ETag, type, and content type were retrieved successfully.

**Result: PASS**

---

## Step 8 — Set and Verify Object Tags

### Command

```bash
docker compose exec silo1 mcli tag set lab/ramzan-lab-bucket/hello.txt "Environment=Lab&Project=Silo"
```

### Expected Result

The command should set the requested tags on the object.

### Actual Result

The tags were set and subsequently listed successfully.

Verified tags:

* `Environment=Lab`
* `Project=Silo`

**Result: PASS**

---

## Step 9 — Generate and Test a Presigned URL

A temporary download URL was generated for the stored object.

### Command

```bash
docker compose exec silo1 mcli share download lab/ramzan-lab-bucket/hello.txt --expire 5m
```

The URL was then used to download the object through the local Nginx routing configuration.

### Download Command

```bash
curl --connect-to silo1:9000:127.0.0.1:9000 \
  -f \
  -o /tmp/presigned-hello.txt \
  "$PRESIGNED_URL"
```

`PRESIGNED_URL` represents the temporary URL generated by `mcli`; the actual signed URL is intentionally excluded from the repository evidence.

### Expected Result

The presigned URL should permit downloading the object while the URL remains valid.

### Actual Result

The object was downloaded successfully using the presigned URL.

Downloaded file:

```text
/tmp/presigned-hello.txt
```

Downloaded content:

```text
Silo distributed storage lab test
```

Downloaded file SHA-256:

```text
02b810feffefcc128e191ec7d74a8db206e75d02eebefc2d14a2de989158434a
```

The checksum matched the original file.

**Result: PASS**

### Security Note

Presigned URLs contain temporary access signatures and must be treated as sensitive while valid. The actual signed URL was excluded from the evidence file.

---

## Results Summary

| Test                         | Result |
| ---------------------------- | ------ |
| Bucket creation              | PASS   |
| Bucket listing               | PASS   |
| Object upload                | PASS   |
| Object download              | PASS   |
| SHA-256 integrity comparison | PASS   |
| Object metadata retrieval    | PASS   |
| Object tags                  | PASS   |
| Presigned URL download       | PASS   |
| Presigned download integrity | PASS   |

**Overall Result: PASS — for the basic S3 operations documented in this phase.**

---

## Findings

1. The Silo client successfully created and listed an S3-compatible bucket.
2. The cluster accepted an object upload and returned the object on download.
3. Matching SHA-256 checksums confirmed that the downloaded test file matched the original.
4. Object metadata, including size, ETag, and content type, was available.
5. Object tags were set and verified.
6. A presigned URL provided temporary access to the object.
7. The presigned URL download produced a file with the expected content and matching checksum.
8. The signed URL was excluded from the public evidence to avoid exposing temporary access credentials.

---

## Scope and Limitations

This phase verifies basic S3-compatible operations using one small synthetic object.

The following capabilities are **not established by the evidence in this phase**:

* Multipart upload of an object at least 1 GB in size
* Python SDK validation using `boto3`
* Bucket versioning and restoration of previous versions
* Object Lock and retention enforcement
* Node and drive failure behaviour
* Healing after data loss
* Full-cluster integrity verification following failure recovery

These capabilities must be documented in their respective phases using their own test results and evidence.

Additionally, all four storage nodes run on the same EC2 instance and underlying host storage. Therefore, this lab does not demonstrate resilience against a physical-host failure.

---

## How to Reproduce

Run the commands from the Silo project directory.

Create the bucket:

```bash
docker compose exec silo1 mcli mb lab/ramzan-lab-bucket
```

List the bucket:

```bash
docker compose exec silo1 mcli ls lab
```

Upload the test object:

```bash
docker compose exec silo1 mcli cp /tmp/hello.txt lab/ramzan-lab-bucket/hello.txt
```

Download the object:

```bash
docker compose exec silo1 mcli cp lab/ramzan-lab-bucket/hello.txt /tmp/downloaded-hello.txt
```

Retrieve metadata:

```bash
docker compose exec silo1 mcli stat lab/ramzan-lab-bucket/hello.txt
```

Verify object tags:

```bash
docker compose exec silo1 mcli tag list lab/ramzan-lab-bucket/hello.txt
```

Compare checksums using the SHA-256 utility available in the environment.

To test the presigned URL workflow, generate a new URL and download the object before the URL expires.

> The bucket and object names must be available in the configured Silo environment. Repeated runs may require checking for existing objects before uploading.

---

## Evidence

The raw evidence for this phase is stored in:

```text
docs/03-s3-operations/evidence/s3-basic-operations.txt
```

This evidence records the basic S3 commands, their results, checksum comparisons, object metadata, tags, and presigned URL validation.

Screenshots, if available, should be stored in:

```text
docs/03-s3-operations/screenshots/
```

Raw command output is the primary evidence; screenshots are supplementary.

---

## Cleanup

If the test object is no longer required, remove it:

```bash
docker compose exec silo1 mcli rm lab/ramzan-lab-bucket/hello.txt
```

Remove the test bucket only after confirming that it contains no other required data:

```bash
docker compose exec silo1 mcli rb lab/ramzan-lab-bucket
```

> Cleanup is optional. Do not remove the bucket or object if they are needed for subsequent phases, including versioning, Object Lock, or other S3 tests.

