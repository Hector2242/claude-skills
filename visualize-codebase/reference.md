# Reference — step recipes, grammar, rules

Read once, before the first publish.

**The rule above all of these: one diagram per step, then stop and let the user choose.** The
step recipes below are a menu to draw from, not a sequence to run. The user picks the order,
and "go deeper" creates another step of whichever type fits.

If a step has no honest answer in this codebase, say so in one line and omit it. Never pad
with an invented diagram.

---

## Step type: What it is  (the system boundary)

**Question it answers:** what is this thing, where does it end, and what crosses that edge?

- `flowchart LR`, **5-9 nodes**.
- The software as 1-3 boxes, plus every external it talks to: the user, the network, disk, a
  database, hardware, an engine, a third-party API.
- Style externals visibly differently from internals. No internal modules at this step.
- If it cannot be grasped in about five seconds, it holds too much.

This is usually, but not always, step 1. Step 1 is whichever diagram carries **the single most
clarifying fact** — sometimes that is the boundary, sometimes it is a surprising flow.

---

## Step type: How the files are organized  (the annotated tree)

**Question it answers:** where is everything, and what is each part for?

A monospace `<pre>` inside an `overflow-x:auto` container. Box-drawing characters, each path
followed by a **3-6 word job**, annotations aligned in a column.

    decky-plugin/
    |-- src/                 the UI and all of the logic
    |   |-- index.tsx        panel, presets, CSS builders
    |   `-- types.d.ts       asset import shims only
    |-- main.py              settings read and write, nothing else
    |-- plugin.json          Decky store manifest
    `-- rollup.config.js     re-export of @decky/rollup

Use real box-drawing glyphs in the output. **Depth 3 at most.**

**The tree carries no arrows.** It answers *where things are*. Interaction is the job of the
flow and gotcha steps. Containment arrows are forbidden — see failure modes.

**Drop outright:**

- Sidecars and markers: `.meta`, `.gitkeep`, `.gitattributes`, `.DS_Store`
- Lockfiles: `package-lock.json`, `pnpm-lock.yaml`, `Cargo.lock`, `poetry.lock`
- Generated: `*.min.*`, `dist/`, `build/`, `obj/`, `bin/`, `Library/`, `.gradle/`

Unity is the extreme case: `.meta` files are 1:1 sidecars and routinely outnumber real files.

**Collapse bulk.** A leaf directory with more than ~5 files of one kind becomes one counted
line: `|-- Assets/Sprites/      34 sprite files`

**Group siblings that form one mechanism.** `EnemyBrain`, `EnemyDecision`, `EnemyMotor`,
`EnemyState` are one subsystem, not four unrelated entries. Keep them adjacent, give each a
one-phrase role.

**Keep any roster short — at most 5 rows, one short line each.** A table of
*path | why it matters* is allowed here for the few paths that matter most, and it sits outside
the 3-sentence prose budget only because each cell is a single line. Two-sentence cells make it
prose again, which breaks rule 6. A full per-file roster is its own "go deeper" step.

---

## Step type: The main flow  (a traced action)

**Question it answers:** what actually happens, step by step, when X occurs?

- `sequenceDiagram`, **6 participants or fewer**.
- The single most representative action. One. Not three.
- Every message names a real symbol, with a verified `path/to/file.ext:line` anchor in the
  caption. Read the file to confirm each line number; a wrong anchor is worse than none.
- Cross-process and cross-language hops are the point of this step. A string-matched RPC, an
  event bus, an engine callback: draw the boundary and label the arrow with the *mechanism*,
  not with "calls".
- Show an error or early-return path when there is a meaningful one.

---

## Step type: The tricky part  (the gotcha)

**Question it answers:** what is the one thing here that is genuinely non-obvious?

- `stateDiagram-v2` or `erDiagram`, whichever actually fits. **8 states** or **6 entities**.
- Candidates: an object lifecycle, a state machine, a retry or timeout rule, an ownership or
  aliasing rule, a cache invalidation rule, a name-matched binding, a migration path.
- This step carries the strongest novelty hook. Name the trap plainly: what breaks, and what
  someone would reasonably assume instead.
- If nothing in this codebase is subtle, write that one line and omit the step.

---

## Entry points by language

