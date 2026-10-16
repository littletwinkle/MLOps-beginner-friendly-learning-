# RFC: VM Start/Stop Monitoring via FastAPI

**Status:** Draft — not yet implemented
**Author:** K
**Related:** fastapi-handlers-async-crud-notes.md (Week 2), fastapi-traceability-matrix.md (R18)

## Summary

Extend the FastAPI learning project so that starting and stopping a virtual machine, and reading its CPU/memory stats, can be triggered and observed through FastAPI endpoints — not just stats of the machine FastAPI itself runs on.

## Motivation

The current `GET /system/stats` endpoint (R15) only reports CPU/memory of the local process (the Kaggle kernel or Cloud Shell VM running FastAPI). The next learning goal is to control and monitor a *separate* cloud VM's lifecycle and resource usage from an API layer — a common real-world MLOps pattern (e.g. spinning up compute on demand for a training job, then tearing it down).

## Current state (what's already built)

- `GET /system/stats` — local CPU/memory via `psutil`, works today, no external dependency
- `on_startup` / `on_shutdown` snapshots — capture local stats at FastAPI process start/stop, stored under `server_start` / `server_stop` keys

These are **not** the same thing as starting/stopping a VM — they only reflect the FastAPI process's own host.

## Proposed design

### What's needed beyond FastAPI
- A Google Cloud **service account** with Compute Engine permissions (`roles/compute.instanceAdmin.v1` at minimum)
- The `google-cloud-compute` Python client library
- Credentials available to the FastAPI process (via `GOOGLE_APPLICATION_CREDENTIALS` env var pointing to a service account key file)

### Proposed endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/vm/{instance_name}/start` | POST | Start a named Compute Engine VM |
| `/vm/{instance_name}/stop` | POST | Stop a named Compute Engine VM |
| `/vm/{instance_name}/status` | GET | Return VM power state (running/stopped/etc.) |
| `/vm/{instance_name}/stats` | GET | Return VM's CPU/memory via Cloud Monitoring API (not `psutil`, since FastAPI isn't running on that VM) |

### Why Cloud Monitoring, not `psutil`, for VM stats
`psutil` only reads the stats of the machine the Python process is running on. A separate VM's CPU/memory has to come from Google Cloud's **Monitoring API**, which the VM reports to automatically — FastAPI would query that API, not the VM directly.

### Sketch (not yet tested)
```python
from google.cloud import compute_v1

@app.post("/vm/{instance_name}/start")
def start_vm(instance_name: str):
    client = compute_v1.InstancesClient()
    operation = client.start(project=PROJECT_ID, zone=ZONE, instance=instance_name)
    return {"status": "starting", "instance": instance_name}

@app.post("/vm/{instance_name}/stop")
def stop_vm(instance_name: str):
    client = compute_v1.InstancesClient()
    operation = client.stop(project=PROJECT_ID, zone=ZONE, instance=instance_name)
    return {"status": "stopping", "instance": instance_name}
```
This is unverified — it has not been run yet and needs a real GCP project, zone, and service account to test.

## Alternatives considered

- **Reading VM stats via SSH + `psutil` installed on the remote VM** — rejected for now, adds SSH/networking complexity beyond this week's scope
- **Using `gcloud` CLI calls via `subprocess` instead of the client library** — simpler to prototype, but less idiomatic and harder to parse responses from; client library preferred for a learning project

## Open questions

- Does this run from Kaggle (no persistent credentials storage) or does it need Cloud Shell / a real deployment?
- Should VM start/stop also write an entry to the existing `logs_db` (R14), treating it as another `request_type`?
- Cost: Compute Engine VMs are billed while running — needs a safeguard (e.g. auto-stop after N minutes) before testing for real

## Out of scope for this RFC

- Billing/cost alerts
- Multi-VM orchestration
- Authentication/authorization on the FastAPI endpoints themselves (currently all endpoints are open, no auth layer yet)
