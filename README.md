# sudocode
A high level  abstraction layer over Python that morphs AI tooling into a compiler instead of a chatbot

**Sudo Code: Complete Project Specification**  
*(Version 0.2 – Ready for Prototype)*

---

### 1. Core Philosophy

- **Structure over Syntax**: The directory tree + high-level JSON blocks define the architecture.
- **Intent over Mechanism**: Humans design components and write pure natural-language intent. The AI acts strictly as a background structural compiler that turns intent into correct, idiomatic Python.
- **Zero Chatbots**: Development happens asynchronously in Markdown files. The AI never converses; it only compiles.
- **Deterministic Intent**: The same block + same project context should produce stable, high-quality output. Non-determinism is actively constrained.
- **Human remains the Architect**: The developer owns structure, naming, interfaces, and high-level logic. The compiler owns loops, types, standard-library usage, edge cases, and mechanical correctness.

---

### 2. File System Architecture

```
[Project-Name]/
├── [Project-Name]-sudo/          # Source of truth (human-written)
│   ├── main.md
│   ├── utils/
│   │   └── helpers.md
│   ├── .sudo/
│   │   ├── project.json          # Project-level config & index
│   │   ├── style.json            # Active style / system-prompt profile
│   │   └── locks/                # Optional frozen compilations
│   └── ...
│
└── [Project-Name]-py/            # Generated output (AI-written)
    ├── main.py
    ├── utils/
    │   └── helpers.py
    └── ...
```

**Rules**
- Every `.md` file in `-sudo/` maps 1:1 to a `.py` file in `-py/`.
- Creating / renaming / deleting a file or directory in `-sudo/` instantly mirrors the action in `-py/`.
- The `.sudo/` directory is private to the tooling and is never compiled.
- Generated `.py` files are considered ephemeral. They may be committed for convenience, but the `-sudo/` tree is the single source of truth.

---

### 3. Editor Experience (VS Code / Cursor Extension)

#### 3.1 Visual Decorations
- `#function` → Vibrant blue
- `##object` → Bright orange
- `###loop` / `###ifelse` / `###match` → Muted purple
- Nested blocks receive progressive indentation coloring.

#### 3.2 Slash Commands
Typing `/` opens a palette that inserts a prettified JSON template:

- `/object`
- `/function`
- `/method` (convenience inside objects)
- `/ifelse`
- `/loop`
- `/match` (future)
- `/block` (generic nested container)

All templates are automatically run through a JSON beautifier.

#### 3.3 Live Preview & Feedback
- Split view or side panel shows the currently compiled Python for the active file.
- Inline diagnostics appear on blocks that failed compilation or produced warnings.
- Right-click any block → “Regenerate this block”, “Add constraint…”, “Lock this compilation”.

#### 3.4 Diff & History
- Easy visual diff between previous and new compilation of a block.
- Optional local history of compilations per block.

---

### 4. Block Specification (JSON Schemas)

All blocks live inside Markdown files as fenced JSON (or raw JSON objects under the decorative headers).

#### 4.1 Common Fields (available on every block)
```json
{
  "id": "optional-stable-id",
  "doc": "human description",
  "constraints": ["list of extra instructions for the compiler"],
  "types": {},                     // optional type hints / contracts
  "locked": false,                 // if true, never regenerate
  "body": []                       // array of nested blocks OR a single string
}
```

#### 4.2 `/object`
```json
{
  "class_name": "",
  "inherits_from": "",
  "docstring": "",
  "class_attributes": {},
  "__init__": {
    "args": "",
    "kwargs": "",
    "body": []
  },
  "instance_methods": {},
  "class_methods": {},
  "static_methods": {},
  "properties": {}
}
```

#### 4.3 `/function` and `/method`
```json
{
  "name": "",
  "args": "",
  "returns": "",
  "raises": "",
  "body": []
}
```

#### 4.4 `/ifelse`
```json
{
  "if_condition": "",
  "then": [],
  "elifs": [
    { "condition": "", "body": [] }
  ],
  "else": []
}
```

#### 4.5 `/loop`
```json
{
  "intent": "natural language description of what to iterate and why",
  "over": "",                     // optional explicit iterable
  "body": []
}
```

