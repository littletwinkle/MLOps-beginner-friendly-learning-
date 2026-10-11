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

## Legend
- **Done** — implemented and manually verified
- **Pending** — designed/discussed but not yet coded
- **Done (design); X pending** — code exists, a specific test case is still outstanding

## Known gaps to close next
- R6: no actual PUT/PATCH/DELETE endpoints yet, only the mapping is documented
- R7: no explicit test confirming FastAPI returns `422` on a malformed `RequestLog` payload
- Persistence is all in-memory (`logs`, `responses`) — lost on restart; no database layer yet
