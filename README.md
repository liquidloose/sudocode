# sudocode
A high level  abstraction layer over Python that morphs AI tooling into a compiler instead of a chatbot

------------------------------
## 📜 Sudo Code: Project Specification## 1. Core Philosophy

* Structure over Syntax: The directory tree and high-level JSON objects handle the architecture of the program.
* Intent over Mechanism: Humans design the system components and write pure natural language inside empty strings. AI compiles the granular execution mechanics (loops, types, standard library calls) into raw Python.
* Zero Chatbots: The developer writes code asynchronously in Markdown files. The AI acts strictly as a background compiler, not a conversational assistant.

------------------------------
## 2. File System Architecture
The workspace uses a mirrored twin-root folder structure.

├── [Project-Name]-sudo/      # Root 1: Developer writes here (.md files)
│   ├── main.md
│   └── utils/
│       └── helpers.md
│
└── [Project-Name]-py/        # Root 2: Sibling directory compiled by AI (.py files)
    ├── main.py
    └── utils/
        └── helpers.py


* Every .md file in the Sudo root maps exactly to a .py file in the Python root.
* If a new file or directory is created in the Sudo root, the background compiler instantly provisions the matching path in the Python root.

------------------------------
## 3. Editor UI & Custom Environment (VS Code / Cursor Extension)
To make Markdown feel like a native IDE, a custom extension implements specific tokens, colors, and automation.
## Custom Decorative Headers
Standard Markdown themes color all headers equally. The extension injects custom text decorations to color specific block identifiers:

* #function $\rightarrow$ Vibrant Blue
* ##object $\rightarrow$ Bright Orange
* ###loop or ###ifelse $\rightarrow$ Muted Purple

## Slash Commands & Formatter
Typing / opens an autocompletion dropdown. Selecting a command prints the un-nested block template, automatically ran through a JSON beautifier for spacing.
------------------------------
## 4. The Block Specifications (Templates)## /object
Used for defining structural classes. Anything smaller than a method is deferred to plain text strings.

{
  "class_name": "",
  "inherits_from": "",
  "docstring": "",
  "__init__": {
    "args": "",
    "kwargs": "",
    "logic": ""
  },
  "instance_methods": {
    "method_name_1": {
      "args": "",
      "logic": ""
    }
  },
  "class_methods": {},
  "static_methods": {}
}

## /function
Used for standalone global logic.

{
  "function_name": "",
  "arguments": "",
  "returns": "",
  "logic": ""
}

## /ifelse
For conditional routing, allowing the developer to explain conditions as natural stories rather than managing complex nested indentation blocks.

{
  "if_condition": "",
  "then_do": "",
  "elif_condition": "",
  "elif_do": "",
  "else_do": ""
}

## /loop
The ultimate abstraction of data iteration. No index tracking, no array boundaries—pure intent.

{
  "loop_logic": ""
}

Example input: "loop through cars list and get rid of the random floats in there"
------------------------------
## 5. Compiler & Orchestration Rules

   1. The Watcher: A background script monitors the -sudo/ folder for changes to .md files.
   2. Context Passing: When a file changes, the compiler packages the JSON blocks alongside a prompt explaining the structural mapping rules.
   3. Execution Mapping:
   * class_name becomes class ClassName:
      * logic strings are translated by the LLM into safe, semantic Python statements, loops, and operations fitting that block's exact context.
   4. Output: The compiled file is written clean, with no AI conversational fluff, straight to the -py/ sibling directory.