**Nesting Rule**  
Any `"body"` field may contain either:
- A plain string (simple intent), or
- An array of further blocks (`/ifelse`, `/loop`, `/function`, raw statements, etc.)

This gives full granularity while still allowing pure natural language when desired.

---

### 5. Compiler & Orchestration

#### 5.1 The Watcher
A background process (Node or Python) watches the `-sudo/` tree using efficient file-system events.

On change:
1. Parse the affected `.md` file(s) into a structured AST of blocks.
2. Update the project index (`.sudo/project.json`).
3. Determine the minimal set of blocks that need recompilation (respecting locks).
4. Gather context (see 5.3).
5. Invoke the LLM compiler.
6. Write clean Python to the mirrored path in `-py/`.
7. Update diagnostics and preview.

#### 5.2 System Prompt / Style Profiles
- Default profile for the prototype incorporates a strong, opinionated Python style (PEP 8 + modern type hints + Google-style docstrings + explicit error handling).
- Profiles live in `.sudo/style.json` (or referenced remote profiles later).
- Users can later create / share / switch profiles.
- The prompt always includes:
  - The exact block being compiled
  - Its nesting context
  - Relevant project symbols and imports
  - The active style rules
  - Strict instructions: “Output only valid Python. No explanations. No markdown. No conversational text.”

#### 5.3 Context & Inter-file Awareness
The compiler receives:
- The full block tree of the current file
- A project symbol index (classes, functions, constants, type aliases) extracted from all `-sudo/` files
- Explicit imports / dependencies declared in the file
- Optionally the compiled Python of directly imported modules (for signature accuracy)

#### 5.4 Compilation Guarantees
- Output is always syntactically valid Python (post-processed with `ruff` / `black` / `mypy` if configured).
- Naming, import style, and formatting follow the active profile.
- Determinism aids: temperature 0, strong few-shot examples, optional seed, and locking.
- Failed compilations leave the previous `.py` intact and surface a clear diagnostic on the offending block.

#### 5.5 Locking & Pinning
- Any block can be marked `"locked": true`.
- Locked blocks store their last successful compilation hash / content in `.sudo/locks/`.
- Useful for freezing critical or hard-won logic.

---

### 6. Project Configuration (`.sudo/project.json`)

```json
{
  "name": "my-project",
  "python_version": "3.12",
  "style_profile": "default-google-pep8",
  "compiler_model": "gpt-4o / claude-3.5 / local / etc.",
  "auto_format": true,
  "type_checking": "mypy",
  "index": { ... }                 // auto-maintained symbol table
}
```

---

### 7. Feedback Loop & Developer Experience

- Side-by-side or inline preview of generated Python.
- One-click regenerate (single block or whole file).
- “Add constraint” action that appends to the block’s `"constraints"` array and recompiles.
- Clear error messages mapped back to the originating block.
- Optional “Explain this compilation” (secondary, non-blocking call) for learning.

---

### 8. Future Evolution Paths (Post-Prototype)

- Bidirectional sync / decompiler (Python → Sudo blocks) for brown-field adoption.
- Richer control-flow blocks (`/match`, `/try`, `/with`, `/async`).
- First-class interface / protocol blocks.
- Multi-language targets (TypeScript, Rust, Go…) by swapping the compiler backend.
- Shared style-profile marketplace.
- Official language server + debugger integration that steps through Sudo blocks.
- Versioned block schemas with automatic migration.

---

### 9. Prototype Implementation Order (Recommended)

1. File watcher + 1:1 mirroring
2. Basic Markdown → JSON block parser
3. Minimal system prompt + single-file compilation (no inter-file context yet)
4. VS Code/Cursor extension with slash commands + colored headers
5. Side-by-side preview pane
6. Project index + inter-file context
7. Locking, constraints, and regenerate actions
8. Style profiles + post-processing (ruff/black)

---

This specification is now complete enough to begin vibe-coding the prototype immediately. It incorporates nested granularity, strong system-prompt direction, inter-file awareness, determinism controls, locking, rich feedback, and clear extension points.

Would you like me to also produce:
- The exact default system prompt skeleton?
- The TypeScript/JSON schemas for the blocks?
- A minimal watcher implementation sketch?
- Or the initial VS Code extension contribution points?

Just say the word and we can drop straight into implementation artifacts.
