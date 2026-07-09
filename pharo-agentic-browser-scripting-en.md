---
marp: true
theme: classmethod
paginate: true
---

<style>
/* Section divider slides: larger heading */
section.section h2 {
  font-size: 56px;
}
</style>

<!-- _class: title -->
<!-- _paginate: false -->

<style scoped>
section {
  justify-content: center;
  align-items: center;
  gap: 12px;
  text-align: center;
}
section > h1:first-child {
  position: static !important;
  width: auto !important;
  height: auto !important;
  padding-left: 0 !important;
  font-size: 72px;
}
section > h1:first-child::after {
  display: none !important;
}
</style>

# pharo-agentic-browser Scripting API

### **Headless Multi-Agent Orchestration, Driven by Code**
Masashi Umezawa
https://github.com/mumez/pharo-agentic-browser

---

<!-- _class: section -->
<!-- _paginate: false -->

<style>
.highlight-box {
  margin-top: 32px;
  background-color: #e8f0fe;
  border-left: 6px solid var(--blue-very-deep);
  padding: 24px 32px;
  border-radius: 0 8px 8px 0;
  font-size: 30px;
}
</style>

## What is the Scripting API?

---

<!-- _class: content-image -->

# Overview

<div class="highlight-box">
An optional package that lets you <strong>orchestrate AgenticBrowser topics from code</strong> — no UI interaction required.
</div>

- Write a small Smalltalk script; AgenticBrowser creates topics, runs the agents, routes results between them, and blocks until everything completes
- A third interface alongside the Spec UI and Web UI

---

# Use Cases

- **Routine AI workflows** — run the same multi-step workflow daily, or wire it into CI
- **Headless environments** — build and run topics with no display at all
- **Complex multi-agent coordination** — concurrent agents with result routing that's awkward to set up by hand in a UI
- **AI-authored orchestration** — the DSL is simple enough for a coding agent to write and run it itself via `st-eval`

---

<!-- _class: section -->
<!-- _paginate: false -->

## Features

---

# A Simple DSL

- Just a few builder messages: `seq:`, `para:`, `topicBy:`, `agentBy:`
- Easy enough for a human **or an AI agent** to write and run with a single `eval`
- Results flow automatically between steps — no manual wiring

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t | t prompt: 'Write a one-sentence summary of Pharo.' ]
    } agentBy: [ :a | a claude ] ].
```

---

# Orchestration Groups

- Coordinate multiple **whole orchestrations**, not just topics within one
- Nest `seq:` / `para:` of orchestrations — and nest groups inside groups
- Enables patterns like **Arena** (same task, different agents, pick the best) or fan-out research merged into one synthesis step

---

# Persistence with Fuel

- Orchestrations (and groups) can be **saved and loaded** via Pharo's Fuel serializer
- Save before running to reuse a built configuration later
- Save after running to keep recorded results, then inspect or `resume` from a loaded copy

---

<!-- _class: section -->
<!-- _paginate: false -->

## Installation

---

# Installation

Load the Scripting package alongside the core package:

```smalltalk
Metacello new
    baseline: 'AgenticBrowser';
    repository: 'github://mumez/pharo-agentic-browser:main/src';
    load: 'Scripting'.
```

Optionally, also load the test suite:

```smalltalk
Metacello new
    baseline: 'AgenticBrowser';
    repository: 'github://mumez/pharo-agentic-browser:main/src';
    load: 'Scripting-Tests'.
```

---

<!-- _class: section -->
<!-- _paginate: false -->

## Script Examples

---

# Single Topic

`runBy:` builds and immediately runs the orchestration, blocking until done:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t |
            t title: 'List Pharo features'.
            t prompt: 'List 3 Pharo Smalltalk features in one sentence each.' ]
    } agentBy: [ :a | a claude ] ].
```

`scriptBy:` builds without running, for later inspection or a `forkRun`.

---

# Sequential Steps — `seq:`

Each topic's result flows into the next topic's prompt:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder seq: {
        builder topicBy: [ :t |
            t prompt: 'List 3 Pharo Smalltalk features in one sentence each.' ].
        builder topicBy: [ :t |
            t prompt: 'Summarize the feature list from the previous step in one sentence.' ]
    } agentBy: [ :a | a claude ] ].
```

---

# Parallel Steps — `para:`

Topics run concurrently; results are combined for the next step:

```smalltalk
AgenticBrowser runBy: [ :builder |
    builder para: {
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk language features.' ].
        builder topicBy: [ :t | t prompt: 'List 3 Pharo Smalltalk development tools.' ]
    } agentBy: [ :a | a claude ] ].