| Language | Look for |
|---|---|
| C / C++ | `int main(`, then the loop it enters |
| Java | `public static void main`, `Main.java`, `run()` on the primary class |
| Python | the `__main__` guard, `__main__.py`, console_scripts in `pyproject.toml` |
| JS / TS | `package.json` main / bin / scripts.start; `src/index.*`, `app/`, `pages/` |
| C# | `Program.cs`, `Main(`; in Unity, MonoBehaviour hooks `Awake`, `Start`, `Update` |
| Go | `func main(`, `cmd/*/main.go` |
| Rust | `fn main(`, `src/main.rs`, the bin target in `Cargo.toml` |
| Shell | the executable carrying a shebang |
| Fallback | whatever the README run command or the manifest scripts point at |

**Inverted lifecycles.** Unity, libGDX, Decky, React and most frameworks own the loop and call
*into* the code, so there is no `while` loop to find. Draw the engine or host as an external in
the boundary step and put its callback contract in the gotcha step. Examples:
`MonoBehaviour.Update`, `ApplicationAdapter.render`, a `Plugin` class resolved by method name.

This is worth saying out loud to a beginner. "There is no main function, the engine calls your
code" is often the most clarifying fact in the whole repo.

---

## Diagram grammar

| The reader is asking | Mermaid type |
|---|---|
| What talks to what? | `flowchart` |
| What happens when I do X? | `sequenceDiagram` |
| What states can this be in? | `stateDiagram-v2` |
| What is the shape of the data? | `erDiagram` |

One type per question. A diagram answering two questions answers neither.

**Colour has a job.** Four roles, no more:

- **Neutral** — the normal path, the default node
- **Red or warm** — errors, friction, the trap
- **Blue** — persistent state, anything that outlives the call
- **Dashed edge** — async, deferred or event-driven; never a direct call

Any other use, make it neutral.

**Theme-safe colours.** Mermaid renders to SVG and `classDef` takes literal values — CSS
custom properties do not reach inside it. Set the base theme with a Mermaid init directive and
pick mid-tone fills that read on both a white and a near-black page:

    classDef ext fill:#6d7886,stroke:#4e5865,color:#fff
    classDef store fill:#3d6fa6,stroke:#2b5178,color:#fff
    classDef dead fill:#ad4a50,stroke:#8a3a3f,color:#fff

Page chrome — headings, captions, the tree, borders — uses normal CSS custom properties and
must be theme-aware per `artifact-design`.

### Mechanics

- Mermaid is **native** in Artifacts: a `pre` element with `class="mermaid"`. Never load a
  Mermaid library from a CDN — the CSP blocks it and it is already present.
- **Never collapse a section with a closed `details` element.** Mermaid renders on load, and a
  closed `details` gives its contents no layout box, so a diagram inside can render at zero
  width and stay blank even after it is opened. This matters here because the covered and
  ahead sections are collapsed by default. Use a panel that keeps its width:

      .panel { max-height: 0; overflow: hidden; opacity: 0; }
      .step.open .panel { max-height: none; opacity: 1; }

  Drive it from a `button` with `aria-expanded`. Never `display:none`, never `el.hidden`.
- Every diagram and the tree each get their own `overflow-x: auto` container, so the page body
  never scrolls sideways at phone width.
- Avoid parentheses and unescaped quotes inside node labels — a Mermaid parse error renders as
  a blank box with no visible message.
- Keep node labels to a few words. Sentences belong in the caption.

---

## Budgets

| Thing | Limit |
|---|---|
| Diagrams per step | **1** |
| Prose per step, caption and hook together | **3 sentences** |
| Roster cell | 1 short line |
| Next-step options | **2** with time estimates, then the stop line, then nothing |
| Boundary step nodes | 5-9 |
| Flow step participants | 6 |
| Gotcha step states / entities | 8 / 6 |
| Tree depth | 3 |
| Roster rows inside the tree step | 5 |

None of these is a judgement call. Over budget means cut, collapse, or split into another
step — splitting is free, and in this skill it is the preferred answer.

---

## The removal test

Take each box, then each arrow, one at a time: **if deleting it leaves the diagram meaning the
same, it was decoration — delete it and do not put it back.** Repeat until every element earns
its place. Run this before every publish; it is a gate.

---

## Failure modes to refuse

- **Dumping every diagram at once.** The single worst failure. It turns a tour into the
  reference document this skill exists to replace. One step, then stop and ask.
