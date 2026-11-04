# CSV Cleaner

A small Python and pandas tool that cleans a CSV of URLs (removes duplicates and blank rows, fills missing labels) and adds simple features such as URL length, HTTPS use and dot count. It is packaged with Docker so it runs the same on any machine.

## Run it

Requires Docker only. No Python or pandas install needed.

```bash
git clone https://github.com/vidathhh9/week2-csv-cleaner.git
cd week2-csv-cleaner
docker compose run --rm cleaner
```

The cleaned data is written to `urls_clean.csv` in the project folder.

## What I did in Week 3: Docker basics

- **Containerised the project.** I wrote a `Dockerfile` based on `python:3.15-slim`. The resulting image is about 91 MB.
- **Optimised build caching.** `requirements.txt` is copied and installed before the code, so editing code doesn't reinstall packages. I proved this by comparing it with a deliberately badly ordered Dockerfile.
- **Ran as a non-root user.** The container runs as `appuser` instead of `root`, which limits the damage if the app were ever compromised.
- **Used volume mounts** (`-v`) so the container reads live files from my Mac and saves `urls_clean.csv` back, without rebuilding the image.
- **Added Docker Compose** (`compose.yaml`) to replace a long `docker run` command with one short one.
- **Practised ports and debugging.** I mapped a container port to my Mac (`-p 8080:8000`), and used `docker ps`, `docker logs`, `docker history`, `docker system df` and a shell inside a container to inspect things.
- **Added a `.dockerignore`** to keep `.venv`, `.git` and generated files out of the image.
- **Tested it like a stranger would.** I cloned the repo fresh from GitHub and ran it with one command.

## Problems I hit and fixed

- A mounted volume from the wrong folder hid `clean.py` inside the container and caused "can't open file". The fix was to run from the project folder.
- My file was named `dockerfile` in lowercase. It worked on macOS but could fail on Linux and in CI, so I renamed it to `Dockerfile`.
- Generated output (`urls_clean.csv`) kept showing up in Git, so I added it to `.gitignore`.

## What I learned

- An image is a read-only template, and a container is a disposable running copy of it.
- `docker build` installs everything once; `docker run` starts the app from the finished image.
- Containers isolate the whole environment, not just Python packages.
