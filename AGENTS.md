# Pi Docker

A dockerised [pi coding agent](https://github.com/earendil-works/pi-coding-agent) with a curated set of tools, extensions, and skills — ready to run anywhere with a single `docker run`.

## What's inside

### Core

| Component | Version | Source |
|-----------|---------|--------|
| pi coding agent | latest | `npm: @earendil-works/pi-coding-agent` |
| rtk | latest | `cargo: github.com/rtk-ai/rtk` |

### Extensions

| Extension | Package | What it does |
|-----------|---------|-------------|
| subagent | `npm:pi-subagents` | Delegate tasks to subagents — chains, parallel, async, forked context |
| web-access | `npm:pi-web-access` | Web search, URL fetch, GitHub clone, YouTube, PDF, video |
| session-inspector | `git: RemyFevry/pi-session-inspector` | Footer + `/inspect` overlay — token/cost breakdown, dust bars |
| rtk | built by `rtk init` | Agent toolkit |

### CLI tools

| Tool | Source | What it does |
|------|--------|-------------|
| `sd` (sonde) | `cargo: github.com/RemyFevry/sonde` | LSP-backed structured code discovery — outline, callers, callees, subclasses, refs |

### Skills

All 29 of [Matt Pocock's engineering skills](https://github.com/mattpocock/skills) are included — diagnose, tdd, review, prototype, writing-beats, etc.

### System tools

`bash`, `build-essential`, `ca-certificates`, `curl`, `git`, `ripgrep`, `rust` (via rustup).

## Quick start

### Build

```bash
docker build -f Dockerfile.pi -t pi-docker .
```

### Run

```bash
# Interactive — requires an API key
docker run -it --rm \
  -e GEMINI_API_KEY=$GEMINI_API_KEY \
  -v $(pwd):/workspace \
  pi-docker

# Non-interactive one-shot
docker run --rm \
  -e GEMINI_API_KEY=$GEMINI_API_KEY \
  -v $(pwd):/workspace \
  pi-docker -p "List all .ts files"

# Run sd (sonde) directly
docker run --rm --entrypoint sd pi-docker outline src/main.ts
```

Provider env vars: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `GROQ_API_KEY`, `DEEPSEEK_API_KEY`, `XAI_API_KEY`, `OPENROUTER_API_KEY`, and [many more](https://github.com/earendil-works/pi-coding-agent).

### Verify the image

```bash
docker run --rm --entrypoint sd pi-docker --help      # sonde CLI works
docker run --rm --entrypoint pi pi-docker list          # extensions registered
docker run --rm --entrypoint ls pi-docker /root/.pi/agent/extensions/  # session-inspector, rtk
```

## Project structure

```
pi-docker/
├── Dockerfile.pi       # The Dockerfile
├── .dockerignore       # Build exclusions
└── AGENTS.md           # This file
```

The `inspect/` and `search-codebase/` directories are local development clones (not used by the Docker build — everything comes from git/npm).

## Adding an extension

1. Add a `RUN` line in `Dockerfile.pi` in the "Install pi extensions" section
2. For npm packages: `RUN pi install npm:<package-name>`
3. For git repos (TypeScript extensions): `RUN git clone <url> /root/.pi/agent/extensions/<name>`
4. For Rust CLI tools: `RUN git clone <url> /tmp/<name> && cargo install --path /tmp/<name> && rm -rf /tmp/<name>`

## Design decisions

- **Everything from remote sources** — the Dockerfile pulls from npm and git, not local files. This keeps the repo small and the image reproducible.
- **Single-stage build** — no multi-stage; the image is the runtime. Acceptable for a dev tool where image size is secondary to simplicity.
- **Rust via rustup** — not the Debian Rust package. Ensures we get the latest stable compiler (needed for `rtk` and `sonde`).
- **pi as ENTRYPOINT** — means `docker run pi-docker` drops into pi directly. Use `--entrypoint` to access `sd` or `bash`.
