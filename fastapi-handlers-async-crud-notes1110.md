# FastAPI Learning Notes: Handlers, Async, Multiple Servers and CRUD

Hands-on notes from my first week with **FastAPI**, run in Kaggle and Google Cloud Shell.

**Status:** scikit-learn study is still in progress. This week's new focus was FastAPI, built alongside it as part of my MLOps journey.

Part of my **mlops-learning-journey** repository, where I publish structured notes as I learn in public.

## This week's summary

- Built and ran my first FastAPI handler — mapped a piecewise math function to a route (`f(x) = x²` if `x > 0`, else `x`)
- Learned `asyncio.sleep` vs `time.sleep` — non-blocking sleep lets concurrent requests actually run concurrently
- Ran multiple FastAPI servers (5 instances on separate ports) and spread 10 async requests across them
- Designed a Pydantic schema (`RequestLog`) to validate JSON requests — client ID, request type, timestamp, seconds
- Captured login/logout/create/update/delete actions as request logs, retrievable via a `/logs/past`-style endpoint
- Built an HTML survey form (`GET /survey`) and handled form-encoded submissions with `Form(...)` (`POST /survey`), distinct from JSON body handling — captures data from both server and front end
- Confirmed FastAPI auto-serializes return values to JSON (no manual `json.dumps`), the same way it auto-validates incoming JSON via Pydantic
- Ran everything in Kaggle (via `nest_asyncio` + background threads, no `main.py` needed) and in Google Cloud Shell (standard `uvicorn main:app`)
- Put together a traceability matrix mapping each concept to its implementation and test status, for future reference

## What is FastAPI?

FastAPI is a modern **Python web framework** for building APIs. It runs on an ASGI server called **uvicorn**.

- Very fast performance (async-first)
- Automatic JSON serialization and validation via type hints (Pydantic)
- JSON supports more than integers: numbers (int and float), strings, booleans, null, lists and objects
- Auto-generated docs at `/docs`

## 1. Handlers

A **handler** is the function attached to a route. It takes the request as input and returns a response. That is a mathematical function in code form.

Piecewise example:

f(x) = x² if x > 0, and f(x) = x if x ≤ 0

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/f/{x}")
def piecewise_handler(x: int):
    result = x ** 2 if x > 0 else x
    return {"input": x, "output": result}
```

| Input | Output |
|-------|--------|
| 5     | 25     |
| -3    | -3     |
| 0     | 0      |

Related ideas:

- Type hints (`x: int`) define the function's domain
- Each route is one piece of the larger system
- Exception handlers are the "else" branch
- Middleware is function composition

An **info handler** (`GET /info`) is a small endpoint that returns service metadata and status, used for health checks and monitoring.

## 2. `asyncio.sleep` vs `time.sleep`

Inside an `async def` handler, `time.sleep()` **blocks the whole event loop**, so requests queue up and each waits for the previous one. `await asyncio.sleep()` hands control back to the event loop, so many requests are processed concurrently.

```python
import asyncio

@app.get("/work")
async def work():
    await asyncio.sleep(1)   # non-blocking
    return {"status": "done"}
```

10 concurrent requests finish in about 1s with `asyncio.sleep`, versus about 10s if blocking.

## 3. Multiple servers

Run several uvicorn instances on different ports and spread requests across them.

In a Jupyter/Kaggle notebook, `uvicorn.run(..., workers=N)` does not work well, so run each server in a background thread:

```python
import asyncio, threading, time, uvicorn, httpx, nest_asyncio
from fastapi import FastAPI

nest_asyncio.apply()
app = FastAPI()

@app.get("/work")
async def work():
    await asyncio.sleep(1)
    return {"status": "done"}

PORTS = [8001, 8002, 8003, 8004, 8005]

def run_server(port):
    uvicorn.run(app, host="0.0.0.0", port=port, log_level="warning")

for p in PORTS:
    threading.Thread(target=run_server, args=(p,), daemon=True).start()
time.sleep(2)

async def call(port):
    async with httpx.AsyncClient() as client:
        return (await client.get(f"http://127.0.0.1:{port}/work")).json()

async def main():
    return await asyncio.gather(*[call(PORTS[i % 5]) for i in range(10)])

