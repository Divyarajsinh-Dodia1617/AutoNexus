# AutoNexus Guides

Comprehensive documentation for every AutoNexus command.

## Guides

| Guide | Description |
|-------|-------------|
| [Getting Started](getting-started.md) | Install, configure Obsidian MCP, first run walkthrough |
| [/autonexus](autonexus.md) | Core autonomous loop with Obsidian integration |
| [/autonexus:plan](autonexus-plan.md) | Goal → config wizard |
| [/autonexus:predict](autonexus-predict.md) | Multi-persona swarm with Obsidian persona notes |
| [/autonexus:debug](autonexus-debug.md) | Autonomous bug-hunting loop |
| [/autonexus:fix](autonexus-fix.md) | Autonomous error repair |
| [/autonexus:security](autonexus-security.md) | STRIDE + OWASP security audit |
| [/autonexus:ship](autonexus-ship.md) | Universal shipping workflow |
| [/autonexus:scenario](autonexus-scenario.md) | Scenario-driven use case generator |
| [/autonexus:learn](autonexus-learn.md) | Autonomous documentation engine |
| [/autonexus:resume](autonexus-resume.md) | Resume a previous session from Obsidian |
| [/autonexus:status](autonexus-status.md) | Read-only project dashboard |
| [/autonexus:review](autonexus-review.md) | Post-session analysis + lessons learned |
| [/autonexus:chain](autonexus-chain.md) | Workflow composition — chain commands into pipelines |
| [/autonexus:canvas](autonexus-canvas.md) | On-demand Obsidian Canvas generation |
| [/autonexus:hook](autonexus-hook.md) | Event-driven triggers for automated reactions |

## Quick Decision Guide

| I want to... | Use |
|--------------|-----|
| Improve a metric autonomously | `/autonexus` |
| Don't know what metric to use | `/autonexus:plan` |
| Get expert opinions before starting | `/autonexus:predict` |
| Hunt all bugs in a codebase | `/autonexus:debug` |
| Fix all errors (tests, types, lint) | `/autonexus:fix` |
| Run a security audit | `/autonexus:security` |
| Ship a PR / deployment / release | `/autonexus:ship` |
| Explore edge cases for a feature | `/autonexus:scenario` |
| Generate docs for a codebase | `/autonexus:learn` |
| Continue where I left off | `/autonexus:resume` |
| Check project health at a glance | `/autonexus:status` |
| Analyze what happened in a session | `/autonexus:review` |
| Chain multiple commands | `/autonexus:chain predict → agile → ship` |
| Use a pipeline template | `/autonexus:chain --template full-sprint` |
| Visualize architecture | `/autonexus:canvas --type architecture` |
| Auto-react to crashes | `/autonexus:hook --template auto-debug-on-crashes` |
| Manage event triggers | `/autonexus:hook --list` |
| Predict then debug | `/autonexus:predict --chain debug` |
| Debug then auto-fix | `/autonexus:debug --fix` |
| Full pipeline | `/autonexus:chain --template full-sprint` |
