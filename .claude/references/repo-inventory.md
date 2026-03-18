# Repository Inventory

All repositories in `C:\Github\` with their characteristic tags for practice applicability matching.

## Inventory

| Repo | Tech Stack | Framework | Claude Config | Tests | Docker | Monorepo | CI | Purpose |
|------|-----------|-----------|:---:|:---:|:---:|:---:|:---:|---------|
| 60k-mono | TypeScript, Node 24.x, React, Vite | React Router 7, Fastify 5.8, tRPC 11 | Y | Y | Y | Y | Y | TruthWatch monitoring/logging/education platform |
| AskThem | TypeScript, React Native, Expo | Expo SDK 54, Expo Router v6 | Y | N | N | N | N | Parent-child Q&A mobile app with audio |
| bp-website | TypeScript, React 19, Node 24.x | Next.js 16, Chakra UI v3 | Y | Y | N | N | Y | BoardPandas marketing website |
| CCHytaleModding | Java 21, Gradle | Hytale server plugins | Y | N | Y | N | N | Hytale server plugin development research |
| CC-Notifier | TypeScript, Node.js | VS Code Extension (esbuild) | Y | Y | N | N | N | VS Code toast notifications for Claude Code |
| claude-code-bootstrap | Markdown, Templates | Claude Code starter template | Y | N | N | N | N | Reusable .claude/ config template |
| Cleaning-Planner | TypeScript, React, Node 24.x, PostgreSQL | Next.js 16, Express, Prisma | Y | N | Y | N | N | ADHD-friendly cleaning planner |
| DeafDirectionalHelper | Unknown | Unknown | N | N | N | N | N | Accessibility directional tool |
| GameDecider | TypeScript, React 19 | Next.js | Y | N | N | N | N | Steam AI-powered game selector |
| Janki | TypeScript, Svelte 5, Rust, SQLite | Tauri 2.x desktop | Y | Y | N | N | N | Japanese learning desktop app (FSRS-6) |
| LL-G | Markdown, YAML | Knowledge base | Y | N | N | N | N | Lessons Learned & Gotchas KB |
| Marketing-Learning | Bootstrap | Bootstrap template | Y | N | N | N | N | Learning/marketing starter template |
| MCP | TypeScript, Node.js | npm workspaces | Y | Y | Y | Y | Y | 3 MCP servers: Zendesk, NinjaOne, Northflank |
| O365 Claude Scripts | PowerShell | Microsoft Graph | N | N | N | N | N | Office 365 Graph API automation |
| pixel-agents-future | TypeScript, React 19, Go, Wails v2 | VS Code Extension + React webview | Y | Y | N | N | Y | Pixel art agents VS Code extension |
| PlexPlaylist | Python 3.11+, FastAPI | FastAPI + python-plexapi | Y | N | N | N | N | Plex smart playlist generator |
| Project-Gitgud | Markdown, Templates | Game dev starter template | Y | N | N | N | N | Game development project template |
| relocation | Vue 3, TypeScript, Node.js | Vue 3 SPA, Cloudflare Pages, Supabase | Y | N | N | N | Y | Relocation dashboard |
| Shadow-Arena | GDScript, Godot 4.6.1 | Godot 4.6.1 | Y | N | N | N | N | Top-down survival shooter (pre-dev) |
| supportforge-platform | TypeScript, React 19, Node 24.x, Go, PostgreSQL | Next.js 16, Go/Wails desktop agents | Y | Y | Y | N | Y | AI-powered IT helpdesk platform |
| Tasker-Knowledge-Repo | Markdown | Knowledge base | Y | N | N | N | N | Tasker Android automation KB |
| tech-assistant | PowerShell, Bash | PowerShell / Microsoft Graph | Y | N | N | N | N | Help desk support toolkit |
| TMNT-SF | TypeScript | Web project | N | N | N | N | N | TMNT-related project |
| urban-robot | Unknown | Unknown | N | N | N | N | N | Unknown purpose |
| wellforce-design-system | TypeScript, CSS | Design system | N | N | N | N | N | Shared design system |
| WorkingDashWF | Unknown | Unknown | N | N | N | N | N | Dashboard/workspace tool |
| Worldbuilder | Unknown | Unknown | N | N | N | N | N | World-building tool |
| Zendesk-MCP | TypeScript, Node.js | Node.js MCP server | Y | Y | N | N | Y | Zendesk MCP server (42 tools) |

## Tag Groups

### By Tech Stack
- **TypeScript/Node.js**: 60k-mono, AskThem, bp-website, CC-Notifier, Cleaning-Planner, GameDecider, Janki, MCP, pixel-agents-future, relocation, supportforge-platform, Zendesk-MCP
- **PowerShell**: tech-assistant, O365 Claude Scripts
- **Python**: PlexPlaylist
- **GDScript/Godot**: Shadow-Arena
- **Java**: CCHytaleModding
- **Go**: pixel-agents-future, supportforge-platform
- **Rust**: Janki
- **Markdown/KB**: LL-G, Tasker-Knowledge-Repo, claude-code-bootstrap, Project-Gitgud, Marketing-Learning, BP

### By Framework
- **Next.js**: bp-website, Cleaning-Planner, GameDecider, supportforge-platform
- **React Router**: 60k-mono
- **Vue**: relocation
- **Svelte/Tauri**: Janki
- **Expo/React Native**: AskThem
- **VS Code Extension**: CC-Notifier, pixel-agents-future
- **Godot**: Shadow-Arena

### By Characteristic
- **Has Docker**: 60k-mono, CCHytaleModding, Cleaning-Planner, MCP, supportforge-platform
- **Is Monorepo**: 60k-mono, MCP
- **Has Tests**: 60k-mono, bp-website, CC-Notifier, Janki, MCP, pixel-agents-future, supportforge-platform, Zendesk-MCP
- **Has CI**: 60k-mono, bp-website, MCP, pixel-agents-future, relocation, supportforge-platform, Zendesk-MCP
- **Has Claude Config**: 20 of 28 repos
- **Missing Claude Config**: DeafDirectionalHelper, O365 Claude Scripts, TMNT-SF, urban-robot, wellforce-design-system, WorkingDashWF, Worldbuilder