print(asyncio.run(main()))
```

10 requests are round-robined across 5 servers (2 each) and run concurrently.

## 4. Request logging with Pydantic

Every user action (login, logout, create, update, delete) can be captured as a request log using a Pydantic schema, then retrieved later through a history endpoint.

```python
from pydantic import BaseModel
from datetime import datetime
from enum import Enum
from fastapi import FastAPI

class RequestType(str, Enum):
    login = "login"
    logout = "logout"
    create = "create"
    update = "update"
    delete = "delete"

class RequestLog(BaseModel):
    client_id: str
    request_type: RequestType
    timestamp: datetime
    seconds: int

app = FastAPI()
logs: list[RequestLog] = []

@app.post("/log")
def capture_request(entry: RequestLog):
    logs.append(entry)
    return {"status": "captured"}

@app.get("/logs/past")
def get_past_logs():
    return logs
```

Pydantic doesn't speed up JSON creation — it **validates and structures** incoming JSON into a typed Python object (and back to JSON on the way out), so you get automatic parsing and validation without writing that boilerplate yourself.

**On the response side**, FastAPI does the reverse automatically too: when a handler returns a Python object (a dict, a list, a Pydantic model), FastAPI serializes it to JSON itself — there's no need to call `json.dumps()` or build the JSON manually. Returning `logs` (a list of `RequestLog` objects) from `/logs/past` is enough; FastAPI converts it to a JSON response on the way out.

## 5. HTML forms

FastAPI can serve and receive HTML forms, which is a different data path from JSON. A form submission arrives as `application/x-www-form-urlencoded` (or `multipart/form-data` for file uploads), not a JSON body — so fields are declared individually with `Form(...)` instead of a single Pydantic model.

```python
from fastapi import FastAPI, Form
from fastapi.responses import HTMLResponse

app = FastAPI()
responses = []

@app.get("/survey", response_class=HTMLResponse)
def show_form():
    return """
    <html><body>
        <h2>Survey</h2>
        <form action="/survey" method="post">
            Name: <input type="text" name="name"><br>
            Email: <input type="text" name="email"><br>
            Rating (1-5): <input type="number" name="rating" min="1" max="5"><br>
            <input type="submit" value="Submit">
        </form>
    </body></html>
    """

@app.post("/survey")
def submit_form(name: str = Form(...), email: str = Form(...), rating: int = Form(...)):
    entry = {"name": name, "email": email, "rating": rating}
    responses.append(entry)
    return {"status": "saved", "entry": entry}

@app.get("/survey/responses")
def get_responses():
    return responses
```

This captures data from both sides — the server renders the form (`GET /survey`), and the front end's submitted data is captured back on the server (`POST /survey`) and logged for later retrieval (`GET /survey/responses`), the same request-log pattern as section 4. Requires `pip install python-multipart` for form parsing.

In Kaggle, there's no browser preview — the form HTML and submissions are tested via `httpx.get(...)` / `httpx.post(..., data={...})` instead of filling the form visually.

## 6. CRUD

| Operation | HTTP method | Purpose |
|-----------|-------------|---------|
| Create    | `POST`      | Add a new resource |
| Read      | `GET`       | Fetch a resource |
| Update    | `PUT` / `PATCH` | Modify a resource |
| Delete    | `DELETE`    | Remove a resource |

## 7. Run in Google Cloud Shell

```bash
pip install fastapi uvicorn
uvicorn main:app --host 0.0.0.0 --port 8080
```

Open **Web Preview -> Preview on port 8080**, or test with `curl http://localhost:8080/info`.

## Key takeaways

- A handler is a function: input -> rule -> output
- Use `asyncio.sleep` (not `time.sleep`) in async handlers
- Multiple servers = multiple uvicorn instances on different ports
- CRUD maps to POST, GET, PUT/PATCH and DELETE
- HTML forms use `Form(...)`, not a JSON body — a separate data path from Pydantic-validated requests

## Next steps

- Build full CRUD endpoints
- Add a validation test confirming FastAPI returns `422` on a malformed `RequestLog`
- Move from in-memory storage to a database layer
- Deploy to Cloud Run
