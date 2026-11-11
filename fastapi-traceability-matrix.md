# FastAPI Learning — Traceability Matrix

Maps each concept covered to its design intent, where it's implemented, and how it was verified. Use this to track what's done and what's still pending as the project grows.

| ID | Requirement / Concept | Design Intent | Implementation (file / function) | Test Method | Status |
|----|------------------------|----------------|-----------------------------------|--------------|--------|
| R1 | Handler (route function) | Map an HTTP input to a deterministic output, like a math function | `piecewise_handler(x)` in `main.py` | `curl /f/{x}` with positive, negative, zero inputs | Done |
| R2 | Info handler | Expose service metadata/status for health checks | `info()` → `GET /info` | `curl /info` | Done |
| R3 | Single server, single request | Confirm a minimal FastAPI + uvicorn server responds to one call | `run_server()` on port 8000 + `httpx.get` | Manual `httpx.get("/ping")` | Done |
| R4 | `asyncio.sleep` vs `time.sleep` | Demonstrate non-blocking I/O so concurrent requests don't queue | `work()` with `await asyncio.sleep(1)` | Compare total time for 10 requests (~1s vs ~10s) | Done |
| R5 | Multiple servers (5 instances) | Simulate load distribution across independent server processes | 5× `threading.Thread(run_server, args=(port,))` on ports 8001–8005 | `asyncio.gather` with 10 round-robined requests | Done |
| R6 | CRUD mapping | Map standard data operations to HTTP methods | POST (create), GET (read), PUT/PATCH (update), DELETE (delete) | Not yet implemented as endpoints | Pending |
| R7 | Pydantic request schema | Validate and structure incoming JSON automatically | `RequestLog(BaseModel)` with `client_id`, `request_type`, `timestamp`, `seconds` | POST invalid payload → expect `422` | Done (design); validation test pending |
| R8 | Request/action logging | Capture login, logout, create, update, delete events | `POST /log`, in-memory `logs` list | `POST /log` then check list grows | Done |
| R9 | Past log retrieval | Retrieve historical logs in real time | `GET /logs/past` | `httpx.get("/logs/past")` | Done |
| R9a | Automatic JSON response serialization | Confirm FastAPI converts returned Python objects to JSON without manual `json.dumps()` | `return logs` in `get_past_logs()` | `httpx.get("/logs/past")` → inspect response is valid JSON | Done |
| R10 | HTML form handling | Accept form-encoded data (not JSON) from a survey page | `GET /survey` (HTML) + `POST /survey` using `Form(...)` | `httpx.get` to view HTML; `httpx.post` with `data={...}` to simulate submission | Done |
| R11 | Form response storage | Persist survey submissions server-side | `responses` list in `submit_form()` | `GET /survey/responses` | Done |
| R12 | Kaggle-compatible execution | Run FastAPI inside a notebook kernel without `main.py` / CLI | `nest_asyncio.apply()` + background thread + `httpx` client calls | Run all cells in Kaggle, confirm no port/file errors | Done |
| R13 | Cloud Shell execution | Run FastAPI as a standalone script outside a notebook | `main.py` + `uvicorn main:app --host 0.0.0.0 --port 8080` | Cloud Shell Web Preview on port 8080 | Done |
| R14 | Unique ID per log entry | Allow a specific record (e.g. one login) to be retrieved individually, not just the full list | `uuid.uuid4()` key in `logs_db` dict, set in `POST /log` | `GET /log/{log_id}` returns the matching entry; invalid ID returns `404` | Done |
| R15 | System stats endpoint | Report CPU/memory of the machine FastAPI runs on | `psutil.cpu_percent()`, `psutil.virtual_memory()` in `GET /system/stats` | `httpx.get("/system/stats")` | Done |
| R16 | Start/stop stats snapshots | Distinguish and compare system stats at server start vs. stop | `capture_stats()` stored under `server_start` / `server_stop` keys in `on_startup` / `on_shutdown` | `GET /system/snapshots`; `server_start` confirmed populated | Done (start); stop pending clean shutdown trigger |
| R17 | Real-time (live) user input | Confirm whether real users can submit forms live | N/A — architectural limitation, not code | Confirmed Kaggle has no browser-facing URL; only Cloud Shell / real deployment supports this | Not possible in Kaggle (documented limitation) |
| R18 | VM start/stop monitoring via FastAPI | Start/stop a separate cloud VM and read its stats from FastAPI | Not implemented — requires Google Cloud Compute Engine API + service account credentials, beyond FastAPI alone | N/A | Pending (scoped, not started) |
| R19 | `response_model` validation | Shape/validate outgoing response separately from the request body model | `response_model=UserOut` on `POST /users/`, distinct from `UserIn` request model | Confirm `password` field is absent from the JSON response even though present on `user` internally | Done |
| R20 | Hiding sensitive fields via response model | Prevent sensitive fields (e.g. password) from ever reaching the client | `UserOut` omits `password`; FastAPI drops undeclared fields automatically | Compare `UserIn` vs `UserOut` response payload | Done |
| R21 | Response-only extra fields | Add fields to a response that aren't part of the base/request model | `ItemOut(Item)` extends `Item` with `branch_name` | `GET /items/{id}` returns `branch_name` not present on `Item` | Done |
| R22 | `response_model=None` | Skip response validation for non-standard responses (redirects, raw JSON) | `response_model=None` on a `RedirectResponse` route | `GET /go` returns a redirect, not validated JSON | Done |
| R23 | JSON response with Pydantic model | Return a structured response (event ID, activity, status) so user actions can be tracked | `EventIn` / `EventOut` models, `POST /events` with `response_model=EventOut` in `projects/fastapi-response-patterns/main.py` | `POST /events` with `{"activity": "login"}` in `/docs` → `200` with event ID 1 | Done |
| R24 | Streaming response | Send live data in chunks instead of one response | `StreamingResponse(live_data())` on `GET /stream`, 5 updates, 1 second apart | `GET /stream` → `200`; body has 5 updates; `curl.exe -N` shows one line per second | Done |
| R25 | Request logging via middleware | Log method, path, status and time taken for every call from one place | `@app.middleware("http")` `log_requests()` | Call each endpoint, check PyCharm terminal for one log line per call | Done |
| R26 | Streaming timing | Understand what the middleware time means for a streaming endpoint | Middleware log for `GET /stream` | Log showed `200 in 1.54 ms` (time to first byte); total time is ~5 s because the middleware stops at response start | Done (time to first byte); total-time logging inside the generator suggested |
| R27 | Reading `/docs` response codes | Tell possible responses apart from actual results | `/docs` **Responses** table vs **Server response** | Confirmed 422 listed on a successful call; real calls checked in the terminal log | Done (documented) |

## Legend
- **Done** — implemented and manually verified
- **Pending** — designed/discussed but not yet coded
- **Done (design); X pending** — code exists, a specific test case is still outstanding

## Known gaps to close next
- R6: no actual PUT/PATCH/DELETE endpoints yet, only the mapping is documented
- R7: no explicit test confirming FastAPI returns `422` on a malformed payload. Send a wrong body on purpose (for example `{"user_id": "abc"}` to `POST /users`) and confirm `-> 422` in the server log
- R16: `server_stop` snapshot needs a way to cleanly trigger shutdown from within the same Kaggle run to verify
- R18: VM start/stop monitoring is scoped but not started — needs Compute Engine API design (see RFC)
- R26: total streaming time not yet logged; add a log line inside `live_data()` after the loop
- Persistence is all in-memory (`logs`, `responses`, `logs_db`, event counter) — lost on restart; no database layer yet
