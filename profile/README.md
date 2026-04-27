<h1 align="center">🦦 OpenOtters</h1>

<p align="center">
  <strong>AI agents as OCI artifacts — declare them, build them, ship them, run them.</strong>
</p>

<p align="center">
  <a href="https://github.com/openotters/openotters"><img alt="otters CLI release" src="https://img.shields.io/github/v/release/openotters/openotters?label=otters&sort=semver"></a>
  <a href="https://github.com/openotters/agentfile"><img alt="agentfile release" src="https://img.shields.io/github/v/release/openotters/agentfile?label=agentfile&sort=semver"></a>
  <a href="https://github.com/openotters/runtime"><img alt="runtime release" src="https://img.shields.io/github/v/release/openotters/runtime?label=runtime&sort=semver"></a>
  <a href="https://github.com/openotters/bin"><img alt="bin release" src="https://img.shields.io/github/v/release/openotters/bin?label=bin&sort=semver"></a>
  <img alt="License" src="https://img.shields.io/github/license/openotters/openotters">
</p>

<p align="center">
  <a href="https://github.com/openotters/agentfile">Agentfile</a> ·
  <a href="https://github.com/openotters/openotters">CLI &amp; daemon</a> ·
  <a href="https://github.com/openotters/runtime">Runtime</a> ·
  <a href="https://github.com/openotters/bin">Tools</a>
</p>

---

## Try it

Each line is the whole interaction.

```sh
# Pull a published agent and chat
otters run --name hello ghcr.io/openotters/agents/base:latest && otters chat hello

# Pipe an inline Agentfile and chat
echo 'FROM scratch
RUNTIME ghcr.io/openotters/runtime:latest
MODEL anthropic/claude-haiku-4-5-20251001
NAME dev' | otters run - --name dev && otters chat dev

# Drive a tool-using agent without the chat loop
otters run --name weather ghcr.io/openotters/agents/meteo:latest && echo 'weather in Lyon' | otters prompt weather
```

The third one is the interesting one: `meteo` ships with `wget` + `jq` BIN
tools, so the agent fetches Open-Meteo and parses the JSON itself — the
prompt just kicks off the tool chain, no shell, no glue code.

## What is OpenOtters?

An open-source platform for packaging AI agents as **portable OCI images**. You
write one declarative file — model, prompts, mounted data, allow-listed tools —
and `otters` builds it into an image you can push to any registry and run
anywhere a Linux/macOS host can dial Anthropic, OpenAI, OpenRouter, Ollama, or
any other OpenAI-compatible endpoint.

Agents are artifacts, not applications. No SDK to import. No bespoke deploys.
Just `otters run`.

```dockerfile
FROM scratch

RUNTIME ghcr.io/openotters/runtime:latest
MODEL anthropic/claude-haiku-4-5-20251001
NAME meteo

CONTEXT SOUL <<EOF
You are a weather assistant. Use wget to fetch the Open-Meteo API,
jq to extract fields, and report temperature in °C.
EOF

CONFIG max-tokens=1024
CONFIG max-iterations=10

BIN wget ghcr.io/openotters/tools/wget:latest "Fetch URL content"
BIN jq   ghcr.io/openotters/tools/jq:latest   "Extract fields from JSON"
BIN cat  ghcr.io/openotters/tools/cat:latest  "Read file contents"

ADD cities.json /data/workspace/cities.json "Known city coordinates"
```

```sh
otters image build -t meteo:latest .
otters run meteo:latest
otters chat meteo
```

## Why?

- 📦 **Portable** — push to GHCR, Docker Hub, or any OCI registry; pull and run
  on a different host with one command.
- 🔒 **Secrets stay in env** — API keys never land in `agent.yaml` or `ps`
  output; the daemon injects them as `<PROVIDER>_API_KEY` env on the runtime
  subprocess.
- 🧬 **Composable** — `FROM` an existing agent image to inherit its prompts,
  tools, and mounts; override what you need.
- 🛠️ **48 tool binaries** — `wget`, `jq`, `grep`, `find`, `printenv`, `sh`,
  `jina`, … each shipped as a static multi-arch OCI image.
- 🧠 **Memory built in** — conversation history with automatic compaction;
  resumable across daemon restarts.
- ♻️ **Self-healing** — agents that fail to start retry with exponential
  backoff; fix `~/.otters/providers.yaml` and the daemon picks it up on the
  next attempt without a restart.
- 📋 **Declarative** — one Agentfile describes the whole agent. No code, no
  YAML, no multi-step build script.

## Repositories

| Repository | Description |
|------------|-------------|
| [**agentfile**](https://github.com/openotters/agentfile) | Spec, parser, builder, OCI store, lifecycle library. The single source of truth for the Agentfile format. |
| [**openotters**](https://github.com/openotters/openotters) | The `otters` CLI and `ottersd` daemon — build, run, manage, chat. |
| [**runtime**](https://github.com/openotters/runtime) | Single-agent gRPC runtime that the daemon spawns per agent: model bridge, tool dispatch, memory, streaming. |
| [**bin**](https://github.com/openotters/bin) | 48 static tool binaries packaged as multi-arch OCI images. |

## Quick start

**Install** (Homebrew, macOS / Linux):

```sh
brew install openotters/tap/otters
```

Or build from source:

```sh
go install github.com/openotters/openotters/cmd/otters@latest
go install github.com/openotters/openotters/cmd/ottersd@latest
```

**Configure a provider** (interactive, picks from the live Catwalk catalog):

```sh
otters provider add
```

Or scriptable:

```sh
echo "$ANTHROPIC_API_KEY" | otters provider add --name anthropic
```

**Run the daemon**:

```sh
ottersd serve
```

**Run an agent**:

```sh
# from a local Agentfile
otters run -f Agentfile

# straight from stdin
cat Agentfile | otters run -

# or pull + run from a registry
otters run ghcr.io/openotters/agents/meteo:latest

# then chat
otters chat <agent-name>
```

The full walkthrough lives in the
[**openotters**](https://github.com/openotters/openotters) repo.

## Status

OpenOtters is in **`v1.0.0-alpha`** — usable end-to-end, with provider
hot-reload, env-only credentials, auto-restart, and a 48-tool catalog
already shipping. The Agentfile spec is stabilising; expect minor breaking
changes until `v1.0.0` proper.

## License

MIT — see each repository's `LICENSE.md`.
