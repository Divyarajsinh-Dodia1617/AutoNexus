# Canvas Workflow — /autonexus:canvas

On-demand Obsidian Canvas generation engine. Produces visual `.canvas` files from project data — architecture diagrams, sprint boards, dependency graphs, session timelines, knowledge maps, data flow diagrams, and pipeline visualizations. Each canvas type has its own data gathering strategy, layout algorithm, and color scheme.

**Core idea:** Project knowledge lives in Obsidian notes, git history, and the codebase. Canvas turns that data into visual, navigable diagrams that reveal structure and relationships at a glance. Every node links back to its source — click a component node to open the component's source, click a story node to open the story note.

## Trigger

- User invokes `/autonexus:canvas`
- User says "generate a canvas", "visualize the architecture", "draw a diagram", "create a dependency graph"
- User says "sprint board canvas", "show me a timeline", "knowledge map", "flow diagram"

## PREREQUISITE: Obsidian Required

Canvas generation REQUIRES Obsidian. If `OBSIDIAN_AVAILABLE = false`:
```
PRINT: "Canvas generation requires Obsidian to write .canvas files.
        Run /autonexus:setup to configure Obsidian MCP."
STOP
```

## PREREQUISITE: Interactive Setup (when type is missing)

If `--type` is not provided, use `AskUserQuestion`:

**Single batched call — 2 questions:**

| # | Header | Question | Options |
|---|--------|----------|---------|
| 1 | `Type` | "What kind of canvas do you want to generate?" | "Architecture — component relationships", "Sprint — kanban board visualization", "Dependency — import/dependency graph", "Timeline — session history over time", "Knowledge — map of all project notes", "Flow — data flow diagram", "Pipeline — pipeline visualization" |
| 2 | `Scope` | "What scope?" (context-dependent) | Type-specific options from auto-detection |

## Architecture

```
/autonexus:canvas
  ├── Phase 1: Gather Data (type-specific)
  ├── Phase 2: Calculate Layout (type-specific algorithm)
  ├── Phase 3: Generate Nodes (with colors, sizes, labels)
  ├── Phase 4: Generate Edges (with labels, directions)
  ├── Phase 5: Serialize to Canvas JSON
  └── Phase 6: Write to Obsidian
```

---

## Canvas Types

### 1. Architecture Canvas

**Data source:** Knowledge/Components.md from Obsidian + codebase scan
**Layout:** Hierarchical top-to-bottom
**Output:** `Projects/{ProjectName}/Canvases/architecture.canvas`

#### Phase 1: Gather Data

```
IF Knowledge/Components.md exists in Obsidian:
  Read components, dependencies, risk areas from the note
ELSE:
  Scan codebase:
    - Identify top-level directories as components
    - Parse import statements for dependency relationships
    - Estimate risk from complexity (file count, LOC)
```

#### Phase 2: Layout

```
# Hierarchical layout — entry points at top, leaf dependencies at bottom
# Group by layer: API/UI → Services/Logic → Data/Storage → External

layers = classify_components(components)
  # Layer 0: Entry points (routes, CLI, main)
  # Layer 1: Controllers/handlers
  # Layer 2: Services/business logic
  # Layer 3: Data access/models
  # Layer 4: External dependencies

FOR layer_index, layer in enumerate(layers):
  y = layer_index * 350  # Vertical spacing between layers
  x_start = -(len(layer) * 450) / 2  # Center the layer
  FOR comp_index, component in enumerate(layer):
    component.x = x_start + (comp_index * 450)
    component.y = y
```

#### Phase 3: Nodes

```
FOR each component:
  node = {
    id: "comp-{sanitized_name}",
    type: "text",
    text: "## {component_name}\n**Path:** `{path}`\n**Files:** {count}\n**Risk:** {risk_level}",
    x: component.x,
    y: component.y,
    width: 400,
    height: 200,
    color: risk_color(component.risk)
      # "1" (red) = high risk
      # "4" (green) = low risk
      # "5" (purple) = external dependency
      # "6" (blue) = entry point
  }
```

