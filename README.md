# 组局旅游大王 (Zuji)

**A visual node-based planner — manage branching tasks with state-machine logic**

---

## What is this

Zuji is a **single-file HTML app** that uses a canvas of nodes and edges to orchestrate tasks with complex dependencies. Its core is not a typical kanban or to-do list, but a **state-machine-driven visual decision engine**: nodes automatically transition between states (actionable → locked → done → eliminated) based on predecessor completion. OR-branches let you pick one option among alternatives and automatically cascade-eliminate the rest.

**Typical use cases:**

- Parallel job hunting across multiple companies (apply → test → interview → offer — accept one, the rest auto-eliminate)
- Comparing alternatives in event planning (pick the Italian restaurant, the Chinese restaurant branch greys out)
- Any scenario where "finish A before B, but A has several possible paths"

---

## Quick Start

### Simplest way: just open it

Zuji is pure frontend, single file. **Zero dependencies, zero build, zero backend.**

1. Download `zuji.html`
2. Open it in any modern browser
3. On the setup screen, click **"Skip for now"** at the bottom to enter local mode
4. Data is automatically saved to `localStorage` — it persists across browser sessions

### Enable cloud sync: connect Supabase

To sync across devices, connect your own Supabase project. Your data stays in your own database — no third-party servers involved.

#### Supabase setup steps

1. Go to [supabase.com](https://supabase.com) → create a new project (or use an existing one)
2. Open **SQL Editor** from the left menu
3. Paste and run the following SQL:

```sql
create table if not exists zuji_data (
  id         text primary key default 'main',
  payload    jsonb not null,
  updated_at timestamptz default now()
);
```

4. Go to **Project Settings → API** and copy two values:
   - **Project URL** (looks like `https://xxxxxxxxxxxx.supabase.co`)
   - **anon key** (a long string starting with `eyJ...`)

5. Back in Zuji, paste the URL and Key on the setup screen → click **"Connect & Start"**

Once connected, the sync dot in the toolbar turns green 🟢. Changes auto-sync to Supabase on every edit. On load, remote data is fetched automatically — if the remote version is newer, Zuji reloads without overwriting your local copy.

---

## Core Logic

### 1. State machine: five visual states, auto-computed

You never manually set a node to "in progress" or "waiting". Zuji **automatically computes** each node's current state from the graph topology and completion status:

| State          | Visual                             | Meaning                                           |
| -------------- | ---------------------------------- | ------------------------------------------------- |
| **Actionable** | Orange left bar                    | All prerequisites met — ready to act              |
| **Locked**     | Blue left bar + 🔒                  | Upstream tasks still incomplete                   |
| **Done**       | Green border + strikethrough       | Checked as complete                               |
| **Eliminated** | Red dashed border + diagonal line  | Ruled out by OR logic                             |
| **Cascade**    | Grey dashed border + diagonal line | Upstream was eliminated, this node is unreachable |

State transitions are real-time: check a node as Done, and every downstream node recalculates instantly.

### 2. OR branching: pick one, eliminate the rest

This is the core mechanism that sets Zuji apart from ordinary task tools.

When multiple outgoing edges from a single node are tagged as the same **OR group**, their target nodes become competing alternatives. Once you mark any one alternative as Done, the others automatically become **Eliminated**, and their entire downstream subtrees become **Cascade-eliminated**.

**Example:** A "Find restaurant" node has two OR edges pointing to "Italian restaurant" and "Chinese restaurant". Check the Italian one as Done:

- Chinese restaurant → Eliminated
- Confirm menu (Chinese) → Cascade
- Pay deposit (Chinese) → Cascade
- Reserve private room → Cascade

**How to set up OR groups:**

- Click any edge → select **"New or-group"** from the popup menu
- For sibling edges from the same parent → click and select **"Join or-group: ogX"** to add to an existing group
- OR edges display an orange **OR** badge on the canvas
- Batch mode: hold `Ctrl` and click multiple edges → a batch menu appears → create or join OR groups in one action

### 3. Unlock rules: ANY vs ALL

When a node has multiple incoming edges, you can configure its unlock rule:

- **ANY** (default): the node unlocks when *any one* predecessor is done — for "just need one path to work out"
- **ALL**: the node unlocks only when *every* predecessor is done — for "all prep must be complete before starting"

Click the **ANY / ALL badge** above the node to toggle. You can also change it via the dropdown in the Detail Panel.

---

## Gantt View

Press `G` or click the **Gantt** button in the toolbar to switch from canvas to Gantt chart. All nodes with dates appear as horizontal bars on a timeline:

- **Bar colors** reflect state-machine status (orange = actionable, blue = locked, green = done, red stripes = eliminated)
- **Drag bar edges** to adjust date ranges; drag the entire bar to shift it
- **Red vertical line** marks today
- **Row order** defaults to start date; drag the `⠿` grip on the left to reorder manually
- `Ctrl+Click` rows to multi-select → batch set dates, mark done, or delete
- **Double-click** a row title to edit the node name inline

---

## Bottom Timeline

The timeline strip below the canvas is interactive and tightly coupled with node data:

- **Green highlighted ranges**: date spans covered by at least one node are highlighted, giving you an instant view of busy periods
- **Orange dots**: mark dates where a node's end date (deadline) falls
- **Click a date**: all nodes whose deadline doesn't match that date dim to 30% opacity, while matching nodes stay fully visible — a quick way to focus on "what needs attention today"
- **Today marker**: today's date number is wrapped in a green circle; clicking the date display in the toolbar scrolls the timeline to today
- **Mini calendar**: click the 📅 button at the top-left of the timeline to expand a month view for quick date navigation

---

## Other Features

**Scenes** — Group nodes into separate scenes (e.g. "Job search" and "Event planning"). Switch between them in the sidebar. The "All" view stacks scenes vertically.

**Tags** — Custom colored tags for categorization. Filter nodes by tag in the sidebar.

**Parent-child nodes** — Any node can have child nodes, forming a hierarchy. Parent nodes can be collapsed and expanded.

**Multi-select** — `Ctrl+Click` to select multiple nodes → a batch action bar appears at the bottom (mark done, set dates, delete). Works in both canvas and Gantt views.

**Copy / Paste** — `Ctrl+C` copies selected nodes (including edges between them), `Ctrl+V` pastes with offset.

**Undo / Redo** — `Ctrl+Z` / `Ctrl+Y`, up to 60 history steps.

**Export** — JSON export/import for full data backup; PNG export for canvas or Gantt chart screenshots.

---

## Keyboard Shortcuts

| Key               | Action                                            |
| ----------------- | ------------------------------------------------- |
| `N`               | New node                                          |
| `G`               | Toggle canvas / Gantt view                        |
| `Space`           | Toggle done on selected node                      |
| `Tab`             | Jump to next node                                 |
| `Del`             | Delete selected node or edge                      |
| `Ctrl+A`          | Select all visible nodes                          |
| `Ctrl+C / V / X`  | Copy / Paste / Cut                                |
| `Ctrl+Z / Y`      | Undo / Redo                                       |
| `Ctrl+Click` edge | Multi-select edges (batch OR group)               |
| `Esc`             | Cancel connection / close panel / clear selection |
| `?`               | Show shortcut hints                               |

---

## License

MIT

---

<p align="center"><sub>Built by <a href="mailto:bkuo4862aire@gmail.com">Galatea</a></sub></p>
