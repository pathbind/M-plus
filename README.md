# M+

**One model owns the work. Everyone else tries to break it.**

M+ is a free, source-available multi-model AI work-and-review system originally developed for internal use at Pathbind Games.

It is built around a deliberately asymmetric idea:

- one **Main** model is the sole author and owner of the work
- independent **Reviewers** do not collaborate with the Main
- Reviewers are instructed to challenge, falsify, question and outperform the Main
- Reviewers do not vote on the answer
- Reviewers do not merge their answers into a committee result
- Reviewers do not directly edit the final work
- the Main must assess every criticism, answer material challenges, accept or reject findings with reasons, and revise its own work
- the process repeats until the available reviewers pass unconditionally, the iteration limit is reached, or the run is stopped

M+ is therefore not a "many models answer, then choose the majority" system. It is closer to **single-owner work under hostile peer review**.

The Main is the absolute ruler of the work, but not an unaccountable one. It owns every decision and every file change. The Reviewers exist specifically to make that ownership difficult: they look for unsupported claims, missed requirements, regressions, edge cases, weak assumptions and better alternatives. A strong objection from one Reviewer matters even if every other Reviewer is satisfied.

M+ runs as a single browser file. There is no mandatory account, subscription, server, CLI, build process or Pathbind-hosted AI proxy for normal local use. You bring your own model API keys and pay the model providers directly.

> **Important:** M+ itself is free. AI API usage is charged separately by the model providers you choose.

---

## Contents

