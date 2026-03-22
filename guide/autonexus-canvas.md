# /autonexus:canvas — On-Demand Canvas Generation

Generates Obsidian Canvas files (`.canvas`) from project data. Produces visual, navigable diagrams — architecture maps, sprint boards, dependency graphs, session timelines, knowledge maps, data flow diagrams, and pipeline visualizations.

## Syntax

```
# Interactive type picker
/autonexus:canvas

# Architecture diagram
/autonexus:canvas --type architecture --scope "src/**"

# Sprint board visualization
/autonexus:canvas --type sprint --sprint 3

# Dependency graph
/autonexus:canvas --type dependency --scope "src/**/*.ts"

# Session timeline
/autonexus:canvas --type timeline

# Knowledge map of all project notes
/autonexus:canvas --type knowledge

# Data flow diagram
/autonexus:canvas --type flow --scope "src/api/**"

# Pipeline visualization
/autonexus:canvas --type pipeline --pipeline full-sprint

# Custom output path
/autonexus:canvas --type architecture --output "Projects/MyApp/Canvases/v2-architecture.canvas"
```

## Canvas Types

### Architecture
- **Shows:** System components and their relationships
- **Data from:** Knowledge/Components.md + codebase scan
- **Layout:** Hierarchical top-to-bottom (entry points → services → data → external)
- **Colors:** Risk level (red = high risk, green = low risk, purple = external, blue = entry point)

### Sprint Board
- **Shows:** Kanban board with story cards in columns
- **Data from:** Sprint board + story notes in Obsidian
- **Layout:** Left-to-right kanban columns (Backlog → To Do → In Progress → In Review → Done)
- **Colors:** Story complexity (green = small, yellow = medium, red = large)
- **Edges:** Blocking relationships between stories

### Dependency Graph
- **Shows:** Import/dependency relationships between files
- **Data from:** Codebase import analysis + Knowledge/Dependencies.md
- **Layout:** Hierarchical by dependency depth
- **Colors:** File type (blue = source, green = test, yellow = config, purple = external)

### Timeline
- **Shows:** Session history over time with key stats
- **Data from:** Session summaries in Obsidian + autonexus-results.tsv
- **Layout:** Left-to-right timeline
- **Colors:** Keep rate (green = >50%, yellow = 30-50%, red = <30%)

### Knowledge Map
- **Shows:** All Obsidian project notes with connections
- **Data from:** All notes in `Projects/{ProjectName}/`
- **Layout:** Force-directed clusters by note type
- **Colors:** Note type (blue = knowledge, purple = decision, green = iteration, yellow = prediction)

### Data Flow
- **Shows:** How data moves through the system
- **Data from:** Codebase analysis (routes, handlers, services, stores)
- **Layout:** Left-to-right flow (entry → processing → data stores → outputs)
- **Colors:** Node type (blue = entry, yellow = processing, purple = store, green = output)

### Pipeline
- **Shows:** Pipeline steps with execution status
- **Data from:** Pipeline definition + latest run log
- **Layout:** Left-to-right sequential
- **Colors:** Step status (green = success, red = failed, yellow = pending, gray = skipped)

## Flags

| Flag | Purpose | Applies To |
|------|---------|------------|
| `--type <type>` | Canvas type | All |
| `--scope <glob>` | File scope | architecture, dependency, flow |
| `--sprint <N>` | Sprint number | sprint |
| `--session <id>` | Session ID | timeline |
| `--pipeline <name>` | Pipeline name | pipeline |
| `--output <path>` | Custom Obsidian path | All |
| `--open` | Print link for Obsidian navigation | All |

## Obsidian Integration

**Canvas generation requires Obsidian.** Unlike most autonexus commands, this one cannot run without Obsidian because the output IS an Obsidian Canvas file.

### Default Output Paths

| Type | Path |
|------|------|
| Architecture | `Projects/{ProjectName}/Canvases/architecture.canvas` |
| Sprint | `Projects/{ProjectName}/Canvases/sprint-{N}-board.canvas` |
| Dependency | `Projects/{ProjectName}/Canvases/dependencies.canvas` |
| Timeline | `Projects/{ProjectName}/Canvases/timeline.canvas` |
| Knowledge | `Projects/{ProjectName}/Canvases/knowledge-map.canvas` |
| Flow | `Projects/{ProjectName}/Canvases/data-flow.canvas` |
| Pipeline | `Projects/{ProjectName}/Canvases/pipeline-{name}.canvas` |

## Tips

- **Architecture canvas** is great for onboarding — generate it before a new team member starts.
- **Sprint board canvas** supplements the markdown board with a visual overview. Regenerate it mid-sprint to see progress.
- **Knowledge map** is most useful after several sessions — it shows how notes connect and reveals isolated clusters.
- **Timeline canvas** helps you see improvement trends across sessions.
- **Use `--scope`** to focus dependency and architecture canvases on specific modules instead of the entire codebase.
- **Canvases are overwritten** by default. Use `--output` for versioned snapshots (e.g., `architecture-v2.canvas`).
- **Open the canvas in Obsidian** to navigate visually — click any file node to jump to its source note.
