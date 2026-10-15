# Phase 07 — Silo Performance Benchmark

## 1. Objective

Measure basic S3-compatible object upload and download performance for the four-node Silo cluster and verify that downloaded data matches the original file.

## 2. Environment

| Component | Configuration |
|---|---|
| Object storage | Silo |
| Silo image | `pgsty/silo:RELEASE.2026-09-03T13-18-01Z` |
| Cluster | 4 Silo nodes |
| Reverse proxy | Nginx |
| Monitoring | Prometheus |
| S3 endpoint | `http://127.0.0.1:9000` |
| Benchmark tool | AWS CLI |
| Test object | `10mb.bin` |
| Object size | 10 MiB (10,485,760 bytes) |
| Benchmark date | 2026-10-15 |

## 3. Health Checks

The following checks completed successfully before the benchmark:

- Silo readiness health check returned `ready: ok`.
- S3 liveness endpoint returned HTTP 200.
- S3 readiness endpoint returned HTTP 200.
- All four Silo containers reported healthy.
- Nginx and Prometheus containers were running.
- AWS CLI successfully listed the existing S3 buckets.

## 4. Benchmark Results

| Operation | Result | Duration |
|---|---|---:|
| Upload 10 MiB object | Successful | 0.723 seconds |
| Download 10 MiB object | Successful | 0.669 seconds |
| Verify object metadata | Successful | N/A |
| List benchmark object | Successful | N/A |
| Verify downloaded data using SHA-256 | Match | N/A |

### Approximate Throughput

Throughput is calculated using the 10 MiB file size divided by the measured wall-clock time.

| Operation | Approximate Throughput |
|---|---:|
| Upload | 13.83 MiB/s |
| Download | 14.95 MiB/s |

These are single-run measurements from a small test object. They are not a formal performance comparison or a guarantee of sustained throughput.

## 5. Data Integrity Verification

The uploaded object reported a content length of 10,485,760 bytes.

The SHA-256 checksum of the original file and downloaded file matched:

`e5b844cc57f57094ea4585e235f36c78c1cd222262bb89d53c94dcb4d6b3e55d`

This confirms that the downloaded file matched the original test file.

## 6. Test Commands

### Check Silo readiness

```bash
docker compose exec -T silo1 silo healthcheck ready

````
### Upload the test object

```bash
time aws --endpoint-url http://127.0.0.1:9000 s3 cp \
  benchmarks/test-data/10mb.bin \
  s3://silo-phase07-benchmark/10mb.bin
````

### Download the test object

```bash
time aws --endpoint-url http://127.0.0.1:9000 s3 cp \
  s3://silo-phase07-benchmark/10mb.bin \
  benchmarks/downloads/10mb-downloaded.bin
```

### Verify object metadata

```bash
aws --endpoint-url http://127.0.0.1:9000 s3api head-object \
  --bucket silo-phase07-benchmark \
  --key 10mb.bin
```

### List benchmark objects

```bash
aws --endpoint-url http://127.0.0.1:9000 s3 ls \
  s3://silo-phase07-benchmark/
```

### Verify data integrity

```bash
sha256sum \
  benchmarks/test-data/10mb.bin \
  benchmarks/downloads/10mb-downloaded.bin
```

Both files should produce the same SHA-256 checksum.

## 7. Limitations

* The benchmark used a single 10 MiB test object.
* Only one measured upload and download result is documented.
* The results represent this lab environment and are not a maximum-throughput guarantee.
* Larger objects, concurrent clients, and sustained-load testing would be needed for a more comprehensive performance evaluation.

## 8. Conclusion

The four-node Silo cluster successfully uploaded and downloaded a 10 MiB object through the Nginx endpoint. Object metadata and listing were verified, and the downloaded file matched the original file's SHA-256 checksum.

The benchmark provides a basic performance baseline for this lab. Further testing would be required to evaluate sustained throughput and behavior under concurrent workloads.