#### Phase 4: Edges

```
FOR each dependency relationship (source → target):
  edge = {
    id: "dep-{source}-{target}",
    fromNode: "comp-{source}",
    toNode: "comp-{target}",
    fromSide: "bottom",
    toSide: "top",
    label: "{dependency type or import count}"
  }
```

---

### 2. Sprint Canvas

**Data source:** Obsidian sprint board (`Sprints/sprint-{N}/Board.md`) + story notes
**Layout:** Kanban columns left-to-right
**Output:** `Projects/{ProjectName}/Canvases/sprint-{N}-board.canvas`

#### Phase 1: Gather Data

```
Read sprint board from Obsidian:
  obsidian_read_note({
    filePath: "Projects/{ProjectName}/Sprints/sprint-{N}/Board.md"
  })

Parse stories into columns: Backlog, To Do, In Progress, In Review, Done
FOR each story: extract ID, title, points, assignee, status
```

#### Phase 2: Layout

```
columns = ["Backlog", "To Do", "In Progress", "In Review", "Done"]
column_width = 350
column_spacing = 400
header_height = 80

FOR col_index, column in enumerate(columns):
  # Column header node
  header.x = col_index * column_spacing
  header.y = 0

  # Story nodes stacked vertically within column
  FOR story_index, story in enumerate(column.stories):
    story.x = col_index * column_spacing
    story.y = header_height + 20 + (story_index * 180)
```

#### Phase 3: Nodes

```
# Column headers
FOR each column:
  node = {
    id: "col-{name}",
    type: "text",
    text: "# {column_name}\n{story_count} stories | {total_points} pts",
    x: column.x, y: 0,
    width: column_width, height: 60,
    color: column_color(name)
      # "6" (blue) = Backlog/To Do
      # "4" (green) = In Progress
      # "3" (yellow) = In Review
      # "1" (red if blocked) or "4" (green) = Done
  }

# Story cards
FOR each story:
  node = {
    id: "story-{id}",
    type: "file",
    file: "Projects/{ProjectName}/Backlog/stories/STORY-{id}.md",
    x: story.x, y: story.y,
    width: column_width, height: 150,
    color: complexity_color(story.points)
      # "4" (green) = 1-3 pts
      # "3" (yellow) = 5-8 pts
      # "1" (red) = 13+ pts
  }
```

#### Phase 4: Edges

```
# Blocking relationships between stories
FOR each story with blockers:
  FOR each blocker_id in story.blocked_by:
    edge = {
      id: "block-{blocker_id}-{story.id}",
      fromNode: "story-{blocker_id}",
      toNode: "story-{story.id}",
      label: "blocks",
      color: "1"  # Red for blocking
    }
```

---

### 3. Dependency Canvas

**Data source:** Codebase import analysis + Knowledge/Dependencies.md
**Layout:** Hierarchical or force-directed
**Output:** `Projects/{ProjectName}/Canvases/dependencies.canvas`

#### Phase 1: Gather Data

```
Scan codebase for import/require/include statements:
  FOR each source file in scope:
    Extract all imports → build adjacency list
    Classify: internal vs external dependency

IF Knowledge/Dependencies.md exists:
  Supplement with documented dependency info
```

#### Phase 2: Layout

```
# Hierarchical layout based on dependency depth
# Files with no dependents at top, deepest dependencies at bottom

depth = calculate_dependency_depth(adjacency_list)
FOR each file:
  file.y = depth[file] * 300
  file.x = position_within_layer(file, depth[file])  # Spread evenly
```

#### Phase 3: Nodes

```
FOR each file/module:
  node = {
    id: "file-{hash}",
    type: "text",
    text: "**{filename}**\n`{relative_path}`\nImports: {import_count}\nExports: {export_count}",
    x: file.x, y: file.y,
    width: 300, height: 120,
    color: type_color(file)
      # "6" (blue) = source code
      # "4" (green) = test files
      # "3" (yellow) = config files
      # "5" (purple) = external packages
  }
```

