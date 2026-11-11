# FastAPI Learning Notes: Handlers, Async, Multiple Servers and CRUD

Hands-on notes from my first week with **FastAPI**, run in Kaggle and Google Cloud Shell.

**Status:** scikit-learn study is still in progress. I've just started building my first API with FastAPI alongside it as part of my MLOps journey.

Part of my **mlops-learning-journey** repository, where I publish structured notes as I learn in public.

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

Related ideas (analogies to help remember, not exact definitions):

- Type hints (`x: int`) define the function's domain: FastAPI rejects input that doesn't fit
- Each route is one piece of the larger system
- Exception handlers cover the cases the normal route can't handle, a bit like an "else" branch
- Middleware wraps every request and response, similar to function composition

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

## 4. CRUD

| Operation | HTTP method | Purpose |
|-----------|-------------|---------|
| Create    | `POST`      | Add a new resource |
| Read      | `GET`       | Fetch a resource |
| Update    | `PUT` / `PATCH` | Modify a resource |
| Delete    | `DELETE`    | Remove a resource |

## 5. Run in Google Cloud Shell

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

## Next steps

- Add Pydantic models for request bodies
- Build full CRUD endpoints
- Deploy to Cloud Run
