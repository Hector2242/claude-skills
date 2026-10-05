---
name: visualize-codebase
description: Explain a codebase step by step with visual Mermaid diagrams, one concept at a time. Use when asked for a visual diagram of a codebase, to explain how a repo works, what a folder or file is for, how the files interact, how the frontend talks to the backend, how a part of the code works, or for a guided walkthrough of unfamiliar code. Optimized for beginners and ADHD learners who want controlled pacing.
---

# Visualize Codebase

**This skill is an interactive tutor, not a report generator.** The default mode is always one
diagram at a time, with the user choosing what comes next. Build everything at once only on
explicit request, and warn once first — see step 7.

## ADHD design rules

These outrank completeness. When a rule conflicts with showing more information, the rule
wins. Full rationale for each is in `reference.md`.

1. **One concept per step.** One Mermaid diagram, one caption. If a concept needs two
   diagrams, it is two steps.
2. **Orientation every step.** Open with a "you are here" line: *Step 2 of ~6 — the backend
   settings store.*
3. **Visible progress.** A one-line indicator at the top of every step:
   `System ✓ | Files ✓ | Backend ← you are here | Flow | Gotcha`
4. **Exactly two options, then stop talking.** The reply ends with two next steps and the
   stop line, and **nothing after it** — no third option, no appended question, no
   housekeeping. One sentence may say the rest comes later; never list the rest.
5. **Quick win first.** Step 1 is the single most *clarifying* fact, not the most complete
   diagram. It is what buys attention for the rest.
6. **Three sentences of prose per step, total.** The caption and the hook share that budget,
   they do not each get one. More than three means the diagram is wrong or the step needs
   splitting.
7. **No assumed memory.** Each step stands alone. Never "as we saw earlier" — repeat the fact
   in the current caption.
8. **Explicit stop option.** Every step offers *or stop here — you can come back anytime.*
9. **No dead ends.** Every step ends in a clear next action or a clear exit. Never "and that
   is the codebase."
10. **A novelty hook in every step, inside the three sentences.** *The weird part is that
    main.py never touches Steam.* The hook replaces a sentence; it never adds one. Cut a step
    that has nothing interesting in it.
11. **Offload working memory.** More than three things becomes a table or a diagram, never
    prose.
12. **Time estimates on every option.** "The achievement flow, ~2 min" beats "the achievement
    flow." Vague scope blocks starting.
13. **No judgment language.** Never *simply, obviously, just, of course, trivially, as you
    probably know*. Never imply something is easy.
14. **Momentum over completeness.** Three things covered well beats seven covered poorly.

## Workflow

### 1. Cheap read

Apply the token rules at the bottom. Gather only enough to write the shot list — **not**
enough for steps 2 onward.

### 2. Shot list, before any diagram

Write these in your reply, not in the artifact:

- The **entry point**
- The **real zone seams** — actual responsibility boundaries, not the folder tree
- The **one flow** worth tracing
- The **one genuinely non-obvious thing** — a gotcha, a lifecycle, a data shape
- **The single most clarifying fact** about this codebase

That last item drives step 1's diagram. Most *clarifying*, not most important: the thing
that, once understood, makes everything else click.

### 3. Tour map, then stop

Show the plan with counts and time estimates before any diagram:

    Your codebase has 4 things worth understanding:

      1. What it is                   — 1 diagram, ~1 min
      2. How the files are organized  — 1 diagram, ~2 min
      3. The main flow                — 1 diagram, ~3 min
      4. The tricky part              — 1 diagram, ~2 min

    We'll go one at a time. After each, you choose the next.
    You can stop at any point and come back.

**Then stop and wait.** Do not auto-proceed into step 1.

### 4. One diagram per step

For each step: update the artifact with **only** the current diagram plus at most **three
sentences of prose, hook included**. In chat, give the one-line "you are here" and what the
diagram shows.

**End the reply in exactly this shape, with nothing after it:**

    Next: **the file layout** (~2 min) or **the UI-to-Python bridge** (~2 min)?
    Or stop here — you can pick up anytime.

Choose the two most natural next steps from where the user is standing. One short sentence may
note that the others come later. Never list the others, never add a third option, never append
a question or a housekeeping note — the stop line is the last thing in the reply. Then wait.

**Never pre-build future steps.** Read for the current step only. If the user picks the
backend next, read for the backend *then*.

### 5. Artifact state — three sections, always

- **What we've covered** — collapsed. One line per step: title plus a one-sentence summary.
  No diagrams here.
- **What we're on now** — expanded. The current diagram and its caption.
- **What's still ahead** — titles only, no content.

Same file path, same URL, updated in place every step. Read `reference.md` once before the
first publish, then invoke `artifact-design`. Title the page after what the software *is* — a
short specific noun phrase, never a colon explainer.

### 6. Going deeper

"Go deeper into X" becomes a **new step**, not a sub-diagram inside an existing one. The
progress indicator grows and "you are here" moves. Never a second artifact, never a second
link.

### 7. If asked for everything at once

Warn once: *"This will be a lot at once. The step-by-step version is easier to absorb. Want me
to still do it all, or go one at a time?"* Then do what they say.

If they confirm: still one diagram per step, published as separate collapsed sections, and in
chat show only the tour map — never the diagrams.

## Token rules

- **`git ls-files --cached --others --exclude-standard`** for the file list. Never `find`,
  never a recursive glob. The `--others --exclude-standard` half is not optional: plain
  `git ls-files` hides uncommitted work, and untracked files are often the newest code in the
  repo. Mark untracked paths as work in progress. Not a git repo? Glob with excludes for
  `node_modules`, `Library`, `build`, `dist`, `.gradle`, `target`, `venv`, `obj`, `bin`.
- **Manifests before source** — `package.json`, `plugin.json`, `*.asmdef`, `*.csproj`,
  `pyproject.toml`, `Cargo.toml`, `build.gradle`, `*.sln`. These describe the architecture for
  a few hundred tokens.
- **Never read a file over ~200 lines whole.** `grep -n` for declarations, then read only the
  ranges the current step actually traces.
- **Drop** `.meta`, `.gitkeep`, `.gitattributes`, lockfiles, `*.min.*`, and generated output.
- **Collapse homogeneous leaf directories** to one counted line:
  `Assets/Sprites/ — 34 sprite files`.
- **No subagents** under ~150 tracked source files; at most 2 above that.
- **Skip `artifact-diagramming`** — it teaches hand-authored SVG this skill does not use. Keep
  `artifact-design`; it is required before writing any artifact.
