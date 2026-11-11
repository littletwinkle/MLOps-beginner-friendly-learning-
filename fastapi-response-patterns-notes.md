# FastAPI Response Patterns — Notes

Beginner notes with tester, data analyst and MLOps perspectives. Every example was run locally in PyCharm and tested through `/docs`.

Code: `projects/fastapi-response-patterns/main.py`

> These are four common patterns, not four official "request types" in FastAPI.

## Setup

```bash
pip install fastapi uvicorn
uvicorn main:app --reload
```

If `uvicorn` is not recognised, use `python -m uvicorn main:app --reload`. Open `http://127.0.0.1:8000/docs`.

`main:app` means the file `main.py` and the variable `app` inside it. The error "Could not import module" means uvicorn cannot find that file name, or the terminal is in the wrong folder.

---

## 1. JSON response with a Pydantic model

Pydantic models define the exact shape of data going in and out. FastAPI validates it automatically and converts the returned object to JSON.

```python
class EventIn(BaseModel):
    activity: str

class EventOut(BaseModel):
    event_id: int
    activity: str
    status: str

@app.post("/events", response_model=EventOut)
def create_event(event: EventIn): ...
```

- **Why it matters:** you can see what the user triggered (activity) and what the backend generated (event ID, status).
- **Test:** `POST /events` with `{"activity": "login"}` returns an event ID and status `recorded`.

## 2. Streaming response

`StreamingResponse` sends the response in chunks instead of all at once. Use it for live data, large files and token-by-token AI output.

```python
async def live_data():
    for i in range(1, 6):
        yield f"data: update {i}\n"
        await asyncio.sleep(1)

@app.get("/stream")
async def stream():
    return StreamingResponse(live_data(), media_type="text/plain")
```

- **Streaming is not the same as handling many users.** Many users at once is handled by async and multiple workers.
- **Test:** `curl.exe -N http://127.0.0.1:8000/stream` shows one line per second. The browser and `/docs` wait and show all 5 lines together after about 5 seconds.
- **Status code:** `200` is sent first, then the body arrives in chunks. The status code alone does not show that streaming worked. Check the body and the timing.

## 3. Controlled response with `response_model`

`response_model` shapes the outgoing response separately from the request body. This is how sensitive fields are kept from the client.

```python
class UserIn(BaseModel):
    user_id: int
    email: str
    password: str

class UserOut(BaseModel):   # no password
    user_id: int
    email: str

@app.post("/users", response_model=UserOut)
def create_user(user: UserIn):
    return user
```

- The password is received but never returned. FastAPI drops fields that are not declared in `UserOut`.
- **Security view:** the response contract is enforced in one place, so a developer cannot leak a field by accident.
- **Type tip:** values like phone numbers are identifiers, not numbers to calculate with. Use `str` so leading zeros and `+` signs are kept.

## 4. Logging with middleware

Middleware runs around every request, so one function can log all endpoints.

```python
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    ms = (time.time() - start) * 1000
    logger.info("%s %s -> %s in %.2f ms", request.method, request.url.path, response.status_code, ms)
    return response
```

Example log line: `POST /events -> 200 in 3.2 ms`

---

## Things I learned while testing

| Observation | Explanation |
|-------------|-------------|
| `/docs` shows both 200 and 422 for an endpoint | The **Responses** table lists the codes the endpoint *can* return. **Server response** shows what actually happened. 422 stays listed because the endpoint has inputs that can fail validation. |
| 422 appears when the body is wrong | The body does not match the Pydantic model: a missing field, a wrong type, or bad JSON. The response names the field in `loc`. |
| A parameter not in the Pydantic model, such as `event_handler: str`, becomes a **required query parameter** | Calling without it returns 422 with `loc: ["query", "event_handler"]`. A default value makes it optional. |
| Streaming logged `200 in 1.54 ms` | The middleware stops its timer when the response **starts**, not when the last chunk is sent. That is time to first byte. The 5 seconds of updates happen after the log line. |

To log the total streaming time, log inside the generator after the loop finishes.

---

## Three perspectives

### 🧪 Functional testing
- Status codes: `200` for valid calls, `422` for invalid bodies
- Response fields and data types match the model
- Hidden fields (like `password`) are absent from the response
- Streaming body contains all 5 updates in order

### ⚡ Performance testing
- Response time per endpoint from the middleware log
- For streaming, measure **both** time to first byte and total time
- A tester who only reads the middleware log would think `/stream` is fast

### 📊 Data analysis
- Logs can be collected and analysed: slow endpoints, error rates, most used activities
- Response structure can be checked and improved with stricter validation
- Response time can be tracked over time to find slowdowns

### 🔧 MLOps view
- The same patterns apply when wrapping a model in an API: Pydantic for input features and prediction output, streaming for long outputs, middleware logs for monitoring, `response_model` to return only what the client needs

---

## Assignment

Build a small app with:
1. A POST endpoint returning JSON with a Pydantic model
2. A streaming endpoint with 5 updates
3. A User model hiding the password with `response_model`
4. Middleware logging method, path, status and time taken

**Status:** ✅ Complete — all four run and tested locally.

## Next steps
- Send an invalid body on purpose (for example `{"user_id": "abc"}`) and confirm a real `422` appears in the server log
- Add a database so data survives a restart
- Week 12: containerise the API with Docker
