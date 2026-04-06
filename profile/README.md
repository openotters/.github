<h1 align="center">🦦 OpenOtters</h1>

<p align="center">
  <strong>Build, ship, and run autonomous AI agents — like containers, but for intelligence.</strong>
</p>

<p align="center">
  <a href="https://github.com/openotters/agentfile">Agentfile</a> ·
  <a href="https://github.com/openotters/openotters">CLI & Daemon</a> ·
  <a href="https://github.com/openotters/runtime">Runtime</a> ·
  <a href="https://github.com/openotters/bin">Tools</a>
</p>

---

### What is OpenOtters?

OpenOtters is an open-source platform for packaging and running AI agents as OCI artifacts. Define your agent in an
**Agentfile** — its model, tools, personality, and data — build it into a portable image, push it to any registry,
and run it anywhere.

```agentfile
FROM scratch
MODEL anthropic/claude-sonnet-4-20250514
NAME support-bot

CONTEXT SOUL <<EOF
You are a helpful support agent.
EOF

BIN wget ghcr.io/openotters/tools/wget:latest "Fetch URLs"
BIN jq   ghcr.io/openotters/tools/jq:latest   "Process JSON"
```

```sh
openotters build -f Agentfile
openotters run support-bot:latest
openotters chat support-bot
```

### Why?

Most agent frameworks are libraries — you write code, manage dependencies, and deploy custom binaries. OpenOtters
takes a different approach: **agents are artifacts, not applications**.

- 📦 **Portable** — OCI images, any registry, any environment
- 🔒 **Secure by default** — no shell, no exec, only declared tools
- 🧬 **Composable** — inherit from parent agents with `FROM`
- 🧠 **Built-in memory** — conversation history with automatic compaction
- 🛠️ **46 tools** — ready-to-use static binaries (wget, jq, grep, find, …)
- 📋 **Declarative** — one file describes the full agent, no code required

### Repositories

| Repository | Description |
|------------|-------------|
| [**agentfile**](https://github.com/openotters/agentfile) | Agentfile spec, parser, builder, and executor — the core library |
| [**openotters**](https://github.com/openotters/openotters) | CLI and daemon — build, run, and manage agents from your terminal |
| [**runtime**](https://github.com/openotters/runtime) | Single-agent gRPC runtime with tools, memory, and streaming |
| [**bin**](https://github.com/openotters/bin) | 46 tool binaries packaged as multi-arch OCI images |

### Get started

```sh
go install github.com/openotters/cli/cmd/openotters@latest
go install github.com/openotters/cli/cmd/openotters-daemon@latest
```

Head over to the [openotters repo](https://github.com/openotters/openotters) for the full quick start guide.

### License

MIT