1. [Why M+ exists](#why-m-exists)
2. [The core architecture](#the-core-architecture)
3. [How M+ differs from other multi-model approaches](#how-m-differs-from-other-multi-model-approaches)
4. [What happens during a run](#what-happens-during-a-run)
5. [Getting started](#getting-started)
6. [Projects and chats](#projects-and-chats)
7. [The main interface](#the-main-interface)
8. [Settings - APIs](#settings---apis)
9. [API-key security](#api-key-security)
10. [Supported providers](#supported-providers)
11. [Custom / BYOK providers](#custom--byok-providers)
12. [Settings - Roles](#settings---roles)
13. [Review levels](#review-levels)
14. [Reviewer behaviour and pass rules](#reviewer-behaviour-and-pass-rules)
15. [Project tools](#project-tools)
16. [Requirements](#requirements)
17. [Checkpoints](#checkpoints)
18. [Verification](#verification)
19. [Web research and evidence](#web-research-and-evidence)
20. [Budget guardrails](#budget-guardrails)
21. [Files and workspaces](#files-and-workspaces)
22. [File editing](#file-editing)
23. [Downloadable text and code files](#downloadable-text-and-code-files)
24. [Image generation and visual review](#image-generation-and-visual-review)
25. [Usage and estimated cost](#usage-and-estimated-cost)
26. [Retry, stop and recovery](#retry-stop-and-recovery)
27. [Export](#export)
28. [Storage](#storage)
29. [Shared workspace mode](#shared-workspace-mode)
30. [M+ Sync API v1](#m-sync-api-v1)
31. [GitHub and update checks](#github-and-update-checks)
32. [Security and privacy model](#security-and-privacy-model)
33. [Current implementation limits](#current-implementation-limits)
34. [Licence](#licence)
35. [Pathbind Games and Gydel](#pathbind-games-and-gydel)
36. [External landscape references](#external-landscape-references)

---

# Why M+ exists

A single strong model can do excellent work, but it can also be confidently wrong, miss a requirement, misunderstand a file, overlook a regression, accept its own assumptions too easily, or produce a superficially plausible result without testing the alternatives.

Using several models can help, but the obvious multi-model design has a weakness of its own: **the models are often treated as peers in a committee**.

That can take several forms:

- ask several models the same question and let the user choose
- let several agents collaborate on a shared answer
- assign specialists to pieces of a problem
- ask several models to propose answers and then aggregate them
- let agents debate until they converge
- use majority voting or an LLM judge to choose a winner
- use a supervisor to route work among multiple agents

All of these patterns can be useful. M+ deliberately chooses a different one.

M+ starts from the belief that many real tasks benefit from **clear ownership**. Someone should own the implementation, the argument, the document, the decision trail and the final deliverable. Review should create pressure on that owner, not dissolve ownership into a group.

So M+ separates the system into two fundamentally unequal roles.

## The Main

The Main owns the task.

It:

- receives the user's request
- receives the relevant conversation context
- receives acceptance criteria
- inspects files when files exist
- conducts or consumes research when web research is enabled
- creates the first candidate
- makes file changes
- answers Reviewer challenges
- evaluates counterproposals
- accepts, partially accepts or rejects findings
- revises the work
- owns the final user-facing result

There is always one Main.

## The Reviewers

Reviewers do not own the task.

They:

- inspect what the Main actually produced
- challenge the Main's assumptions
- look for evidence gaps
- test acceptance criteria
- inspect changed files
- test alternatives and boundary conditions
- identify regressions
- ask challenge questions
- propose stronger alternatives
- refuse to pass work that still has a material weakness

There can be up to twelve Reviewer slots.

The same model may occupy more than one Reviewer slot. Duplicate Reviewers run independently.

## The asymmetry is intentional

The Main is not one vote among many.

A Reviewer cannot directly replace the Main's work with its preferred version. It can only attack, question and propose. The Main must then respond visibly and decide what to do.

This produces a decision trail:

1. Reviewer finding
2. Main response
3. Main disposition: **accept**, **partial**, or **reject**
4. reason
5. revised work
6. another review pass

The Main remains responsible even when it adopts a Reviewer's better idea.

---

# The core architecture

A normal reviewed run looks conceptually like this:

```text
                     USER
                      |
                      v
                +-----------+
                |   MAIN    |
                | owns work |
                +-----------+
                      |
                 first candidate
                      |
                      v
        +-------------+-------------+
        |             |             |
        v             v             v
   REVIEWER 1    REVIEWER 2    REVIEWER N
   challenges    challenges    challenges
   independently independently independently
        |             |             |
        +-------------+-------------+
                      |
          findings / questions /
             counterproposals
                      |
                      v
                +-----------+
                |   MAIN    |
                | decides   |
                +-----------+
                      |
          accept / partial / reject
                      |
                revised work
                      |
                      v
              review again
```

There is no majority-vote stage.

There is no "average answer".

There is no automatic merge of several competing answers.

There is no requirement for the Main to agree with the Reviewers.

There **is** a requirement for the Main to address them.

---

# How M+ differs from other multi-model approaches

M+ is not claiming that every other multi-agent system uses voting, nor that asymmetric agent roles have never existed elsewhere. The difference is that **single-owner adversarial review is the default product architecture of M+**, not an optional workflow assembled from a general-purpose agent framework.

Several common patterns illustrate the contrast.

## 1. Side-by-side model comparison

Products such as ChatHub send one prompt to multiple chatbots and present the answers together so the user can compare them.

That is useful when the user's goal is:

> "Show me what several models think."

M+ asks a different question:

> "How do I make one model's work survive serious criticism before it reaches me?"

In M+ the user does not receive five independent final answers and have to become the human aggregator. The Main owns one evolving result.

## 2. Collaborative multi-agent crews

Frameworks such as CrewAI explicitly describe collaborative agents, crews and flows. General-purpose multi-agent frameworks allow specialists to cooperate, delegate, use tools and compose larger workflows.

That is useful when a problem decomposes naturally into specialist jobs.

M+ instead makes Reviewers structurally adversarial. They are not assistants to the Main. They are not given sub-tasks by the Main. They exist to find reasons the Main's result should not pass.

## 3. Supervisor / worker architectures

Current LangChain multi-agent documentation describes supervisor patterns in which a central agent coordinates specialised subagents.

M+ is not primarily a routing architecture. The Main does not send tasks to specialist Reviewers and combine their work. The Reviewers independently inspect the Main's candidate after the Main has committed to it.

## 4. Mixture-of-Agents and aggregation

Mixture-of-Agents research commonly uses multiple proposer models followed by an aggregator that synthesises their outputs.

M+ deliberately does not start with a pool of equal proposals.

There is one proposal owner. Other models are critics.

## 5. Multi-agent debate and voting

Multi-agent debate research includes systems in which agents exchange opinions, refine judgments, aggregate perspectives or use majority voting.

M+ avoids a basic danger of that metaphor: a correct minority objection should not disappear because several other models missed it.

In M+:

- one Reviewer can force another iteration
- "PASS" is unconditional
- a Reviewer with an issue cannot hide it behind a positive overall score
- the Main must respond to each material issue rather than merely win a vote

This is the most important conceptual difference in M+.

> **M+ is not a digital democracy. It is a dictatorship under hostile audit.**

The Main is sovereign over the work. The Reviewers have no governing power, but they have the power to make unresolved defects impossible to ignore.

---

# What happens during a run

The exact path depends on whether the task is normal text work, file/workspace work, or image generation, but the overall sequence is consistent.

## 1. M+ establishes requirements

Each chat can have binding acceptance criteria.

If no criteria exist yet, M+ derives a compact checklist from the request. For more complex requests, workspace tasks, long prompts or evidence-sensitive tasks, the Main model may be used to derive them. Simpler requests can use a lightweight heuristic checklist.

Criteria are stored per chat.

They can be reviewed and edited before later runs.

## 2. Optional evidence is prepared

Depending on Project > Evidence:

- web research may run
- pinned evidence URLs may be refreshed
- Evidence Mode may require explicit citations

The same web research dossier is made available to the Main and all Reviewers. This matters because Reviewers should not attack the Main using a completely different hidden evidence base.

## 3. The Main creates the work

For text tasks, the Main writes a candidate.

For file tasks, the Main first inspects the actual workspace and returns concrete file operations.

For image tasks, the Main creates the image brief and M+ uses the image-generation adapter.

## 4. Deterministic validation runs

Where applicable, M+ performs non-LLM checks such as:

- archive integrity
- Office-package structure
- JSON parsing
- XML parsing
- HTML local-reference checks
- evidence citation resolution

A deterministic validator failure is not a suggestion. It becomes a binding problem that the Main must address.

## 5. Reviewers run independently

Configured Reviewers launch in parallel.

They do not see one another's first-pass findings.

Each Reviewer receives:

- the user request
- the Main candidate
- the relevant acceptance criteria
- relevant evidence
- the current workspace when files are involved

For workspace changes, a Reviewer must directly inspect every materially changed file before it can finish.

## 6. Each Reviewer either passes or challenges

A Reviewer can return:

- a verdict
- acceptance-criterion results
- issues with severity
- evidence for each issue
- suggested corrections
- challenge questions
- a counterproposal
- an overall message to the Main

A Reviewer is not allowed to use "PASS with notes" as an escape hatch.

## 7. The Main responds

If any available Reviewer requires changes, or a deterministic check fails, the Main receives the review digest.

The Main must:

- answer material challenge questions
- respond to counterproposals
- disposition findings
- make corrections
- preserve what was already correct

For file work, accepted findings should lead to concrete file changes where appropriate.

## 8. Review repeats

M+ continues until:

- all available Reviewers pass unconditionally and validators pass
- the maximum iteration count is reached
- the user stops the run
- a fatal provider/storage/protocol error prevents continuation

M+ does not silently relabel an iteration-limit result as "passed".

---

# Getting started

M+ is distributed as a single `index.html`.

## Local use

1. Download the M+ release.
2. Extract the ZIP.
3. Open `index.html` in a modern browser.
4. Open **Settings**.
5. Add one or more API keys.
6. Validate the providers and load their available models.
7. Open **Roles**.
8. Select the Main model.
9. Add zero or more Reviewers.
10. Choose the review level.
11. Create a project.
12. Type a request or attach files and work on them directly.

No local server is required for the normal browser-local workflow.

Some third-party APIs impose browser CORS restrictions. M+ cannot override a provider's network policy. A provider that blocks direct browser requests may require a compatible endpoint or a different hosting arrangement.

---

# Projects and chats

M+ distinguishes **projects** from **chats**.

## Project

A project is the durable work container.

It owns:

- project name
- project team configuration
- active workspace files
- project-wide checkpoints
- project settings
- evidence configuration
- budget settings
- one or more chats

## Chat

A chat is a conversation inside a project.

It owns:

- chat name
- messages
- runs
- acceptance criteria
- chat verification history

This means one project can contain several conversations while retaining the same evolving file workspace.

## Sidebar controls

### New project

Creates a new project with:

- a new chat
- the current default team
- default Evidence settings
- default Budget settings

### Expand/collapse project

Each project can be expanded to reveal its chats.

### New chat

Creates another chat inside an existing project.

### Rename project

Changes the project name without forcing the chat name to change.

### Rename chat

Changes only the conversation name.

### Delete chat

Deletes the selected chat and unreferenced chat-only attachments while protecting files still used by:

- the project workspace
- checkpoints
- other chats
- other run artifacts

### Delete project

Deletes the project and its stored attachments from the selected storage backend.

### Refresh

Reloads the project list from the current storage backend.

---

# The main interface

## Main model button

The model button in the top bar shows the selected Main model and opens the Roles configuration.

The selected model and API-key availability are treated separately. A model can remain selected even if its API key is temporarily locked or unavailable.

## Project name and chat name

Project and chat names are independent.

M+ automatically gives new work a useful name after the first request, but both can be changed manually.

## Project button

Opens the Project tools:

- Requirements
- Checkpoints
- Verification
- Evidence
- Budget
- Files

## Usage

Opens detailed model usage and cost information.

## Export

Exports the current chat as:

- HTML
- Markdown

## Composer

The composer supports:

- typed requests
- file attachments
- drag-and-drop files
- Enter to send
- Shift+Enter for a new line

While a run is active, the Send button becomes a Stop control.

---

# Settings - APIs

The APIs tab configures model providers and API-key security.

It is split into:

1. API-key security
2. built-in model providers
3. Custom / BYOK providers

M+ does not hard-code one tiny model list and assume those IDs exist for every user. Built-in providers attempt to discover the models actually available to the current API key.

## Provider validation

For a built-in provider:

1. enter the API key
2. M+ requests the provider's model catalogue
3. obviously incompatible families such as embedding, image, speech or moderation models are filtered where appropriate
4. compatible models are ranked for usability
5. the resulting exact model IDs become available to Main and Reviewer roles

The refresh button repeats discovery.

A successful provider catalogue can establish that a previously saved model is no longer available.

A temporary timeout, locked key or discovery failure should not be treated as proof that a previously verified model ceased to exist.

## Model selection

Each built-in provider card contains:

- API key
- Get API key link
- Model selector
- Refresh models button
- current validation/status message
- provider-specific setup help

Amazon also has an AWS Region field.

### Automatic model selection

Every provider model selector has an **Automatic** option.

In Automatic mode, M+ uses its detected preferred/strongest available compatible model for that provider. The exact model name is shown beside the Automatic option when known.

### Explicit model selection

Selecting a specific discovered model pins that exact model ID rather than leaving the provider on Automatic.

This distinction matters because the APIs tab controls the provider's preferred/default model, while the Roles tab can assign exact models independently to Main and Reviewer slots.

---

# API-key security

M+ intentionally does not persist plaintext model API keys in ordinary browser settings.

There are two modes.

## Session-only keys

This is the default.

Keys entered into M+ are held in page memory.

They disappear when the page is closed or reloaded.

Use this when you prefer maximum simplicity and do not mind re-entering keys.

## Encrypted browser vault

The APIs tab can optionally store keys in an encrypted vault on the current device.

The current implementation uses:

- AES-256-GCM encryption
- PBKDF2-SHA256 key derivation
- 310,000 PBKDF2 iterations
- a random salt
- a random AES-GCM IV
- a passphrase that is never stored

The vault can be:

- created
- unlocked
- locked immediately
- removed

When the vault is locked, saved keys are not restored into provider memory.

When M+ opens and an encrypted vault exists, it can ask for the passphrase once for that browser session.

### Passphrase requirements

A newly created vault requires a passphrase of at least 8 characters.

### Important security boundary

Encryption protects keys **at rest** in browser storage.

It cannot protect an unlocked key from:

- a malicious browser extension
- compromised browser code
- a compromised operating system
- other code already able to inspect the page's live memory

## Legacy plaintext migration

M+ includes migration handling for settings created by older builds that persisted plaintext keys. When detected, those keys are moved into volatile session memory and removed from normal persistent settings.

---

# Supported providers

M+ currently exposes these built-in provider cards in this order:

1. GPT
2. Claude
3. Gemini
4. Grok
5. Mistral
6. Meta
7. Perplexity
8. Amazon
9. Cohere
10. Kimi
11. Qwen
12. DeepSeek

The point of the provider list is not to impose one "best" model. M+ is explicitly designed so the user can choose different model families for Main and Reviewers.

## GPT

- native OpenAI integration
- model discovery through the OpenAI model catalogue
- usable as Main or Reviewer
- supported by M+ web research
- supported for visual review
- GPT Image is also used by the current image-generation path

## Claude

- native Anthropic Messages integration
- model discovery
- usable as Main or Reviewer
- supported by M+ web research
- supported for visual review

## Gemini

- native Google Gemini integration
- model discovery filtered to generation-capable models
- usable as Main or Reviewer
- supported by M+ web research
- supported for visual review

## Grok

- xAI integration
- OpenAI-compatible normal work path
- model discovery
- usable as Main or Reviewer
- supported by M+ web research

## Mistral

- Mistral integration
- model discovery
- usable as Main or Reviewer
- supported by M+ web research

## Meta

- Meta Model API integration
- uses the OpenAI-compatible Chat Completions surface
- model discovery
- usable as Main or Reviewer

## Perplexity

- Perplexity Gateway integration
- OpenAI-compatible model access
- Gateway model discovery
- usable as Main or Reviewer

M+ currently treats Perplexity here as a **model provider through Gateway**, not as the Project Evidence web-research engine. The dedicated web-research pipeline currently uses GPT, Claude, Gemini, Grok or Mistral.

## Amazon

- Amazon Bedrock integration
- uses Bedrock's OpenAI-compatible Mantle interface
- requires an **AWS Region**
- model discovery is limited by M+ to Amazon Nova conversational models to avoid duplicating every third-party model available through Bedrock
- usable as Main or Reviewer

Default region shown by M+ is `us-east-1`, but users should select the region that matches their Bedrock key and model access.

## Cohere

- Cohere integration
- OpenAI-compatible work path
- model discovery
- usable as Main or Reviewer

## Kimi

- Moonshot/Kimi OpenAI-compatible API
- model discovery
- usable as Main or Reviewer

## Qwen

- Alibaba Qwen / Model Studio compatible API
- M+ can try supported regional compatible-mode endpoints
- successful discovery can retain the working base URL
- usable as Main or Reviewer

## DeepSeek

- DeepSeek OpenAI-compatible API
- model discovery
- usable as Main or Reviewer

---

# Custom / BYOK providers

M+ includes four custom provider slots:

- Custom 1
- Custom 2
- Custom 3
- Custom 4

They are designed for:

- smaller providers
- private gateways
- local model servers
- organisation-specific OpenAI-compatible infrastructure

Each custom slot exposes:

### Display name

Lets you rename the slot to the actual provider or local model service.

### Base URL

Example:

```text
http://localhost:1234/v1
```

The endpoint must be OpenAI-compatible.

### API key - optional

A key is optional because many local model servers do not require authentication.

### Discovered model

M+ attempts:

```text
GET {baseUrl}/models
```

### Manual model ID

If `/models` is unavailable, a manual model ID can be entered.

Manual models are deliberately labelled as manual rather than falsely presented as API-validated.

### Work endpoint

Custom providers use:

```text
POST {baseUrl}/chat/completions
```

### CORS

Because M+ runs in the browser, the custom endpoint must allow browser CORS.

---

# Settings - Roles

The Roles tab defines the actual AI team.

It contains:

- Main model
- Review level
- Maximum iterations
- Reviewer 1 to Reviewer 12

## Main model

Exactly one Main model owns the work.

The selector is built from models that M+ has validated or retained from successful provider discovery.

The Main can be from any configured built-in or Custom/BYOK provider.

## Reviewer slots

There are twelve independent Reviewer slots.

Each can be:

- Not used
- any available validated model

Reviewers are assigned at the exact model level, not merely by provider.

For example, two reviewer slots can both use the same Claude model.

## Duplicate Reviewers

Duplicates are allowed deliberately.

Two copies of the same exact model run as separate Reviewer instances.

This is not guaranteed to create independent reasoning in a statistical sense, but it allows repeated independent review calls rather than forcing model uniqueness.

## Project-specific teams

Each project retains its own:

- Main
- Reviewer slots
- Review level
- maximum iterations

Opening a project restores that project's team.

## Default team

Saving from the Roles tab also makes the current team the default used when future projects are created.

Existing projects keep their own saved teams.

---

# Review levels

## Off

Disables model Reviewers.

The Main still performs the work.

Deterministic validators may still run where applicable, so "Off" means **no LLM competitive review**, not "disable every quality check".

Use Off when:

- the task is trivial
- cost matters more than adversarial review
- you want the Main only
- you are testing provider connectivity

## Standard - adversarial

This is the default review level.

A Standard Reviewer is instructed to:

- act as an independent competing expert
- question unsupported conclusions
- ask what evidence supports important claims
- test at least one plausible alternative or what-if scenario
- look for missed requirements
- look for edge cases
- propose a stronger approach when one exists
- avoid manufacturing objections merely for the sake of disagreement

The goal is scepticism without performative contrarianism.

## Rigorous - competitive

Rigorous mode increases the pressure.

A Rigorous Reviewer is explicitly treated as a rival expert trying to prove it can deliver a better result.

It is instructed to:

- treat important assumptions as contestable
- demand evidence and sources
- look for falsification
- search for counterexamples
- test alternative scenarios
- find hidden regressions
- challenge the Main's reasoning
- produce concrete counterproposals
- concede PASS only after a serious attempt to disprove or outperform the candidate

Rigorous mode can consume significantly more tokens and calls.

---

# Reviewer behaviour and pass rules

This is one of the most important parts of M+.

## Reviewers run independently

Configured Reviewers are launched in parallel for each review iteration.

A Reviewer's first-pass findings are not shown to the other Reviewers before they finish their own pass.

This prevents the review process from becoming one model anchoring the rest.

## Reviewers do not directly edit the work

A Reviewer can propose a correction or alternative.

It cannot commit the final revision.

The Main must decide.

## Issue severities

Reviewer issues can be:

- blocker
- high
- medium
- low

Severity describes importance.

It does **not** control whether another iteration is required.

A real low-severity unresolved issue is still an issue.

## Challenge questions

Reviewers can ask questions that the Main must be able to answer.

The question should matter to the result, for example:

- What evidence supports this assumption?
- What happens in this edge case?
- Why was alternative B rejected?
- How is this file path reached?
- What source would falsify this claim?

## Counterproposals

A Reviewer can propose a materially better approach.

The Main must respond with:

- adopt
- partial
- reject

and give a reason.

## Finding dispositions

For each material issue the Main can decide:

- **accept**
- **partial**
- **reject**

The decision trail is displayed in the conversation.

Rejecting a finding is allowed. The Main is not required to obey the Reviewer blindly. But rejection should be evidence-based.

## Unconditional PASS

A Reviewer passes only when all of these are true:

- the Reviewer call completed successfully
- verdict is `pass`
- there are no issues
- there are no unresolved challenge questions
- there is no counterproposal that the Reviewer believes is superior
- every supplied acceptance criterion is passed

A "pass" with an unresolved caveat is converted into changes required.

## Reviewer failures

One provider failing does not necessarily destroy the entire run.

M+ can continue with the Reviewers that remain available.

Final status distinguishes clean review success from cases where one or more Reviewers were unavailable.

## Maximum iterations

The Roles tab contains **Maximum iterations**.

Default:

```text
5
```

Minimum:

```text
1
```

This limits the Main -> Review -> Revision loop.

M+ does not currently impose a separate hard maximum in the field, so very large values should be used carefully because they can create high API cost.

Budget guardrails exist for this reason.

If the iteration limit is reached before unconditional verification, M+ reports that explicitly rather than pretending the work passed.

---

# Project tools

The Project dialog is the control centre for work that needs stronger structure.

It contains six tabs:

1. Requirements
2. Checkpoints
3. Verification
4. Evidence
5. Budget
6. Files

Project settings are saved with **Save project**.

---

# Requirements

Requirements are binding acceptance criteria.

## Automatic criteria

When a new chat receives its first meaningful request and has no criteria yet, M+ derives a compact checklist.

Depending on the task, criteria can be derived heuristically or with the Main model.

M+ can generate up to twelve automatic criteria.

## Add criterion

Adds a new criterion manually.

## Edit criterion

Criteria are editable text.

## Enable / disable

Each criterion has a checkbox.

A disabled criterion remains stored but is not binding for the next run.

## Remove

Deletes the criterion.

## Per-chat scope

Acceptance criteria belong to the chat, not the entire project.

Different chats inside one project can therefore pursue different objectives while sharing the same project workspace.

## Reviewer enforcement

Reviewers receive the enabled criteria.

If a Reviewer fails to return an unconditional pass for a required criterion, M+ forces that criterion into the changes-required path.

---

# Checkpoints

Checkpoints provide project-wide workspace history.

## Baseline

The first attached workspace becomes a baseline checkpoint.

Older imported projects can also receive an automatic imported-workspace baseline.

## Automatic run checkpoints

When reviewed file work changes the workspace, M+ can persist intermediate/final workspace checkpoints with metadata including:

- label
- time
- originating chat
- run ID
- stage
- iteration
- changed files
- workspace attachment IDs

## Diff

The **Diff** button compares a selected checkpoint with the current project workspace.

It reports:

- added files
- removed files
- modified files
- changed text lines where available

## Restore

The **Restore** button makes the selected checkpoint the current active project workspace.

This does not require manually re-uploading the old files.

## Project-wide scope

Checkpoints are project-wide, even though the UI records which chat created them.

---

# Verification

Verification combines model review with deterministic checks.

The Verification tab displays the latest report for the active chat and can also show reports from other chats in the project.

## Summary

The latest report shows:

- run status
- passed acceptance criteria count
- validator failure count
- estimated known cost

## Acceptance criteria

Each criterion can be:

- pass
- fail
- unverified

## Deterministic validation

The current browser validator can check several things without asking another LLM.

### Archive package integrity

Checks for:

- duplicate archive paths
- absolute paths
- parent-traversal paths

### DOCX structure

Checks required package entries such as:

- `[Content_Types].xml`
- `word/document.xml`

### XLSX structure

Checks required package entries such as:

- `[Content_Types].xml`
- `xl/workbook.xml`

### PPTX structure

Checks required package entries such as:

- `[Content_Types].xml`
- `ppt/presentation.xml`

### ODF structure

For ODT/ODS/ODP packages, checks required entries including:

- `mimetype`
- `content.xml`

### JSON syntax

Parses textual `.json` files.

### XML syntax

Parses XML-like files including:

- XML
- SVG
- RELS
- package content-type XML

### HTML local references

Examines local `src` and `href` references in HTML and reports missing local workspace paths.

### Evidence citations

When Evidence Mode is on, M+ verifies that citation tokens resolve to real evidence:

- `[[E1]]` style pinned evidence
- `[[W1]]` style web research sources
- `[[workspace:path:Lx-Ly]]` style workspace line citations

## Run deterministic checks

The Verification tab includes a manual **Run deterministic checks** button.

This re-runs validators against the current workspace.

## Download report

Downloads a Markdown verification report containing:

- project/chat
- run status
- Main
- Reviewers
- criteria
- deterministic checks
- web research
- evidence status
- changed files
- usage

---

# Web research and evidence

Project > Evidence contains two related but distinct systems:

1. live web research
2. pinned evidence sources

## Web research modes

### Auto

Default.

M+ decides whether the current request needs live research.

Auto recognises signals such as:

- research
- web / online / browse
- look up
- latest / current / recent / today
- news
- sources / citations
- public information
- price / availability
- release / version
- CEO / president / election
- weather
- sports scores / schedules / standings
- stock / market
- laws / regulations
- statistics
- competitors
- explicit URLs

### Always

Runs research before every task.

### Off

Never starts automatic web research.

## Search budget

Range:

```text
1 to 12
```

Default:

```text
5
```

The budget is supplied both:

- as a provider tool limit where the provider supports one
- as an explicit instruction to the research model

## Web-capable research providers

The current M+ research adapter can use connected models from:

- GPT
- Claude
- Gemini
- Grok
- Mistral

M+ considers:

- the current Main if it is web-capable
- configured Reviewers if they are web-capable
- other connected web-capable configured providers

## Shared research dossier

Research is performed once for the task.

The resulting dossier and source list are then shared with:

- Main
- every Reviewer

This avoids paying each Reviewer to independently rediscover the same public facts and reduces evidence asymmetry.

## Web citations

Web research sources are identified as:

```text
[[W1]]
[[W2]]
...
```

M+ renders these as clickable citations when the source exists.

## Evidence Mode

Toggle:

**Require evidence citations in Main and reviewer work**

When enabled, M+ instructs models to ground factual claims in:

- workspace evidence
- pinned evidence
- web research

The deterministic validator also checks that citation references resolve.

## Pinned evidence URLs

Enter one per line.

Either:

```text
https://example.com/reference
```

or:

```text
Specification | https://example.com/spec
```

Pinned evidence sources are numbered:

```text
E1
E2
...
```

and cited as:

```text
[[E1]]
```

## Refresh URLs

Pinned URLs can be refreshed manually.

When Evidence Mode is used in a run, stale or missing pinned content is refreshed automatically. Current refresh age is approximately 24 hours.

## Pinned-source CORS

Pinned URLs are fetched directly by the browser.

A site can block that fetch with its CORS policy.

This is different from model-native web research, which runs through the connected model provider's server-side search capability.

---

# Budget guardrails

Project > Budget protects against accidentally expensive review loops.

There are two independent thresholds.

## Warn after estimated USD

Default:

```text
0 = off
```

When the known estimated run cost reaches the threshold, M+ adds a budget warning.

The run continues.

## Require approval after estimated USD

Default:

```text
0 = off
```

When the known estimated cost reaches this threshold, M+ pauses before the next API call.

The user chooses:

- **Stop run**
- **Continue this run**

Once approved, the current run is allowed to continue without asking at every subsequent call.

## Important limitation

The dollar guardrail only works for calls whose cost M+ can estimate or whose provider reports an exact cost.

Unknown-price models still contribute token usage but cannot be reliably governed by a dollar threshold.

---

# Files and workspaces

Attaching files turns M+ from a chat-only system into a project workspace system.

## Project-wide active workspace

Uploaded files are added to the project's active workspace.

They remain available to later turns and other chats in the same project until replaced, restored or removed through project evolution.

## Files tab

Project > Files displays the current active workspace.

Each file shows:

- type
- filename
- size
- package/editability information
- Download button

## Supported editable package formats

M+ currently treats these archive/package formats as editable workspaces:

- ZIP
- DOCX
- XLSX
- PPTX
- ODT
- ODS
- ODP

For Office and ODF packages, M+ edits textual/XML members inside the package and preserves unchanged binary members.

## Text files

M+ recognises a broad set of textual formats, including common:

- plain text
- Markdown
- JavaScript / TypeScript
- JSON
- CSS
- HTML
- XML
- CSV / TSV
- YAML
- TOML
- INI/config
- C/C++
- Python
- Java
- Kotlin
- Swift
- Go
- Rust
- C#
- SQL
- shell
- BAT
- PowerShell
- assembly
- HDL
- Arduino
- build/config files

## Binary files

Opaque binary files are retained in the workspace but cannot currently be rewritten by the browser text-editing engine.

PDF is specifically preserved but is not rewritten by the current workspace editor.

---

# File editing

For workspace tasks, the Main does not receive a vague file summary and hallucinate edits.

It works through an explicit virtual workspace protocol.

## Inspection

The Main can ask M+ to:

- search current workspace text
- read exact files
- read exact line ranges
- inspect the original workspace when comparison matters

The application returns actual evidence before the Main continues.

## Main change operations

M+ supports exact file operations:

### `replace_text`

Replace exact existing text.

The Main supplies:

- path
- exact find string
- replacement
- expected match count

M+ rejects the edit if the exact match count differs.

### `replace_file`

Replace the complete text content of an existing textual file.

### `add`

Add a new textual path.

### `delete`

Delete an existing path.

## Atomic rejection

If a proposed change set is invalid, M+ rolls the workspace back to its pre-change state and gives the Main one correction opportunity.

Examples of rejected operations include:

- missing path
- ambiguous path
- wrong operation type
- trying to text-edit binary data
- exact replacement count mismatch

Repeated invalid change sets are stopped to avoid paid no-progress loops.

## Reviewer inspection requirement

A workspace Reviewer cannot honestly pass a file change by reading only the Main's explanation.

M+ requires the Reviewer to directly read every materially changed file before completion.

The Reviewer can also inspect related dependencies and the original workspace.

## No-progress protection

M+ includes guards against pathological model loops.

Examples:

- repeated identical Main inspections
- repeated identical Reviewer inspections
- repeated invalid workspace-control JSON
- repeated rejected file-change sets
- Reviewer attempts to finish without inspecting changed files

These guards stop or isolate the failing participant rather than endlessly spending API credit.

---

# Downloadable text and code files

A request can explicitly ask M+ to create a real downloadable text/code/data file.

Examples:

- "Give me this as `report.md`"
- "Create a downloadable Python file"
- "Save this as JSON"
- "Produce `index.html`"

M+ instructs the Main to return the actual file contents through an internal delivery protocol.

The browser then creates the downloadable artifact locally.

Supported extensions include common:

- TXT
- MD
- PY
- JS
- TS
- JSON
- CSV
- XML
- SVG
- HTML
- CSS
- YAML
- TOML
- C/C++
- Java
- Go
- Rust
- SQL
- shell / BAT / PowerShell
- Arduino

M+ explicitly tells models **not to invent fake `sandbox:`, `blob:`, `data:` or local file links**.

The downloadable file is created by M+, not claimed by the model.

---

# Image generation and visual review

M+ can detect a direct image-generation request and use its current image workflow.

## Main's role

The selected Main model:

1. interprets the user's request
2. creates the image-generation brief
3. remains responsible for revisions after visual review

## Image generator

The current browser build uses:

```text
GPT Image 2
```

through the GPT provider.

A GPT API key is therefore required for image generation even when the selected Main is another provider.

## Current generation defaults

The current adapter requests:

- PNG
- 1024 x 1024
- medium quality
- one image

## Visual reviewers

The current visual-review adapter supports:

- GPT
- Claude
- Gemini

Other configured Reviewer providers can be valid for normal text/workspace review but may be unavailable for image inspection in the current single-file build.

## Visual review loop

Reviewers inspect the **actual generated image**, not merely the prompt.

They can report:

- visible defects
- missing requested elements
- acceptance-criterion failures
- challenge questions
- stronger visual approaches

The Main then revises the image prompt and M+ regenerates the image.

---

# Usage and estimated cost

The top-bar **Usage** control opens M+'s local usage dashboard.

## Current / last run

Shows:

- estimated cost
- successful API call count
- input tokens
- cached input tokens where reported
- output tokens
- reasoning tokens where reported
- per-model rows

## Current scope

Shows aggregate usage for:

- current chat
- current project

## Tracked on this browser

Shows:

- Today
- This month
- All tracked

## Cost data

Token counts come from provider API responses.

Monetary values are based on:

- provider-reported exact cost where available
- M+'s embedded public list-price catalogue where known

The current v1.3.11 price catalogue is dated:

```text
2026-08-08
```

Pricing can differ because of:

- free tiers
- enterprise agreements
- regional pricing
- special processing modes
- newly released models
- provider changes after the embedded catalogue date

A trailing `+` in M+ indicates that some calls had no known price and are therefore excluded from the dollar total.

## Clear local usage history

Deletes the local M+ usage ledger.

It does **not** delete chats.

---

# Retry, stop and recovery

## Stop

While a run is active, the Send control becomes Stop.

Stopping aborts the active request where possible and records the run as stopped.

## Retry

The latest failed or stopped run can be retried without manually reconstructing the entire user prompt.

M+ can retain resume information such as:

- candidate
- review stage
- iteration
- acceptance criteria
- web research
- workspace state/reference

The exact resume path depends on where the run stopped.

## Provider errors

M+ attempts to present provider errors in a more useful form, including billing/quota failures where recognised.

## Reviewer unavailability

A Reviewer failure is surfaced explicitly.

The rest of the team can continue where possible.

M+ distinguishes:

- clean pass
- pass with Reviewer errors
- review unavailable
- maximum iterations reached
- unresolved findings
- request failure

---

# Export

The Export control supports two formats.

## HTML

Creates a standalone HTML representation of the current chat including the visible work/review timeline styling.

Interactive app controls are removed from the export.

## Markdown

Creates a Markdown transcript containing:

- user messages
- attachment names
- Main candidates
- Reviewer iterations
- challenge questions
- counterproposals
- issue findings
- Main dispositions
- revised candidates
- final status
- verification summary
- generated artifact names

Web citations are converted to ordinary Markdown links where the source is available.

---

# Storage

M+ has two project-storage modes.

## Local browser

Default.

Projects and attachments are primarily stored in IndexedDB.

Non-secret settings and usage metadata use browser storage.

If IndexedDB is unavailable, M+ contains a reduced fallback path. In that condition the UI may indicate that storage is session-only or otherwise limited.

## Shared workspace

Optional.

Shared mode moves project persistence to an external M+ Sync API v1 server.

The AI providers themselves remain separate.

---

# Shared workspace mode

Settings > Storage contains:

## Storage mode

Options:

- Local browser
- Shared workspace

## Your display name

Optional collaborative attribution.

M+ can send this to the sync server as the current actor name.

## M+ Sync server URL

Base URL of the external sync service.

The service must support M+ Sync API v1 and browser CORS.

## Workspace

Logical shared-workspace name.

Default:

```text
default
```

## Access token

Optional bearer token for the M+ Sync server.

The access token is session-only.

It is not stored in project data.

## Test connection

Performs the M+ Sync handshake and validates the configured server/workspace.

## Copy current project

Copies the active project between local and shared storage.

The copy includes referenced:

- project data
- chats
- files
- checkpoints

This is useful both for migration and for keeping a local/shared copy.

## Collaboration behaviour

Shared mode includes:

- project revisions
- conflict detection
- project locks
- polling for remote changes
- collaborator attribution

The default internal poll interval is 5 seconds.

## Secret separation

Shared project data does not contain:

- model provider API keys
- shared access token

---

# M+ Sync API v1

A compatible shared server is expected to expose this basic contract.

## Handshake

```text
GET /v1/workspaces/{workspace}
```

The handshake must identify:

```text
mplus-sync-v1
```

## Project list

```text
GET /v1/workspaces/{workspace}/projects
```

## Individual project

```text
GET    /v1/workspaces/{workspace}/projects/{id}
PUT    /v1/workspaces/{workspace}/projects/{id}
DELETE /v1/workspaces/{workspace}/projects/{id}
```

Project PUT uses revision-aware `If-Match` semantics.

A revision conflict returns:

```text
HTTP 409
```

with the current project state.

## Attachments

```text
GET    /v1/workspaces/{workspace}/attachments/{id}
PUT    /v1/workspaces/{workspace}/attachments/{id}
DELETE /v1/workspaces/{workspace}/attachments/{id}
```

Attachment PUT uses raw file bytes plus M+ metadata headers.

## Workspace locks

```text
POST   /v1/workspaces/{workspace}/locks/{projectId}
DELETE /v1/workspaces/{workspace}/locks/{projectId}
```

A project locked by another client returns:

```text
HTTP 423
```

---

# GitHub and update checks

The License tab contains:

- View on GitHub
- Check for updates
- current M+ version
- update status

The current development build uses a repository placeholder until the public repository path is inserted.

Once configured, M+ checks:

```text
https://api.github.com/repos/{OWNER}/{REPO}/releases/latest
```

## Automatic check

M+ caches the result locally and checks at most once every approximately 24 hours.

## Manual check

**Check for updates** forces an immediate check.

## Version comparison

M+ compares the current semantic-style version against the latest GitHub Release tag.

Example release tag:

```text
v1.4.0
```

## Download selection

M+ looks first for a ZIP release asset whose filename contains `main-review`.

If none exists, it falls back to another ZIP or the release page.

## No silent self-update

M+ does not replace its own local HTML automatically.

It tells the user that an update exists and provides the release/download link.

This is intentional because M+ handles local projects and API credentials. Update installation remains an explicit user action.

---

# Security and privacy model

M+ is a client-side browser application.

## No mandatory Pathbind account

Normal local use does not require a Pathbind Games account.

## Provider calls

Model requests are sent directly from the browser to the provider endpoint configured in M+.

M+ does not need a Pathbind-hosted AI relay for ordinary local use.

## API keys

Provider keys are:

- session-only by default
- optionally encrypted locally
- excluded from persisted normal settings
- excluded from project data
- excluded from shared project data

## Shared token

The shared-storage access token is held only for the current browser session.

## Local data

Local projects and files are primarily stored in IndexedDB.

## Content Security Policy

The single-file application includes a restrictive CSP.

The current design blocks arbitrary default resources and explicitly permits only the resource classes M+ needs, including provider network connections.

## What M+ cannot protect against

M+ should not be treated as a hardened secret-management appliance.

It cannot protect an unlocked API key from a compromised:

- browser
- extension
- operating system
- injected script running with equivalent page privileges

Users handling high-value organisational credentials should apply their normal endpoint-security and key-rotation practices.

---

# Current implementation limits

These are current implementation boundaries, not promises about future releases.

## Browser CORS

Direct provider and pinned-URL calls depend on browser CORS policy.

## Attachment/context limits

Current internal limits include:

```text
Total extracted attachment context: 180,000 characters
Single-file extracted context:       40,000 characters
ZIP entries:                         500
ZIP uncompressed size:               30 MiB
```

These limits protect a single browser page from pathological archives and oversized prompt context.

## Office editing

M+ edits textual/XML parts of supported packages.

It is not a complete Microsoft Office rendering engine.

It preserves unchanged binary package parts but cannot visually re-layout a document with the fidelity of Word, Excel or PowerPoint.

## PDF

PDF files can be preserved as workspace data but are not rewritten by the current browser workspace engine.

## Image generation

Current image generation depends on a GPT API key and GPT Image 2.

## Visual review provider coverage

Current direct image inspection supports GPT, Claude and Gemini.

## Web research provider coverage

Current project web-research adapters support GPT, Claude, Gemini, Grok and Mistral.

This is separate from a provider's general ability to access web information through its own products.

## Cost estimates

Cost estimates can become stale as providers change prices.

## Model catalogues

Provider model availability changes frequently.

M+ therefore discovers model IDs from the user's actual API access wherever possible instead of treating the README as the model catalogue.

---

# Licence

M+ is **source-available**, not OSI open source.

It is released under the **Pathbind M+ License 1.0**.

In plain language:

## You may

- use M+ free of charge
- use it personally
- use it educationally
- use it for research
- use it inside a business
- use it to produce paid client work
- use it to create other commercial work/products/services
- modify it for your own use
- redistribute original or modified copies free of charge, subject to the licence terms

## You may not

- sell M+ itself
- rent or lease M+
- charge a subscription for access to M+
- commercially redistribute a substantially derived version
- rebrand it, make superficial modifications and sell the result
- make substantially derived M+ functionality a material paid feature of another product or hosted service

Redistributed copies must retain the licence and copyright notice.

Modified redistributions must be identified as modified.

The complete licence text in the application is authoritative. This README summary is not a substitute for the licence.

Copyright © 2026 Pathbind Games.

---

# Pathbind Games and Gydel

M+ was originally built as an internal Pathbind Games tool and is released free of charge.

If you find it useful, consider supporting the people who made it.

## Pathbind Games

Independent games and experimental play.

https://pathbind.games/

## Gydel

Audio games where anything can happen.

https://gydel.games/

M+ itself has no subscription or paid tier. Model API usage is separate and billed by the providers you choose.

---

# External landscape references

This section exists to explain the design context, not to claim that every competing product follows one identical architecture.

## Side-by-side model comparison

**ChatHub**

ChatHub describes itself as a way to use and compare multiple AI chatbots simultaneously.

- https://chathub.gg/
- https://doc.chathub.gg/premium-features/summarize-chat

M+ differs by maintaining one Main-owned evolving result rather than making the user choose among parallel final answers.

## Collaborative multi-agent systems

**CrewAI**

CrewAI describes collaborative AI agents, crews and flows.

- https://docs.crewai.com/

M+ Reviewers are intentionally not collaborators or delegated workers. They are adversarial reviewers of a Main-owned result.

## Supervisor and specialist architectures

**LangChain / LangGraph multi-agent patterns**

Current LangChain documentation describes multi-agent systems and supervisor patterns that coordinate specialised agents.

- https://docs.langchain.com/oss/python/langchain/multi-agent
- https://docs.langchain.com/oss/python/langchain/multi-agent/subagents
- https://docs.langchain.com/oss/python/langgraph/overview

M+ is not primarily a routing/supervisor framework. Its defining relation is author versus critic.

## General-purpose conversational multi-agent frameworks

**AutoGen**

AutoGen documents single-agent and multi-agent conversational applications and historically emphasised cooperation among multiple agents.

- https://microsoft.github.io/autogen/stable/
- https://www.microsoft.com/en-us/research/project/autogen/

M+ is a complete end-user work-and-review product rather than a general framework for assembling arbitrary agent interactions.

## Aggregation / Mixture-of-Agents

**Mixture-of-Agents Enhances Large Language Model Capabilities**

The MoA research pattern uses multiple LLM outputs and an aggregation stage.

- https://arxiv.org/abs/2406.04692

M+ deliberately preserves a single author instead of treating several proposer answers as equally owned inputs to a synthesiser.

## Debate and majority voting

Research on multi-agent debate commonly studies voting, aggregation and consensus dynamics.

Examples:

- https://arxiv.org/abs/2508.17536
- https://arxiv.org/abs/2510.12697

M+ rejects majority rule as the core review mechanism. One strong criticism is enough to require the Main to address it. Reviewers do not need to outvote one another.

---

# Short version

If you only remember one thing about M+, remember this:

> **The Main owns everything. The Reviewers own nothing. Their job is to make the Main earn the right to keep its answer.**

That is the M+ model.