#### Phase 4: Edges

```
FOR each import (source → target):
  edge = {
    id: "import-{source_hash}-{target_hash}",
    fromNode: "file-{source_hash}",
    toNode: "file-{target_hash}",
    fromSide: "bottom",
    toSide: "top",
    label: "{symbols imported}"  # Omit if too many
  }
```

---

### 4. Timeline Canvas

**Data source:** Obsidian session summaries + autonexus-results.tsv
**Layout:** Left-to-right timeline
**Output:** `Projects/{ProjectName}/Canvases/timeline.canvas`

#### Phase 1: Gather Data

```
IF OBSIDIAN_AVAILABLE:
  obsidian_list_notes({
    dirPath: "Projects/{ProjectName}/Iterations/",
    recursionDepth: 2
  })
  Read session summary frontmatter for each session:
    date, goal, baseline, final, iterations, keeps, discards

ALSO read autonexus-results.tsv for metric values over time
```

#### Phase 2: Layout

```
# Timeline: X = date, Y = metric value
x_spacing = 500  # Per session
y_scale = calculate_y_scale(min_metric, max_metric, canvas_height=800)

FOR session_index, session in enumerate(sessions):
  session.x = session_index * x_spacing
  session.y = 400  # Center vertically — metric shown as text
```

#### Phase 3: Nodes

```
FOR each session:
  keep_rate = session.keeps / session.iterations * 100
  node = {
    id: "session-{session_id}",
    type: "file",
    file: "Projects/{ProjectName}/Iterations/{date}/{session-id}-summary.md",
    x: session.x, y: session.y,
    width: 400, height: 250,
    color: keep_rate_color(keep_rate)
      # "4" (green) = >50% keep rate
      # "3" (yellow) = 30-50%
      # "1" (red) = <30%
  }
```

#### Phase 4: Edges

```
# Sequential connections between sessions
FOR i in range(1, len(sessions)):
  edge = {
    id: "timeline-{i-1}-{i}",
    fromNode: "session-{sessions[i-1].id}",
    toNode: "session-{sessions[i].id}",
    fromSide: "right",
    toSide: "left",
    label: "{metric_delta}"
  }
```

---

### 5. Knowledge Canvas

**Data source:** All Obsidian notes in the project
**Layout:** Force-directed clusters by note type
**Output:** `Projects/{ProjectName}/Canvases/knowledge-map.canvas`

#### Phase 1: Gather Data

```
obsidian_list_notes({
  dirPath: "Projects/{ProjectName}/",
  recursionDepth: 3
})

FOR each note:
  Classify by type from tags or path:
    - iteration (Iterations/)
    - decision (Decisions/)
    - prediction (Predictions/)
    - knowledge (Knowledge/)
    - sprint (Sprints/)
    - backlog (Backlog/)
    - hook (Hooks/)
    - pipeline (Pipelines/)

  Extract backlinks and explicit [[wiki-links]] from content
  Build link graph
```

#### Phase 2: Layout

```
# Cluster by type — each type gets a region of the canvas
clusters = {
  "knowledge": { center_x: 0, center_y: 0 },
  "decision": { center_x: 800, center_y: 0 },
  "iteration": { center_x: 0, center_y: 800 },
  "prediction": { center_x: 800, center_y: 800 },
  "sprint": { center_x: -800, center_y: 0 },
  "backlog": { center_x: -800, center_y: 800 },
  "hook": { center_x: 400, center_y: -600 },
  "pipeline": { center_x: -400, center_y: -600 }
}

FOR each note:
  cluster = clusters[note.type]
  # Spread within cluster — grid or radial
  offset_x, offset_y = position_in_cluster(note, cluster)
  note.x = cluster.center_x + offset_x
  note.y = cluster.center_y + offset_y
```

