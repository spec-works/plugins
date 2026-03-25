# plugins

The official SpecWorks plugins collection ✨

Extend the power of your AI coding assistant with skills, MCP servers, and other extensibility tools — all in one place.

## 🔌 What's Inside

- **Skills** — Reusable prompts and workflows for document conversion and specification work
- **MCP Servers** — Model Context Protocol servers that give AI assistants new capabilities

## Available Plugins

| Plugin | Type | Description |
|--------|------|-------------|
| [a2a-ask](plugins/a2a-ask/) | Skill | Interact with remote A2A (Agent-to-Agent) protocol agents |
| [markmyword](plugins/markmyword/) | Skill | Bidirectional Markdown ↔ Word (.docx) conversion |
| [markmydeck](plugins/markmydeck/) | Skill | Markdown → PowerPoint (.pptx) conversion |
| [xregistry-mcp](plugins/xregistry-mcp/) | MCP Server | xRegistry specification discovery and navigation |

## How It Works

This repo is a **publishing destination** — like NuGet or npm. Plugin source code and skill definitions live in their respective part repos. When a part releases a new version, its CI workflow publishes the plugin artifacts here.

```
Part Repo (source of truth)          This Repo (published feed)
├── skills/*/SKILL.md          →     plugins/*/skills/*/SKILL.md
├── .mcp.json (if MCP server)  →     plugins/*/.mcp.json
└── build-and-publish.yml            marketplace.json (updated)
```

## 🤝 Contributing

Plugin content is managed through the source part repos:

- [MarkMyWord](https://github.com/spec-works/MarkMyWord) — Word document conversion skill
- [MarkMyDeck](https://github.com/spec-works/MarkMyDeck) — PowerPoint conversion skill
- [A2A-Ask](https://github.com/spec-works/A2A-Ask) — A2A agent interaction skill
- [xRegistry-MCP-Server](https://github.com/spec-works/xRegistry-MCP-Server) — xRegistry MCP server

To contribute, submit PRs to the appropriate part repo. Changes will be published here automatically on release.

## 📄 License

This project is licensed under the [MIT License](LICENSE).