```

`seq:` and `para:` steps can be mixed freely — e.g. parallel research → sequential synthesis.

---

# Orchestration Groups — Arena Pattern

Run two candidates with different agents, then pick the winner:

```smalltalk
AgenticBrowser groupRunBy: [ :groupBuilder |
    groupBuilder para: {
        groupBuilder orchestrationBy: [ :b |
            b seq: { b topicBy: [ :t | t prompt: 'Implement feature XXX.'. t goal: 'all tests pass' ] }
            agentBy: [ :a | a claude ] ].
        groupBuilder orchestrationBy: [ :b |
            b seq: { b topicBy: [ :t | t prompt: 'Implement feature XXX.'. t goal: 'all tests pass' ] }
            agentBy: [ :a | a codex ] ]
    }.
    groupBuilder singleTopicBy: [ :t | t prompt: 'Pick the best implementation above and open a PR.' ]
    agentBy: [ :a | a kilo ]
].
```

---

<!-- _class: section -->
<!-- _paginate: false -->

## Execution Variations

---

# Synchronous & Background Execution

**Synchronous** — `runBy:` / `script run` blocks until everything completes.

**Background** — for long-running orchestrations:

```smalltalk
script forkRunThen: [ :orc | Transcript crShow: 'Done: ' , orc result ].
"... later, if needed:"
script isRunning.
```

`forkRun` is a shorthand for `forkRunThen: [ :orc | ]`.

<div class="highlight-box">
While a background orchestration runs, open the <strong>Spec UI</strong> or <strong>Web UI</strong> to watch it live — think messages, permission confirmations, and topic progress all show up there.
</div>

---

# Cancel & Resume

**Cancel** a running orchestration:

```smalltalk
script terminate.
```

**Resume** after a step times out (continues from the first incomplete step, reusing recorded results):

```smalltalk
script resume.
```

---

<!-- _class: section -->
<!-- _paginate: false -->

## Save & Load

---

# Save & Load

Persist an orchestration (or group) via Fuel — before or after running:

```smalltalk
script save.                                  "-> <sharedDirectoryPath>/<name>.fuel"
script saveTo: '/path/to/my-script.fuel' asFileReference.

loaded := AbTopicOrchestration loadFrom: '/path/to/my-script.fuel'.
loaded result.                                 "results from the saved run"
loaded resume.                                 "continue if the saved run stopped early"
```

<div class="highlight-box">
A saved orchestration can also be loaded straight into a new group with <code>orchestrationLoadFrom:</code>.
</div>

---

<!-- _class: section -->
<!-- _paginate: false -->

## Settings

---

# Timeout Settings

| Setting | Default | Scope |
|---------|---------|-------|
| `orchestrationStepWaitTimeoutSeconds` | 900 s | Per topic step |
| `orchestrationGroupItemWaitTimeoutSeconds` | 900 s | Per group item running in `para:` |

Adjust globally or per orchestration/group:

```smalltalk
script settings orchestrationStepWaitTimeoutSeconds: 1800.
```

---

# Keeping Topics for Review

| Setting | Default | Effect |
|---------|---------|--------|
| `lingerOrchestrationTopicsAfterRun` | `false` | Whether topics stay in the Topic Manager after the orchestration finishes |

- `false` — topics are removed from the Topic Manager once the run completes
- `true` — topics remain, so you can review the whole run's progress from the UI afterward

```smalltalk
script settings lingerOrchestrationTopicsAfterRun: true.
```

<div class="highlight-box">
Set it to <code>true</code> when you want to go back through the Spec UI or Web UI later and retrace how the result was reached.
</div>

---

<!-- _class: section -->
<!-- _paginate: false -->

## Summary

---

# Summary

The **Scripting API** brings headless, code-driven orchestration to AgenticBrowser:

- **Simple DSL** — `seq:` / `para:` / `topicBy:` / `agentBy:`, easy for humans and AI agents alike
- **Orchestration Groups** — compose whole workflows in parallel, in sequence, or nested
- **Result routing** — automatic, no manual wiring between steps
- **Flexible execution** — synchronous or background, with cancel and resume
- **Fuel persistence** — save and reload orchestrations and their results

---

<!-- _class: all-text-center align-center -->
<!-- _paginate: false -->

# **Feedback and contributions are welcome!**

https://github.com/mumez/pharo-agentic-browser
