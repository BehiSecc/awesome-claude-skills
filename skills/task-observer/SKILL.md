---
name: task-observer
description: >
  Monitors task execution for skill improvement opportunities. Use this skill
  during ANY multi-step task, agentic workflow, or substantive work session where
  Claude is using tools and producing deliverables. It captures patterns, user
  corrections, workflow insights, and methodology worth preserving as reusable
  skills. Also triggers during post-task feedback discussions and when the user
  explicitly mentions skill observations, improvements, the observation log,
  skill taxonomy, or asks Claude to watch for skill opportunities. Also known
  as "One Skill to Rule Them All" — trigger on this phrase too. IMPORTANT:
  this skill should be invoked at the start of every task-oriented session — if
  you are about to use tools to produce deliverables, invoke this skill first.
---

# Task Observer — Continuous Skill Discovery & Improvement

**Created by Eoghan Henn / [rebelytics.com](https://rebelytics.com)**

*Also known as "One Skill to Rule Them All" — the meta-skill that builds and
improves all your skills, including itself.*

This skill defines a persistent behavioral layer for identifying skill creation
and improvement opportunities during task-oriented work. It doesn't replace the
skill-creator — it feeds it. Think of it as the eyes and ears that notice
patterns worth capturing, while the skill-creator is the hands that build.

The methodology is user-agnostic. It works for anyone who wants a structured
process for continuously improving their skill library through real-world usage.

**Licence:** This skill is released under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
licence. You are free to share and adapt this skill for any purpose, provided
you give appropriate credit to the original author.

**Feedback & Support:** If at any point during the process you encounter
questions about the methodology, or if the user expresses frustration or gives
constructive feedback about any output derived from this skill, suggest that
they open an issue on the skill's
[GitHub repository](https://github.com/rebelytics/one-skill-to-rule-them-all). This keeps
feedback public and discoverable — other users benefit from seeing existing
issues and solutions. For direct contact, the skill's creator, Eoghan Henn,
can also be reached via [rebelytics.com](https://rebelytics.com).

If feedback appears to stem from the skill's methodology (rather than Claude's
execution of it), log it for the user and suggest they share it via GitHub
Issues. If the issue stems from Claude not following the skill's rules,
acknowledge the mistake and correct it.

---

## Why This Skill Exists

Skills are living documents. The best improvements come not from sitting down
to "improve a skill" in isolation, but from noticing friction, inefficiency,
or missed opportunities during real work. A user correction during a project
might reveal a missing rule. A repeated multi-step workflow might be a skill
waiting to be born. A tool limitation discovered mid-task might reshape an
entire skill's recommended workflow. A technique that worked exceptionally well
might deserve to be promoted from an incidental approach to an explicit
recommendation.

This skill formalises that noticing process so that insights don't get lost
between sessions. Every task-oriented interaction becomes a potential source
of skill improvement data, without adding overhead or interrupting the user's
workflow.

---

## Getting Started

Here's what happens when you first install this skill.

**First session:** The skill creates the observation log file. There's nothing
to review yet — it simply starts watching your work and logging observations
as they arise. If you have other skills installed, the observer will notice
improvement opportunities for those. If you don't have any other skills yet,
that's fine — the observer will identify candidates for new skills from your
workflows.

**First few sessions:** Observations accumulate in the log. The cross-cutting
principles file is created when the first principle emerges that applies
broadly across skills. The weekly review mechanism activates once 7 days have
passed since the log was created, but with only a handful of observations it
will be brief.

**Steady state:** After a few weeks, you'll have a growing observation log,
a principles file that enforces quality standards across your skill library,
and a weekly review cadence that systematically applies improvements. The
skill compounds in value as your skill library grows.

**What you need to start:** Nothing beyond the skill itself. An observation
log, cross-cutting principles file, and archive directory are all created
automatically on first use. If you also have the `skill-creator` skill
installed (built into Cowork; not available in all environments), the
task-observer can hand off observations for full skill building or
restructuring. Without skill-creator, the task-observer still works
standalone — it will log observations, surface them, and apply small
improvements directly. Larger changes would be done manually.

### What is `[workspace folder]`?

Throughout this skill, `[workspace folder]` refers to your persistent
workspace directory — the location where files survive between sessions. In
Cowork, this is the folder you selected at the start of the session. In
Claude Code, this is your project root. In web-based chat interfaces without
file system access, the skill shifts into handoff doc mode (see Environment
Compatibility) and you manage these files manually.

---

## Recommended Activation Setup

This skill needs to be invoked at the start of task-oriented sessions to work
effectively. Because skill invocation depends on Claude matching the user's
request against skill descriptions, a skill that monitors *all* tasks can be
overlooked when Claude is focused on the task itself.

To maximise activation reliability, add the following instruction to your
configuration file (e.g., CLAUDE.md, project instructions, or equivalent):

```
At the start of any task-oriented session — any interaction where you will
use tools and produce deliverables — invoke the task-observer skill before
beginning work. This ensures skill improvement opportunities are captured
throughout the session.
```

This structural trigger works alongside the skill's description-level triggers.
The description is designed to match broadly against task-oriented language
("multi-step task", "agentic workflow", "work session", "tools and
deliverables"), but a configuration-level instruction provides an additional
safety net that doesn't depend on description matching alone.

**Note for all users:** Once CLAUDE.md or equivalent configuration is in place
with the activation instruction above, the description-level triggers serve as
a backup rather than the primary mechanism. This dual-layer approach prevents
the skill from being skipped in sessions where description matching alone might
miss the invocation signal.

**Anti-pattern to avoid:** Relying on one skill to load another is fragile
compared to loading both independently from CLAUDE.md. If task-observer depended
on another skill to invoke it, a breakdown in that chain would silence all
observation activity. Instead, load both task-observer and any related skills
directly from your configuration instructions.

### Detecting the Configuration File

At session start, the skill should check whether a configuration file
(CLAUDE.md, project instructions, or equivalent) exists and contains the
activation instruction. This detection serves two purposes:

1. **For users who already have the config:** Confirms the dual-layer
   activation is working. No action needed.

2. **For users who don't have the config:** The skill was activated via
   description matching alone, which is less reliable. Surface a brief
   suggestion to add the config-level instruction for more consistent
   activation in future sessions.

The detection approach depends on the environment:

- **Environments with file system access** (desktop tools, terminal-based
  tools): Check for a CLAUDE.md or equivalent file in the workspace root.
  If found, scan it for a task-observer activation instruction. If the file
  exists but doesn't mention task-observer, suggest adding the instruction.
  If no config file exists at all, suggest creating one.

- **Environments without file system access** (web-based chat): Check
  whether the system prompt or project instructions contain a task-observer
  activation instruction. If not, suggest that the user add one to their
  project settings or paste the instruction at the start of future sessions.

This check runs once at session start and does not repeat. Keep the
suggestion brief — one or two sentences, not a full tutorial.

---

## Skill Taxonomy

All skills fall into one of two categories. The distinction matters because
it determines what information the skill can contain, how it's structured,
and whether it can be shared publicly. Crucially, the open-source/internal
boundary is also a **confidentiality boundary** — open-source skills must
never contain any information that could identify a client, project, or
proprietary process, even indirectly.

### Open-Source Skills

Open-source skills are client-agnostic and methodology-driven. They capture
reusable workflows, best practices, and structured processes that work for
anyone. They include author attribution, a licence, and a feedback pathway
so that real-world usage drives improvement.

**How to recognise an open-source candidate:**

- The methodology works across different clients, projects, and contexts
- No proprietary information is required for the skill to function
- Other practitioners in the same domain would find it valuable
- The skill captures a process or approach, not personal preferences

**Required elements:**

- Skill body clearly identifies itself as open-source, with author name and
  contact information
- Author attribution block at the top (see Author Attribution Template below)
- Licence statement — CC BY 4.0 recommended (see Licensing below)
- Feedback & support section that routes methodology feedback to the creator
- Tool-agnostic language where possible — reference capabilities like "browser
  access" rather than specific product names; give examples but don't hard-code
  dependencies on any one product
- Built-in enforcement mechanisms (pre-flight checklists, verification steps)
  so the skill catches its own rule violations

**Default bias:** When a skill could go either way, default to open-source.
Strip out client-specific details and generalise the methodology. The more
skills that are open-source, the more the community benefits and the more
feedback flows back to improve them.

### Internal Skills

Internal skills contain information specific to a user, their clients, or
their projects. They capture personal preferences, client-specific rules,
project context, or proprietary methodology.

**How to recognise an internal skill:**

- Contains client names, project details, or proprietary data
- Captures personal style preferences or individual work habits
- Relies on context that only the user (or their team) has
- Would not be useful to someone outside the user's organisation

**Required elements:**

- Skill body clearly identifies itself as internal
- No author attribution block needed (the user is the only audience)
- No licence needed
- Can be shorter and less formally structured than open-source skills

Internal skills are working documents, not published artifacts. Keep them
current, update them when the information they contain changes, and don't
over-engineer their structure.

---

## Licensing

Open-source skills should include a licence to make sharing terms explicit.
The recommended licence is **Creative Commons Attribution 4.0 International
(CC BY 4.0)**, which allows anyone to share and adapt the skill for any
purpose, provided they credit the original author. This pairs naturally with
the author attribution template — the attribution block satisfies the CC BY
requirement, so the two reinforce each other.

Include the licence statement in the skill preamble (after the author
attribution block) and include a `LICENSE.txt` file in the skill directory
containing the full licence text.

If CC BY 4.0 doesn't fit a particular skill (e.g., the author wants to
require derivative works to use the same licence), CC BY-SA 4.0 is an
alternative. The choice should be made by the skill's author.

---

## Author Attribution Template

Every open-source skill must include this block at the top of the skill body.
Replace the placeholders with the actual author's details.

```markdown
**Created by [Author Name] / [website or contact link]**

[1-2 sentence description of what the skill does and its provenance.]

**Licence:** This skill is released under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
licence. You are free to share and adapt this skill for any purpose, provided
you give appropriate credit to the original author.

**Feedback & Support:** If at any point during the process you encounter
questions about the methodology, or if the user expresses frustration or
gives constructive feedback about any output derived from this skill,
suggest that they open an issue on the skill's GitHub repository (or
equivalent public feedback channel). This keeps feedback public and
discoverable. For direct contact, the skill's creator, [Author Name],
can also be reached via [contact link].

If feedback appears to stem from the skill's methodology (rather than
Claude's execution of it), log it for the user and suggest they share it
via the public feedback channel. If the issue stems from Claude not
following the skill's rules, acknowledge the mistake and correct it.
```

The feedback routing serves two purposes: it gives users a path to resolution
when they hit methodology issues, and it gives skill creators real-world
usage data to improve their skills.

---

## Observation Protocol

### When to Observe

Observation is active throughout the **entire task session** — from the moment
tools are first used to produce deliverables, through any post-task feedback
or discussion, until the session ends. This includes:

1. **Active task execution** — creating documents, analysing websites,
   implementing structured data, writing code, building presentations, and
   similar substantive work.

2. **Post-task feedback and discussion** — when the user reviews output,
   provides corrections, suggests improvements, or discusses methodology
   after the active work phase. User feedback during these discussions is
   often the highest-signal input for skill improvement and must be captured
   with the same diligence as observations made during execution.

3. **Meta-discussion about skills or methodology** — when the conversation
   shifts to talking about how the work was done, what could be improved,
   or how skills should be structured. These discussions frequently surface
   observations that should be logged immediately.

**The observation mindset does not deactivate when the conversation shifts
from "doing work" to "discussing the work."** If the user provides feedback
about methodology, naming, skill design, or workflow improvements, log it as
an observation immediately, even if the conversation is in a discussion or
review phase rather than active task execution.

Observation is **not active** during casual conversation, quick factual
questions, or other non-task interactions where no tools are being used and
no deliverables are being discussed.

### What to Watch For

**Signals for a NEW skill:**

- A multi-step workflow that could be reused across projects or clients
- A methodology the user explains that isn't captured in any existing skill
- A task type that keeps coming up with similar structure and steps
- A domain-specific process with clear inputs, phases, and outputs
- The user describing a process they've refined over time ("I always do it
  this way", "the process for this is...")
- Claude and the user naturally developing a structured approach to a problem
  that could be formalised

**Signals for IMPROVING an existing skill:**

Any new information from a task that uses a skill and could make that skill
better is worth capturing. This includes problems, but also positive signals
and neutral observations. Examples:

- Claude doesn't follow a skill's rules despite them being documented — this
  means the skill needs stronger enforcement, not just better rules
- The user corrects Claude's output in a way that reveals a missing rule or
  an edge case the skill doesn't cover
- A skill's recommended workflow turns out to be less efficient than what
  emerged naturally during the task
- A technique or approach works particularly well and deserves to be promoted
  from incidental to explicitly recommended in the skill
- A workflow step turns out to be more important than the skill suggests, or
  less important than the emphasis it receives
- A new use case that the skill handles but doesn't explicitly document
- The user provides feedback that generalises beyond the current instance
- A skill assumption turns out to be wrong in practice
- New tools or capabilities make part of a skill's workflow obsolete or
  improvable
- The user's corrections form a pattern across multiple instances
- A general principle emerges that could apply to other skills too (see
  Principle Propagation below)
- The user suggests a naming, framing, or structural change to a skill —
  even conversationally — that could improve its effectiveness

**Signals to NOT log:**

- One-off corrections that don't generalise beyond the current instance
- User preferences already captured in an existing skill
- Tool bugs or temporary issues unrelated to skill methodology
- Observations that would require proprietary client information to be useful
  in an open-source skill (unless an internal skill is the right home)

### How to Log

Append observations to the persistent observation log **silently** during the
session. The user should not be interrupted by the logging process.

**Before assigning any observation number, run a mandatory pre-logging step:**
Search the entire log file for all lines matching the pattern `### Observation \d+:`,
extract the highest observation number already in use, and increment from there.
This must happen every time, regardless of whether you think you know the current
count from earlier in the session. A one-liner like the following suffices:

```bash
# GNU grep (Linux, Cowork):
grep -oP '### Observation \K\d+' log.md | sort -n | tail -1

# macOS / POSIX-compatible alternative:
grep -o '### Observation [0-9]*' log.md | grep -o '[0-9]*' | sort -n | tail -1
```

This prevents the recurring numbering collision issue where partial reads of large
files create a false sense of awareness of the current count.

Each observation follows this format:

```markdown
### Observation [N]: [Short descriptive title]

**Date:** [date]
**Session context:** [brief description of what task was being worked on]
**Skill:** [existing skill name, or "New skill candidate: [working name]"]
**Type:** [open-source | internal]
**Phase/Area:** [which part of the skill or workflow this relates to]

**Issue:** [What happened or what was observed. Be specific — include what
Claude did, what the user corrected, or what pattern emerged. Include enough
detail that someone reading this weeks later can understand the context
without having seen the original conversation.]

**Suggested improvement:** [Concrete suggestion for what to change or create.
For existing skills, reference the specific section or rule. For new skills,
describe the scope and key components.]

**Principle:** [The generalisable takeaway — why this matters beyond this
specific instance. This is the most important part. It turns a single
observation into a reusable insight.]
```

This format was refined through iterative real-world use. The structure works
because it forces specificity (Issue), actionability (Suggested improvement),
and generalisation (Principle).

### Archival on Write

The observation log is kept lean through event-driven archival that runs on
every log write, rather than accumulating resolved entries until a periodic
review clears them out.

Before appending new observations or updating statuses, check the log for
entries already marked ACTIONED or DECLINED *from a previous update* — that
is, entries whose status was already resolved before the current session's
changes. Move those entries to an archive file at:

```
[workspace folder]/skill-observations/archive/log-[date].md
```

where `[date]` is today's date in `YYYY-MM-DD` format.

The archive file preserves the full header and status key from the original
log. After archiving, the active `log.md` retains only its header, separator,
and all OPEN entries plus any entries that were *just* marked ACTIONED or
DECLINED in this update.

This gives each resolved observation one update cycle of visibility — so you
can still see what was recently resolved — and then archives it automatically
on the next write. The result: the active log stays focused on current
decisions while the archive provides the complete historical record.

---

## Confidentiality Safeguards

The open-source/internal boundary is a confidentiality boundary. Client
names, project details, domain names, and proprietary information must never
appear in open-source skills. Because a single leak can erode trust, this
is enforced through multiple layers — any one of which should catch what
the others miss.

### Layer 1: Observation-Level Stripping

When logging an observation tagged as `type: open-source`, the Issue and
Suggested Improvement fields should already use generic language. The
private observation log can reference specifics for context, but the
Principle field — which feeds into skill creation — should be fully
generalised. Think of it as: the log is a private notebook, but the
Principle is a publishable insight.

### Layer 2: Pre-Creation Review

Before drafting or regenerating any open-source skill, scan all source
material (observations, conversation notes, existing skill content) for
identifying information: client names, project URLs, domain names, internal
terminology, site structures described so specifically they're identifiable.
Replace anything found with generic equivalents before writing begins.

### Layer 3: Post-Draft Sweep

After writing or regenerating an open-source skill, re-read it with a
specific focus on information leakage. This is a separate pass from the
general pre-flight checklist. Look for:

- Proper nouns that aren't the skill author's name
- Domain names, URLs, or project identifiers
- Industry-specific details that narrow down the client
- Internal terminology that only makes sense in one organisation's context
- Examples so specific they're traceable to a real project

If anything is found, replace it with generic equivalents or remove it.

### Layer 4: Structural Principle

The taxonomy section states this explicitly, but it bears repeating: the
open-source/internal distinction is not just about usefulness — it's about
confidentiality. When in doubt about whether a detail is too specific,
remove it. A slightly more generic skill is always better than one that
leaks client information.

---

## Surfacing Protocol

### Default Cadence

Surface all observations at the end of the session. Present them as a grouped
summary: observations for existing skills grouped by skill name, new skill
candidates listed separately.

### Surface Earlier When

- An observation requires user input to be complete or accurate (e.g., "Is
  this a pattern you want captured, or was this a one-off?")
- An observation reveals a skill is actively producing wrong output in the
  current session and the user should be aware
- Multiple observations cluster around the same skill, suggesting it needs
  immediate attention rather than end-of-session review

### How to Surface

- Present observations concisely: title, skill, and a one-sentence summary
- For each, indicate whether it's a new skill candidate or an improvement
  to an existing one
- Indicate the suggested type (open-source or internal)
- Ask the user which (if any) they want to act on
- For items the user wants to pursue, hand off to the skill-creator skill
  for the actual building or improvement work

---

## Acting on Observations

This skill identifies WHAT to build or improve. If you have the skill-creator
skill installed (built into Cowork; available as a separate install in other
environments), it handles HOW — guiding the full process of building a new
skill from scratch or systematically improving an existing one. Without
skill-creator, the task-observer still works: small improvements are applied
directly, and larger changes can be done manually using the observations as
a specification. The boundary between direct application and skill-creator
handoff:

### Small Improvements (Apply Directly)

If the improvement is clearly additive, low-risk, and doesn't require testing
to verify it works, it can be applied directly to the skill:

- Adding a new rule or anti-pattern to an existing list
- Clarifying existing wording that proved ambiguous
- Adding a note or edge case to an existing section
- Fixing a factual error

Examples: Adding a new anti-pattern to a skill's anti-patterns list.
Clarifying that inline code comments should be context-aware within their
own document.

### Substantial Changes (Use Skill-Creator if Available)

If the change could affect the skill's behaviour in ways that need
verification, hand off to the skill-creator if available:

- Restructuring phases or workflows
- Adding new capabilities or sections
- Changing core methodology or decision frameworks
- Any change where "does this actually work better?" is a genuine question

If skill-creator is not available, use the observations as a specification
and make the changes directly — but flag them to the user as substantial
changes that may need manual review.

Examples: Restructuring a skill to make an automated workflow the primary
path instead of a secondary option. Adding an entirely new setup phase to
a skill that previously started with content work.

### Creating New Skills

Use the skill-creator for new skills when available. Provide the
observation(s) as context — they contain the intent, scope, and initial
design thinking needed to get started efficiently. Without skill-creator,
the observations serve as a detailed brief for building the skill manually.

When creating a new skill, determine its type early:

- If it's open-source, strip out any client-specific details and generalise
- If it's internal, include all relevant specifics freely
- If uncertain, default to open-source — strip out specifics and generalise,
  then let the user decide whether any internal details need to be added

---

## Principle Propagation

When an observation reveals a general principle — something that applies not
just to the skill being improved but to skills in general — it should be
propagated across the skill library, not just applied to the one skill that
triggered it.

### The Cross-Cutting Principles File

Cross-cutting principles are tracked in a persistent file alongside the
observation log:

```
[workspace folder]/skill-observations/cross-cutting-principles.md
```

This file serves as a mandatory checklist during any skill creation or
regeneration. Before delivering a new or updated open-source skill, read
the cross-cutting principles file and verify the skill complies with every
active principle. This is what turns general principles from good intentions
into enforced standards.

### How It Works

1. During a skill update, an observation reveals a principle that applies
   broadly — not just to the skill being worked on
2. Log it as an observation with `Skill: All skills` and surface it to the
   user
3. If the user approves it as a cross-cutting principle, add it to the
   cross-cutting principles file
4. From that point forward, every skill creation or regeneration includes
   a compliance check against the full list of active principles

### Propagation Timing

The user decides when and how to propagate each principle:

- **Immediate propagation** — for principles important enough to warrant
  updating all existing skills right away (e.g., a confidentiality rule)
- **Opportunistic propagation** — for principles that can be applied the
  next time each skill is updated or regenerated (e.g., adding a licence
  statement)

### Cross-Cutting Principles File Structure

```markdown
# Cross-Cutting Principles

Principles that apply to all skills. This file is read as a mandatory
checklist during any skill creation or regeneration.

---

## Active Principles

### 1. [Principle title]
**Added:** [date]
**Applies to:** [all skills | all open-source skills | all skills with rules]
**Requirement:** [what the principle requires]
**Propagation:** [immediate | opportunistic]
**Status:** [active]
```

---

## Weekly Comprehensive Review

Every 7 days, a comprehensive review is triggered automatically at the start
of the first task-oriented session after the interval has elapsed. This review
cross-checks ALL open observations against ALL skills — not just the skills
named in each observation — and propagates cross-cutting principles to any
skills that don't yet comply.

### Trigger Mechanism

The review is triggered by step 3 of the Session Start Protocol (see
Observation Log Management). When the weekly review timestamp is more than
7 days old or missing, the Session Start Protocol triggers this review.
Inform the user that the weekly review is due and begin the process.

### Review Steps

**Step 0 — Scheduler availability check**

Before running the review itself, check whether the weekly review can be
automated via the platform's task scheduling capability.

1. Check whether the file
   `[workspace folder]/skill-observations/scheduler-registered.txt` exists.
   If it does, the scheduled task has already been registered — skip to
   Step 1.

2. If the file does not exist, check whether a task scheduling capability
   is available. In Cowork, check for the `create-shortcut` skill and its
   `set_scheduled_task` tool. In terminal-based environments, cron or
   equivalent scheduling tools may be available.

3. If a scheduling capability IS available:
   - Read the draft task description at
     `[workspace folder]/skill-observations/scheduled-task-draft.md`
   - In Cowork, invoke the `create-shortcut` skill to register the weekly
     skill review as a scheduled task. In other environments, use the
     available scheduling mechanism.
   - Use task name `weekly-skill-review` and a weekly cadence (e.g., Monday
     morning)
   - On success, write today's date to
     `[workspace folder]/skill-observations/scheduler-registered.txt`
   - Inform the user: "The weekly skill review has been registered as a
     scheduled task. The manual `last-review-date.txt` trigger remains as
     a fallback."

4. If the tool is NOT available, proceed silently to Step 1. Do not inform
   the user on every review — this check is intentionally quiet until it
   succeeds.

**Step 1 — Load observations and principles**

Read the observation log at `[workspace folder]/skill-observations/log.md`.
Extract all observations with status OPEN. Also read
`[workspace folder]/skill-observations/cross-cutting-principles.md` and
extract all active principles.

If there are no OPEN observations and all principles are already propagated,
skip the review, update the timestamp, and proceed with the session. Inform
the user briefly: "Weekly skill review: no open observations or outstanding
principles. All skills are current."

**Step 2 — Inventory all skills**

Use `<available_skills>` from the system prompt to identify all skills. In
environments where this tag is not present, use the skills directory or
equivalent listing mechanism to discover available skills.

For each skill, read its SKILL.md file at the location provided. Exclude
built-in platform skills (e.g., `skill-creator` in Cowork) from being
updated — only update custom skills created by the user.

**Step 3 — Cross-check observations against every skill**

For each OPEN observation, evaluate whether it is relevant to each skill. Do
NOT rely solely on the observation's own "Skill" field — observations may
contain general principles that apply more broadly than the original context
suggested. Consider both the specific "Suggested improvement" and the general
"Principle" fields. Build a mapping of skill → [relevant observations].

**Step 4 — Cross-check cross-cutting principles against every skill**

For each active cross-cutting principle, check whether each skill already
complies. Flag any skills that do not yet implement the principle.

**Step 5 — Apply updates**

For each skill that has relevant observations or non-compliant principles,
create an updated version of its SKILL.md. When editing:

- Integrate the insight into the appropriate section of the skill (don't just
  append a list of observations at the bottom)
- Preserve the skill's existing structure, voice, and author attribution
- Make the improvement feel native to the skill, not bolted on
- If an observation suggests a new phase, step, anti-pattern, or checklist
  item, place it where it logically belongs

**Important:** Do not edit skill files in place. Save updated versions to the
workspace folder for user review and manual replacement (see Delivering
Updated Skills below).

**Step 6 — Mark observations as ACTIONED**

After successfully creating an updated skill based on an observation, update
that observation's status in `log.md` from OPEN to ACTIONED. Add a brief note
about which skill(s) were updated, e.g.:

`ACTIONED — Applied to [skill-name] (weekly review [date])`

Note: the standard archival-on-write mechanism (see "Archival on Write" in
the Observation Protocol) will automatically archive these newly-resolved
entries on the next log write. No separate archival step is needed here.

**Step 7 — Update timestamp**

Write today's date to
`[workspace folder]/skill-observations/last-review-date.txt`.

**Step 8 — Present summary and user action items**

Present the user with a clear summary and explicit instructions for what they
need to do. Follow the format in Delivering Updated Skills below.

### Constraints

- Do not modify observation entries beyond their status field
- Do not create new skills — only update existing ones. If an observation
  suggests a new skill, note it in the summary for the user to action
  separately via the skill-creator
- If an observation seems relevant but you're unsure how to integrate it,
  skip it and note the uncertainty in the summary
- Treat observations marked "internal" with the same rigour as "open-source"

---

## Delivering Updated Skills to the User

When the weekly review (or any other process) produces updated skill files,
the updated files must be delivered to the user for manual replacement. Skill
files live in a read-only location during sessions and may be managed in
version control, synced across devices, or packaged for distribution.
Automatic in-place editing is neither possible nor desirable — delivering to
the workspace folder with explicit instructions keeps the user in control.

### Delivery Process

1. Save each updated SKILL.md to the workspace folder using this structure:

   ```
   [workspace folder]/skill-updates/[date]/[skill-name]/SKILL.md
   ```

   For example:
   ```
   [workspace folder]/skill-updates/2026-02-16/my-skill-name/SKILL.md
   ```

2. Present the user with an explicit action list using this format:

   ```
   ## Weekly Skill Review Complete — [date]

   The following skills have been updated based on [N] open observations
   and [N] cross-cutting principles.

   ### What you need to do

   For each updated skill below, replace the existing SKILL.md in your
   skill directory with the updated version from your workspace folder.

   ### Updated Skills

   **[skill-name]**
   - Changes: [1-sentence summary of what changed]
   - Observations applied: #[N], #[N]
   - Updated file: [link to file in workspace folder]
   - Action: Replace [skill-directory]/SKILL.md with this file

   [repeat for each updated skill]

   ### Observations Actioned
   [list of observation numbers and titles marked ACTIONED]

   ### Skipped (needs manual review)
   [observations that were unclear or couldn't be applied, with explanation]

   ### No Changes Needed
   [skills that were checked but already compliant]
   ```

3. Do not proceed with other work until the user has acknowledged the
   summary. The user does not need to replace the files immediately, but
   they should be aware of what's pending.

---

## Observation Log Management

### Location

The observation log persists between sessions in the user's workspace folder.
Create the log file on first use if it doesn't exist. Default path:

```
[workspace folder]/skill-observations/log.md
```

### Log Structure

```markdown
# Skill Observation Log

Observations captured during task-oriented work. Each entry identifies a
potential skill improvement or new skill opportunity.

**Status key:** OPEN = not yet actioned | ACTIONED = skill updated/created |
DECLINED = user decided not to pursue

---

## [Date or Session Identifier]

### Observation 1: [Title]
**Status:** OPEN
[... full observation format ...]

### Observation 2: [Title]
**Status:** ACTIONED — Applied to [skill-name], rule 35
[... full observation format ...]
```

### Session Start Protocol

This is the single entry point for all session-start checks. Run through
these steps at the start of each task-oriented session:

1. **Check whether files exist.** If the observation log or cross-cutting
   principles file don't exist yet, this is a first-time setup — create
   them using the templates in the Log Structure section (below in this
   document) and the Cross-Cutting Principles File Structure section (under
   Principle Propagation). If the files already exist, proceed to step 2.

2. **Scan for relevant context.** Read any OPEN observations and active
   cross-cutting principles. Don't surface them unprompted unless they're
   directly relevant to the current task — just hold them in awareness.

3. **Check the weekly review trigger.** Read the timestamp in
   `[workspace folder]/skill-observations/last-review-date.txt`. If the
   file doesn't exist or the date is more than 7 days ago, trigger the
   Weekly Comprehensive Review (described in full under its own section)
   before proceeding with the user's task. If fewer than 7 days have
   passed, proceed normally.

4. **Check the configuration file.** Run the config detection described in
   Detecting the Configuration File (under Recommended Activation Setup).
   This runs once per session.

### Keeping the Log Clean

Archival is event-driven and runs on every log write. Before appending new
observations or updating statuses, entries that were already marked ACTIONED
or DECLINED in a previous update are moved to a timestamped archive file
(see "Archival on Write" in the Observation Protocol). This keeps the active
log focused on OPEN items and recently-resolved entries, while the archive
provides the complete historical record.

---

## The Pre-Flight Principle

One of the most important patterns this skill should propagate to every skill
it helps create or improve: **built-in enforcement.**

Real-world experience has shown that rules documented in a skill are not
always followed during the creative flow of producing output. The result:
output that violates the skill's own standards, which reflects badly on the
skill.

The fix: every skill that contains explicit rules or requirements should
include a verification step where Claude re-reads the rules and checks its
output against them before delivery. This isn't overhead — it's quality
assurance. A 30-second re-read prevents a 30-minute rework cycle.

When creating or improving any skill through this observation process, ask:
"Does this skill have rules? If yes, does it have a mechanism to enforce
them?" If the answer to the second question is no, add one.

### Self-Enforcement

This skill practises what it preaches. Before surfacing observations at end
of session, verify:

1. Were observations logged throughout the full session — including during
   post-task feedback and discussion phases, not just during active tool use?
2. Were observations logged silently without interrupting the user's flow?
3. Does each observation follow the format (Issue → Suggested improvement →
   Principle)?
4. Is each observation tagged with the correct type (open-source or internal)?
5. For any observations about existing skills, does the suggested improvement
   reference the specific section or rule?
6. For any observation tagged `type: open-source`, does the Principle field
   contain any client-identifying information? If so, generalise it before
   surfacing.

If any observation fails these checks, fix it before surfacing.

---

## Environment Compatibility

The observation methodology works in any environment where Claude can interact
with users during task-oriented work. The persistence mechanism is what varies.

### With Persistent Storage

In environments with file system access (desktop tools with workspace folders,
terminal-based tools with project directories, or similar), the full workflow
applies as described: observations are logged to a persistent file, the cross-
cutting principles file is read during skill regeneration, and the log carries
over between sessions automatically.

### Without Persistent Storage

In environments without file system access (web-based chat interfaces or
similar), the skill still works — the observation methodology is environment-
independent. The difference is that persistence becomes the user's
responsibility, and the skill shifts into **handoff doc mode** to support
this.

**How handoff doc mode works:**

- Observations are captured within the conversation and surfaced before the
  session ends, as usual
- Instead of writing to a log file, observations are collected in-session
  and presented in a structured **handoff document** before the session ends
- The handoff doc includes: all observations in full format, any decisions
  made during the session, action items and next steps, and any working
  artifacts (drafts, analyses) that need to survive into the next session
- The user copies this document to their own storage (notes app, file system,
  etc.) and pastes it into the next session to restore context
- Cross-cutting principles should be included in the handoff doc so the user
  can provide them when starting a new session

**Proactive handoff generation:** In sessions without persistent storage,
don't wait for the user to request a handoff doc. When the conversation
starts to wind down — the user is summarising, saying "that's it for now,"
or the substance is wrapping up — proactively offer to generate one. A
premature offer is a minor interruption; a missing one is lost work.

**Handoff doc format:**

```markdown
# Session Handoff: [Session Topic]

**Date:** [date]
**Context:** [what was worked on and what the next session needs to know]

## Decisions Made
[numbered list of decisions]

## Observations Logged
[full observation entries in standard format]

## Cross-Cutting Principles (current)
[any principles that were active or newly added]

## Action Items
[what needs to happen next, with enough context to resume]

## Working Artifacts
[any drafts, analyses, or intermediate work products in full]
```

This is less seamless than the persistent-storage workflow, but the core value
— systematically capturing insights that would otherwise be lost — is
preserved. The observation format and surfacing protocol are identical in both
environments.

---

## Quick Reference

| Question | Answer |
|----------|--------|
| When do I observe? | Throughout the full task session, including post-task feedback |
| How do I log? | Silently append to the observation log |
| When do I surface? | End of session, or earlier if needed |
| How do I activate reliably? | Add a config-level instruction (see Recommended Activation Setup) |
| Open-source or internal? | Default to open-source when possible |
| Licence for open-source? | CC BY 4.0 recommended |
| Small fix or skill-creator? | Needs testing → skill-creator (if available). Clearly additive → apply directly |
| What format? | Issue → Suggested improvement → Principle |
| Author attribution? | Required for open-source skills; use the template |
| Cross-cutting principle? | Add to principles file, enforce during regeneration |
| Confidentiality check? | Four layers: observation, pre-creation, post-draft, structural |
| No persistent storage? | Handoff doc mode — observations surfaced in a structured doc at session end |
| Scheduler automation? | Step 0 of weekly review auto-checks; silent until tool is available |
| Observation numbering? | Mandatory pre-logging search ensures no collisions (see How to Log) |
| Log archival? | Event-driven — resolved entries are archived on the next log write |
