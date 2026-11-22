# FastAPI Configuration with Dynaconf — Notes

Beginner notes with tester, data analyst and MLOps perspectives. Every step was run and tested locally in PyCharm.

Code: `projects/fastapi-dynaconf-config/` (`main.py`, `config.py`, `settings.toml`)

## Why configuration matters

A production-ready app should not have ports, debug flags or passwords written in the code. When the settings live outside the code, the same app runs in development and production, and only the settings change.

**Dynaconf** is a separate Python library for this. It works with FastAPI, Flask, Django and plain scripts. It is not part of FastAPI.

## Setup

```bash
pip install fastapi uvicorn dynaconf
dynaconf init -f toml
```

`dynaconf init` creates three files:

| File | Purpose |
|------|---------|
| `config.py` | Creates the `settings` object that the app imports |
| `settings.toml` | Normal settings, in sections per environment |
| `.secrets.toml` | Passwords, keys and tokens. Added to `.gitignore` automatically |

## The files

`config.py`

```python
from dynaconf import Dynaconf

settings = Dynaconf(
    envvar_prefix="DYNACONF",
    settings_files=["settings.toml", ".secrets.toml"],
    environments=True,   # use [default], [development], [production] sections
    load_dotenv=True,
)
```

> Without `environments=True`, the `[development]` and `[production]` sections are not used.

`settings.toml`

```toml
[default]
app_name = "FastAPI Config Assignment"
host = "127.0.0.1"

[development]
debug = true
port = 8000

[production]
debug = false
port = 9000
```

`.secrets.toml` (dummy values, never upload this file)

```toml
[default]
api_key = "dev-secret-123"

[production]
api_key = "prod-secret-456"
```

## Using the settings in FastAPI

```python
app = FastAPI(title=settings.APP_NAME, debug=settings.DEBUG)

@app.get("/info")
def info():
    return {"app_name": settings.APP_NAME, "environment": settings.current_env,
            "debug": settings.DEBUG, "port": settings.PORT}

@app.get("/secret-check")
def secret_check():
    return {"secret_loaded": bool(settings.get("API_KEY"))}   # never return the key itself

if __name__ == "__main__":
    uvicorn.run("main:app", host=settings.HOST, port=settings.PORT, reload=settings.DEBUG)
```

**Uvicorn does not read the config files.** Our code reads `settings` and passes the values to uvicorn.

## Switching environment and overriding values

Values are changed with **environment variables**, set in the terminal before running. In PowerShell:

```powershell
$env:ENV_FOR_DYNACONF="production"   # choose the environment
$env:DYNACONF_PORT=9500              # override one value
python main.py
```

On Linux or Cloud Shell: `ENV_FOR_DYNACONF=production DYNACONF_PORT=9500 python main.py`

Clean up afterwards, because the variables stay set in that terminal tab:

```powershell
Remove-Item Env:DYNACONF_PORT
Remove-Item Env:ENV_FOR_DYNACONF
```

**Priority order:** environment variable → value in the active environment section → `[default]` section.

## What I tested

| Setting | Result at `/info` |
|---------|-------------------|
| Nothing set | DEVELOPMENT, debug true, port 8000 |
| `DYNACONF_PORT=8500` | Still DEVELOPMENT, port 8500 |
| `ENV_FOR_DYNACONF=production` | PRODUCTION, debug false, port 9000 |
| production + `DYNACONF_PORT=9500` | PRODUCTION, port 9500 (override beats the file) |
| `GET /secret-check` in both environments | `{"secret_loaded": true}` |

## Things I learned

- **Environment and port are set separately.** Setting the port does not change the environment.
- **Settings are read once, when the server starts.** After changing a variable or file, stop the server (Ctrl+C), start it again, then refresh `/docs`. Calling an endpoint before the restart shows the old values.
- **Terminal variables stay set** in that tab until removed, so old values can leak into later tests.
- **The "note that secrets file is in .gitignore"** after `dynaconf init` is good news: the secrets will not be uploaded to GitHub.
- **Dynaconf stores credentials but does not do authorization.** Authorization is FastAPI code.
- **In real production,** pass secrets as environment variables or use a secret manager instead of a `.secrets.toml` file on disk.

