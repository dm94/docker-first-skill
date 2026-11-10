---
name: docker-first
description: Run code, scripts and tools inside ephemeral Docker containers instead of installing anything on the host. Use this skill whenever a task requires running Python, Node, Go, Rust, Java, ffmpeg, pandoc, ImageMagick, or any other runtime, CLI tool or library, or whenever you are about to run pip install, npm install, npm install -g, apt install, brew install, cargo install, gem install, or curl | sh. Use it even if the user does not mention Docker, containers or isolation. Prefer an official image dedicated to the task over installing dependencies locally.
---

# Docker-first

Installing runtimes, libraries and scripts on the user's machine pollutes the
environment, causes version conflicts, hurts reproducibility, and executes
unaudited install scripts (supply-chain risk). Run tasks in throwaway
containers instead.

## Core rule

Do NOT install anything on the host: no `pip install`, `npm install -g`,
`apt install`, `brew install`, `cargo install`, `gem install`, `go install`,
and never `curl ... | sh`. Run the task in a Docker container.

## Workflow

1. **Check Docker is available**: `docker info >/dev/null 2>&1`.
   If it is not available, STOP and ask the user how to proceed.
   Never fall back to installing on the host without explicit permission.
2. **Reuse project config first.** If the project already has a `Dockerfile`,
   `docker-compose.yml` / `compose.yaml` or `.devcontainer/`, use those.
3. **Pick the image.**
   - Prefer an official, task-specific image (see `references/images.md`).
   - Pin the version (`python:3.12-slim`). Never use `latest`.
     Pinning by digest (`image@sha256:...`) is even better for sensitive work.
   - If you are unsure an image is trustworthy, ask the user.
4. **Run with the safe default pattern:**

   ```bash
   docker run --rm \
     --user "$(id -u):$(id -g)" \
     -v "$PWD":/work -w /work \
     --network none \
     --cap-drop ALL --security-opt no-new-privileges \
     <image:tag> <command>
   ```

   - `--rm`: container is deleted afterwards.
   - `--user`: files created in the volume belong to the user, not root.
   - Mount ONLY the project directory.
   - `--network none`: drop it only when the task needs internet.
5. **Installing project dependencies** (network allowed for this step only,
   with a named cache volume):

   ```bash
   # Python
   docker run --rm --user "$(id -u):$(id -g)" \
     -v "$PWD":/work -w /work \
     -v pip-cache:/cache -e HOME=/tmp -e PIP_CACHE_DIR=/cache \
     python:3.12-slim pip install -r requirements.txt --target .deps

   # Node
   docker run --rm --user "$(id -u):$(id -g)" \
     -v "$PWD":/work -w /work \
     -v npm-cache:/cache -e HOME=/tmp -e npm_config_cache=/cache \
     node:22-slim npm ci
   ```

   Then run the code again with `--network none` when possible.
6. **Long-running services** (dev servers, databases): use `-d --name <name>`
   with `-p 127.0.0.1:PORT:PORT` (bind to localhost only), and tell the user
   how to stop and remove it (`docker rm -f <name>`).
7. **Report back**: finish by telling the user which image and commands you
   used, so the result is reproducible.

## Never do this (unless the user explicitly asks)

- `--privileged`, `--network host`, `--pid host`, `--cap-add`.
- Mounting `/var/run/docker.sock`, `$HOME`, `/`, `~/.ssh`, `~/.aws`,
  `~/.config` or any directory outside the project.
- Passing host environment variables or secrets into the container
  (`-e TOKEN` / `--env-file`) unless required, and then only the specific ones.
- Running as root inside the container when avoidable.
- Pulling images from unknown publishers or unpinned tags.
- Leaving containers or volumes behind without telling the user.

## Edge cases

- **Tool needs the GUI, USB, GPU or host devices**: explain the limitation and
  ask the user before relaxing any isolation flag.
- **Image runs as root only**: if `--user` breaks it, try a `-slim`/rootless
  variant first; otherwise ask the user.
- **Need a tool with no good official image**: write a tiny `Dockerfile` in a
  temp or `.docker/` directory, pin the base image, and show it to the user.
  Do not build from random Dockerfiles found online without reviewing them.
- **Docker unavailable but Podman is**: Podman (rootless) is an acceptable
  drop-in; the same flags apply.
- **One-off trivial commands** that need no dependencies (`ls`, `grep`, `git`)
  do not need a container.