#### Phase 3: Nodes

```
FOR each note:
  node = {
    id: "note-{sanitized_path}",
    type: "file",
    file: "{note_path}",
    x: note.x, y: note.y,
    width: 300, height: 150,
    color: type_color(note.type)
      # "6" (blue) = knowledge
      # "5" (purple) = decision
      # "4" (green) = iteration (keep)
      # "1" (red) = iteration (discard/crash)
      # "3" (yellow) = prediction
      # "2" (orange) = sprint/backlog
  }

# Cluster label nodes
FOR each cluster:
  label_node = {
    id: "cluster-{type}",
    type: "text",
    text: "# {Type}\n{count} notes",
    x: cluster.center_x - 150, y: cluster.center_y - 250,
    width: 300, height: 80,
    color: type_color(type)
  }
```

#### Phase 4: Edges

```
FOR each [[wiki-link]] found:
  IF both source and target are in the graph:
    edge = {
      id: "link-{source_hash}-{target_hash}",
      fromNode: "note-{source}",
      toNode: "note-{target}"
    }
```

---

### 6. Flow Canvas

**Data source:** Codebase analysis (entry points, data stores, processing steps)
**Layout:** Left-to-right data flow
**Output:** `Projects/{ProjectName}/Canvases/data-flow.canvas`

#### Phase 1: Gather Data

```
Scan codebase for:
  - Entry points: HTTP routes, CLI handlers, event listeners, main functions
  - Processors: middleware, service functions, transformers
  - Data stores: database calls, file I/O, cache operations
  - Outputs: responses, emitted events, file writes, API calls

Build flow graph: entry → processor → ... → store/output
```

#### Phase 2: Layout

```
# Left-to-right flow in 4 columns
columns = ["Entry Points", "Processing", "Data Stores", "Outputs"]
column_x = [0, 500, 1000, 1500]

FOR each node:
  col = classify_column(node)  # Based on node type
  node.x = column_x[col]
  node.y = position_in_column(node, col) * 200
```

#### Phase 3: Nodes

```
FOR each flow node:
  shape_text = flow_shape(node.type)
    # Entry: ">>> {name} >>>"
    # Process: "[ {name} ]"
    # Store: "(( {name} ))"
    # Output: "<<< {name} <<<"

  node_obj = {
    id: "flow-{hash}",
    type: "text",
    text: "{shape_text}\n`{file}:{line}`\n{description}",
    x: node.x, y: node.y,
    width: 350, height: 150,
    color: type_color(node.type)
      # "6" (blue) = entry
      # "3" (yellow) = processing
      # "5" (purple) = data store
      # "4" (green) = output
  }
```

#### Phase 4: Edges

```
FOR each data flow (source → target):
  edge = {
    id: "flow-{source}-{target}",
    fromNode: "flow-{source_hash}",
    toNode: "flow-{target_hash}",
    fromSide: "right",
    toSide: "left",
    label: "{data type or description}"
  }
```

---

### 7. Pipeline Canvas

**Data source:** Pipeline definition + run logs from Obsidian
**Layout:** Left-to-right sequential + conditional branches
**Output:** `Projects/{ProjectName}/Canvases/pipeline-{name}.canvas`

#### Phase 1: Gather Data

```
IF --pipeline <name>:
  Read pipeline definition from Obsidian
  Read most recent run log for status
ELSE:
  Read all pipelines from Projects/{ProjectName}/Pipelines/
```

#### Phase 2-4: Layout + Nodes + Edges

