# Week 4: A containerised FastAPI service

## What I built
`hello-api` is a small web API written with FastAPI. It has a `GET /health` endpoint and a `POST /features` endpoint that takes a URL and returns its length, whether it uses HTTPS, and its dot count. It is packaged with Docker and Docker Compose, and it is on GitHub. It is the first step towards PhishGuard.

## Concepts I learned
- **API:** a defined way for one program to ask another for something. Mine uses HTTP, and the data is JSON. An API is the menu of requests and answers, and HTTP is the protocol that carries them.
- **FastAPI and uvicorn:** FastAPI is the library I use to define the endpoints, and uvicorn is the server that runs the app.
- **The `/docs` page:** FastAPI generates an interactive test page from my code, so I can see and try every endpoint without extra tools.
- **Input validation:** I limited the URL to between 1 and 2048 characters. Empty or oversized input gets a 422 error before my own code runs. This is an allowlist idea: define what is accepted and reject everything else. It protects against wasted resources and unexpected cases, but it is only one layer of defence.
- **Testing:** I wrote automated tests with pytest and FastAPI's test client, so I can check the API in a second instead of by hand.
- **`--host 0.0.0.0`:** inside a container, `127.0.0.1` means only the container itself. `0.0.0.0` makes the app listen on every network connection, so traffic that comes in through `-p` can reach it. With `127.0.0.1`, the container starts but my browser cannot reach it.
- **Port mapping:** `-p 8000:8000` connects a Mac port to a container port. I limited it with `127.0.0.1:8000:8000` so only my own Mac can reach the API.
- **Logs:** each request shows up as a log line with a status code, for example 200 for success and 422 for rejected input. Logs like these are how problems and attacks get noticed.

## Docker skills practised
- Wrote a Dockerfile (slim base image, packages installed before code, non-root `appuser`, only the `app` folder copied in)
- Used `.dockerignore` to keep `.venv`, tests and secrets out of the image
- Image size: about 54 MB, much smaller than the 91 MB cleaner because FastAPI is lighter than pandas
- Used `docker compose up --build`, `ps`, `logs`, `logs --tail`, `up -d` and `down`
- Used `docker exec` to check the container runs as `appuser`
- Rebuilt the project from a fresh clone on GitHub to prove it runs with only Docker installed

## Problems I hit and fixed
- `code .` gave "command not found". VS Code was installed, but the `code` shell command was not. I opened it with `open -a "Visual Studio Code" .` instead.
- uvicorn said `Could not import module "app.main"`. My file was called `app.py`, but the command `app.main:app` means folder `app`, file `main.py`, variable `app`. Renaming and moving it to `app/main.py` fixed it.
- My first commit only included the Docker files, so the code was still untracked. I added the code, tests and requirements in a second commit after reading `git status`.
- A test I wrote assumed `"abc"` would be rejected, but my rule only rejects empty input. The test showed me a wrong assumption. I also learned that a whitespace-only URL slips past a length check unless whitespace is stripped first.

## What I learned
- Containers have their own private network, so "which address the app listens on" and "which port is mapped" are two separate settings, and I need both.
- Tests, logs and small input limits are habits I want in every project, not extras.
- Reading the prompt tells me where I am: `Vidath@...%` is my Mac, and `appuser@...:/app$` is inside a container.