## Editing settings while the server is running

The running server is not affected right away. It keeps the values it loaded at startup, so the file and the running app go out of sync until the next restart. The longer a server runs without a restart, the bigger that gap gets, so some of these risks only show up in long-running servers:

| Risk | What happens |
|------|--------------|
| Confusion | You think the new port or environment is live, but the app still uses the old values. Tests in that gap check the old configuration |
| Inconsistent servers (long-running issue) | With several instances, a partial restart means different instances behave differently. An instance that crashes and restarts on its own quietly picks up the new file, so the difference appears days later |
| Stale secrets (long-running issue) | If a key or password is changed at the provider while the server keeps running, the app keeps using the old one. Calls start failing, and the cause is hard to find because the config file already shows the new value |
| No auto-reload for `.toml` | With `reload=True`, uvicorn watches `.py` files by default, so editing a `.toml` file does not restart the server |

Dynaconf has a manual way to reload settings, but it is not automatic. **Safe habit: change the file, restart, then test.**

## Three perspectives

### 🧪 Functional testing
- Check that each environment loads the right values: environment name, debug, port
- Confirm secrets load but are never returned in a response
- Confirm an override beats the file value

### ✅ Sanity checks (configuration)
- Run the same checks against development and production settings without changing code: environment name, debug flag and port
- A config change is not live until the app restarts, so a check that runs before the restart tests the old configuration

### ⚡ Performance testing (production-like settings)
- Environment settings change how the app runs: debug mode and `reload=True` slow the app down, production does not
- In real projects, a separate Dynaconf environment called **`performance`** is used, with production-like values: debug off and reload off (in our code `reload=settings.DEBUG`). It is switched on with `ENV_FOR_DYNACONF=performance`. I have not practised this hands-on, because I only have my local machine. Numbers from development settings are misleading
- The `performance` environment runs on its **own separate server**, not on the production server. Because it is a different machine, it does not need a different port: it uses the same port as production (9000)
- A separate environment and a separate server keep performance tests away from the real `production` values and real users
- To accept connections from other machines, the host must be `0.0.0.0` (or the server's address) instead of `127.0.0.1`. Set it on that server with `DYNACONF_HOST=0.0.0.0`
- Give that server the same setup as production: same debug and reload values, similar CPU and memory, realistic data size. A smaller server gives numbers that do not match production
- Run the load tool from a **different machine**, so the tool does not use the server's CPU and change the result
- Rehearse locally first with `python main.py` in a normal terminal, not under the PyCharm debugger, because the debugger adds overhead
- Secrets on that server come from its own environment variables or a secret store. Don't copy `.secrets.toml` there
- Restart the app on the performance server after any settings change and confirm `/info` shows PERFORMANCE before measuring, so you don't measure the old configuration
- Record the environment and settings next to each result, so numbers can be compared fairly
- "Production-like" means a test copy set up like production. Don't load-test the real production server

### 📊 Data analysis
- Settings per environment make results comparable: the same test, with different configuration
- Logs can record which environment produced them

### 🔧 MLOps view
- The same pattern is used for model paths, thresholds and API keys
- Docker (Week 12) and CI/CD (Week 13) will set `ENV_FOR_DYNACONF` and overrides through environment variables

## Assignment

Make a FastAPI app configurable with Dynaconf: run `dynaconf init`, add development and production sections, add a secret, build `/info` and `/secret-check`, start uvicorn from the settings, then test the environment switch and a port override.

**Status:** ✅ Complete — all checks passed locally.

## Next steps
- Pass secrets as environment variables instead of a file (production style)
- Week 12: containerise the API with Docker and set the environment from the container