- **Auto-proceeding.** Building the next step without being asked is the same failure wearing
  a different hat.
- **Containment arrows.** `src/` pointing at `src/index.tsx` is nesting, not a relationship.
  The tree shows nesting; arrows are reserved for real mechanisms.
- **Redrawing the file tree and calling it architecture.** Folders are filing. Name the
  responsibility each path actually holds.
- **One node per class.** Classes are not zones. Group by responsibility.
- **The 30-node complete graph.** Completeness is the opposite of explanation.
- **Prose paragraphs with a diagram bolted on.** If the words carry the meaning, the step
  failed.
- **A legend longer than the diagram.**
- **Anything after the stop line.** A third option, "want me to also…", or a housekeeping
  question re-opens the decision the two-option rule just closed. Save it for the next reply.
- **Listing every remaining step as the choice.** The progress map may show them; the question
  offers two.
- **A path with no job.** A tree line reading only `utils/` is filler. Name it or collapse it.

---

## The ADHD design rules, in full

ADHD is not an attention deficit. It is a difference in working memory, task initiation, and
dopamine regulation. Each rule below answers one of those directly. They outrank
completeness: when a rule conflicts with showing more information, the rule wins.

**1. One concept per step.** One diagram, one caption. Two diagrams means two steps.
*Why:* working memory is the binding constraint. Two diagrams on screen means neither gets
held onto.

**2. Orientation at every step.** Open with *Step 2 of ~6 — the backend settings store.*
*Why:* it pre-empts the "where am I, what is this, why am I reading it" spiral that ends a
session.

**3. Visible progress.** `System ✓ | Files ✓ | Backend ← you are here | Flow | Gotcha`
*Why:* dopamine regulation needs visible gains. Knowing how much is done and how much is left
is itself a reward.

**4. Choice, not a menu — exactly two options, and the stop line is last.**
*Why:* task initiation is the hard part. Two options is a choice; five is decision fatigue, and
a six-item menu produces paralysis rather than engagement. This is why nothing may follow the
stop line: an appended "or shall I also…" is a third option wearing a disguise, and it restores
the decision load the rule just removed. The artifact's "what's still ahead" list is a progress
map, never the choice.

**5. Quick win first.** The first diagram is the most *clarifying* fact, not the most
complete one.
*Why:* the "oh, THAT is what this is" moment is what funds attention for everything after it.
A thorough but unremarkable opener spends that budget and returns nothing.

**6. Three sentences of prose per step, caption and hook together.**
*Why:* prose is where overwhelm enters, and a caption of three plus a hook of two is five — long
enough to undo a one-minute step. The two share one budget. If three sentences cannot carry it,
the diagram is wrong or the step needs splitting, and splitting is free.

**7. No assumed memory.** Never "as we saw earlier" — repeat the fact.
*Why:* steps may be read days apart, or out of order, or after a tab was closed. Every step
must stand on its own.

**8. An explicit stop option, every step.** *Or stop here — you can come back anytime.*
*Why:* rejection sensitivity makes an unfinished task feel like a failure. Naming the exit as
a legitimate choice removes the guilt that otherwise attaches to stopping.

**9. No dead ends.** Every step ends in a clear next action or a clear exit.
*Why:* a step that just stops leaves the reader to generate their own next move, which is the
exact executive-function task that is hardest.

**10. A novelty hook in every step, spent from the three sentences.** *The weird part is that
main.py never touches Steam.*
*Why:* curiosity is the most reliable fuel available, so every step needs one — but as one of
the three sentences, never as a fourth. A boring step should be cut, not padded with a hook.

**11. Offload working memory.** More than three things becomes a table or a diagram.
*Why:* holding five file names in your head to follow one sentence is not possible. Put them
on the screen instead.

**12. Time estimates on every option.** "The achievement flow, ~2 min."
*Why:* time blindness makes unbounded scope a hard blocker. A number is the difference between
starting and not.

**13. No judgment language.** Never *simply, obviously, just, of course, trivially, as you
probably know*.
*Why:* these tell a beginner that their difficulty is a personal failing. Never claim
something is easy — say what it does and let them decide how it felt.

**14. Momentum over completeness.** Three things well beats seven poorly.
*Why:* a finished tour of three subsystems teaches more than an abandoned tour of seven. If a
step has no honest answer, say so in one line and omit it.