```
x_spacing = 400

FOR step_index, step in enumerate(pipeline.steps):
  step.x = step_index * x_spacing
  step.y = 0

  # Conditional branches shown below main track
  IF step.condition != "always" AND step.condition != "on-success":
    step.y = 200  # Below main track

node = {
  id: "step-{index}",
  type: "text",
  text: "## Step {index + 1}\n**/autonexus:{step.command}**\n`{step.args}`\nCondition: {step.condition}\nStatus: {step.status or 'pending'}",
  x: step.x, y: step.y,
  width: 350, height: 200,
  color: status_color(step.status)
    # "4" (green) = success
    # "1" (red) = failed
    # "3" (yellow) = running/pending
    # "0" (gray) = skipped
}

# Sequential edges
FOR i in range(1, len(steps)):
  edge = {
    id: "pipe-{i-1}-{i}",
    fromNode: "step-{i-1}",
    toNode: "step-{i}",
    fromSide: "right",
    toSide: "left",
    label: "{condition}"
  }
```

---

## Canvas JSON Serialization

All canvas types output the same JSON structure:

```json
{
  "nodes": [
    {
      "id": "{unique-id}",
      "type": "text",
      "text": "{markdown content}",
      "x": 0, "y": 0,
      "width": 400, "height": 200,
      "color": "4"
    },
    {
      "id": "{unique-id}",
      "type": "file",
      "file": "{obsidian-path}",
      "x": 500, "y": 0,
      "width": 400, "height": 200
    }
  ],
  "edges": [
    {
      "id": "{unique-id}",
      "fromNode": "{node-id}",
      "toNode": "{node-id}",
      "fromSide": "right",
      "toSide": "left",
      "label": "{optional label}"
    }
  ]
}
```

### Color Reference (Obsidian Canvas)

| Code | Color | Used For |
|------|-------|----------|
| `"0"` | No color (gray) | Skipped, inactive |
| `"1"` | Red | High risk, failed, critical, blocking |
| `"2"` | Orange | Sprint/backlog, warning |
| `"3"` | Yellow | In review, medium risk, processing |
| `"4"` | Green | Success, low risk, done, output |
| `"5"` | Purple | External, decision, data store |
| `"6"` | Blue | Entry point, knowledge, source |

---

## Write Protocol

```
canvas_json = JSON.stringify({ nodes: [...], edges: [...] })

obsidian_update_note({
  targetType: "filePath",
  targetIdentifier: "{canvas_path}",
  content: canvas_json,
  modificationType: "wholeFile",
  wholeFileMode: "overwrite",
  createIfNeeded: true
})
```

Uses retry-then-queue protocol. If canvas write fails, print the canvas path and note that it couldn't be written — the data is still available in the source notes.

---

## Output

After writing the canvas, print:

```
Canvas generated: {canvas_path}
  Type: {canvas_type}
  Nodes: {count}
  Edges: {count}
  Open in Obsidian to view the visual diagram.
```

---

## Flags Summary

| Flag | Purpose |
|------|---------|
| `--type <type>` | Canvas type: architecture, sprint, dependency, timeline, knowledge, flow, pipeline |
| `--scope <glob>` | File scope (architecture, dependency, flow) |
| `--sprint <N>` | Sprint number (sprint canvas) |
| `--session <id>` | Session ID (timeline canvas) |
| `--pipeline <name>` | Pipeline name (pipeline canvas) |
| `--output <path>` | Custom Obsidian path for the canvas file |
| `--open` | Print a link for easy Obsidian navigation |

## Error Handling

| Scenario | Response |
|----------|----------|
| Obsidian unavailable | "Canvas generation requires Obsidian." STOP |
| No components found (architecture) | "No components detected. Ensure scope covers source files." |
| No sprint data (sprint) | "Sprint {N} not found. Run /autonexus:agile first." |
| No sessions found (timeline) | "No session data. Run /autonexus first." |
| Empty project in Obsidian (knowledge) | "No notes found. Run some autonexus workflows to populate." |
| Too many nodes (>200) | Warn: "Large canvas ({N} nodes). Obsidian may be slow. Consider narrowing scope." |
| Canvas write fails | Use retry-then-queue. Print: "Canvas couldn't be written to Obsidian. Data is in source notes." |
