# Task → image reference

Read this when choosing an image. Always pin an explicit tag. Versions below
are examples: check the registry for currently supported tags.

| Task | Image (example) | Notes |
|---|---|---|
| Python scripts | `python:3.12-slim` | Use `--target .deps` + `PYTHONPATH=/work/.deps` for installs |
| Python + data science | `python:3.12` + pip install in container | Avoid unmaintained "all-in-one" images |
| Node.js | `node:22-slim` | Set `HOME=/tmp` when using `--user` |
| Bun / Deno | `oven/bun`, `denoland/deno` | Pin tag |
| Go | `golang:1.23` | Set `GOCACHE=/tmp/gocache`, `GOPATH=/tmp/go` |
| Rust | `rust:1-slim` | Set `CARGO_HOME=/tmp/cargo` |
| Java / Maven | `maven:3-eclipse-temurin-21` | |
| Ruby | `ruby:3.3-slim` | |
| PHP / Composer | `composer:2` | |
| .NET | `mcr.microsoft.com/dotnet/sdk:8.0` | Set `DOTNET_CLI_HOME=/tmp` |
| Documents (md → pdf/docx/html) | `pandoc/core`, `pandoc/latex` | |
| Video/audio | `jrottenberg/ffmpeg` | Check tag; or a maintained alternative |
| Images | `dpokidov/imagemagick` | Verify publisher before use |
| PDF tools | `pdftk`/`qpdf`-based images | Verify publisher; or build a small Dockerfile |
| Static site / docs | `squidfunk/mkdocs-material` | |
| Linting | `hadolint/hadolint`, `koalaman/shellcheck`, `pipelinecomponents/*` | |
| Databases (dev) | `postgres:16`, `mysql:8`, `redis:7` | Bind ports to `127.0.0.1` only |
| Terraform | `hashicorp/terraform` | Never mount cloud credentials unless asked |

## Tips

- Prefer `-slim` or `-alpine` variants, but note Alpine (musl) can break some
  Python/Node native packages. Fall back to `-slim` (Debian).
- When `--user` is set, the home directory may not exist or be writable.
  Set `-e HOME=/tmp` (or a cache volume).
- Use named volumes for caches (`pip-cache`, `npm-cache`, `cargo-cache`) and
  mention them to the user so they can run `docker volume rm` later.
- If a tool must be run often, suggest a `Dockerfile` or `compose.yaml`
  committed to the repo instead of long `docker run` commands.
