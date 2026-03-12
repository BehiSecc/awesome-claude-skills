---
name: lean-ux
description: 'Apply lean thinking to UX: hypothesis-driven design, collaborative sketching, and rapid experiments instead of heavy deliverables. Use when the user mentions "Lean UX", "design hypothesis", "UX experiment", "collaborative design", or "outcome over output". Covers hypothesis statements, MVPs for UX, and cross-functional collaboration. For Build-Measure-Learn, see lean-startup. For usability audits, see ux-heuristics.'
license: MIT
metadata:
  author: wondelai
  version: "1.0.0"
---

# Lean UX Framework

A practice-driven approach to user experience design that replaces heavy deliverables with rapid experimentation, cross-functional collaboration, and continuous learning. Based on a fundamental truth: teams that obsess over pixel-perfect specs before testing with real users waste months building the wrong thing. Lean UX shifts the question from "What should we design?" to "What do we need to learn?"

## Core Principle

**Outcomes over outputs.** The value of a design is not measured by the fidelity of the deliverable but by the change in user behavior it produces.

**The foundation:** Traditional UX waterfalls requirements into wireframes, wireframes into mockups, mockups into specs, and specs into code. At every handoff, context is lost and assumptions go untested. Lean UX eliminates waste by compressing the distance between idea and evidence. Instead of debating opinions in conference rooms, teams declare assumptions, form hypotheses, run the smallest possible experiment, and let real user behavior settle the argument. Shared understanding replaces documentation. Learning velocity replaces pixel perfection.

## Scoring

**Goal: 10/10.** When reviewing or creating UX processes, design plans, or team workflows, rate them 0-10 based on adherence to Lean UX principles. A 10/10 means full alignment with hypothesis-driven design, minimal deliverables, collaborative practices, and outcome-focused metrics; lower scores indicate heavy-deliverable thinking or untested assumptions. Always provide the current score and specific improvements needed to reach 10/10.

### 1. Declaring Assumptions

**Core concept:** Every design starts with assumptions. Lean UX makes those assumptions explicit so they can be prioritized and tested, rather than baked invisibly into specifications.

**Why it works:** When assumptions remain unspoken, teams build on shaky ground and discover problems only after launch. By surfacing assumptions early, the team can focus energy on the riskiest ones first, reducing the cost of being wrong.

**Key insights:**
- Business assumptions define what must be true for the business to succeed (revenue model, market size, willingness to pay)
- User assumptions define who the users are, what they need, and what behaviors they exhibit
- Assumption prioritization is based on two axes: risk (how damaging if wrong) and uncertainty (how little we know)
- High-risk, high-uncertainty assumptions are tested first
- The team writes assumptions collaboratively, not in isolation

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **New feature kick-off** | Assumption mapping workshop | "We assume users want to share reports with teammates" |
| **Redesign initiative** | Identify what you believe about current users | "We assume users leave because the dashboard is confusing" |
| **Roadmap planning** | Rank features by assumption risk | Prioritize features whose success depends on untested beliefs |
| **Stakeholder alignment** | Expose hidden assumptions across roles | PM assumes pricing works; engineer assumes scale works; designer assumes flow works |

**Ethical boundary:** Assumptions should be honest assessments, not post-hoc justifications for decisions already made. If leadership has already committed to a direction, acknowledge that constraint rather than pretending the assumption is open to falsification.

See: [references/hypothesis-canvas.md](references/hypothesis-canvas.md)

### 2. Hypothesis Statements

**Core concept:** A hypothesis translates an assumption into a testable prediction. The Lean UX hypothesis format links a proposed change to a measurable outcome for a specific user segment.

**Why it works:** Hypotheses force precision. Instead of "make onboarding better," the team commits to a specific prediction that can be proven or disproven. This prevents scope creep, sharpens success criteria, and makes the learn step unambiguous.

**Key insights:**
- Standard format: "We believe [outcome] will happen if [persona] achieves [action] with [feature]"
- Every hypothesis should specify the persona, action, outcome, and measurable signal
- Sub-hypotheses break a large bet into smaller, independently testable parts
- Hypotheses are not goals; they are predictions that could be wrong
- The team must agree on what "validated" and "invalidated" look like before running an experiment

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **Feature design** | Write hypothesis before wireframing | "We believe trial-to-paid conversion will increase by 10% if new users complete a guided setup wizard" |
| **A/B tests** | Formalize test rationale | "We believe click-through will rise 15% if we move the CTA above the fold" |
| **Sprint planning** | Attach hypothesis to each user story | Story: "As a user I can filter by date." Hypothesis: "We believe task completion time drops 30%" |
| **Retrospectives** | Review validated vs. invalidated hypotheses | "3 of 5 hypotheses validated this quarter; 2 pivoted" |

**Ethical boundary:** Never cherry-pick metrics after the fact to declare a hypothesis validated. Pre-commit to success criteria.

See: [references/hypothesis-canvas.md](references/hypothesis-canvas.md)

### 3. MVPs and Experiments

**Core concept:** An MVP in Lean UX is the smallest design artifact that can test a hypothesis with real users. It is not a product launch; it is a learning tool.

**Why it works:** Heavy deliverables delay learning. A paper prototype tested with five users in a hallway can invalidate a hypothesis that would otherwise consume a full sprint of engineering. By matching experiment fidelity to the risk of the assumption, teams learn faster and waste less.

**Key insights:**
- Experiment types range from low fidelity (paper prototypes, concierge tests) to high fidelity (coded A/B tests, Wizard of Oz)
- Choose the lowest-fidelity experiment that can answer the question
- A good experiment has a clear hypothesis, defined audience, measurable signal, and time box
- "Proto-personas" can stand in for full research when speed matters, but must be validated later
- The goal is to learn, not to ship

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **Early concept validation** | Paper prototype or clickable mockup | Sketch 3 concepts, test with 5 users same day |
| **Demand validation** | Landing page smoke test | "Sign up for early access" measures real interest |
| **Usability validation** | Clickable prototype test | Figma prototype tested with 5-8 users |
| **Technical feasibility** | Wizard of Oz | Manual backend, automated frontend to test experience |
| **Pricing validation** | Painted door test | Show pricing page, measure click-through before building billing |

**Ethical boundary:** Smoke tests and fake door tests must not mislead users into believing a product exists when it does not. Always disclose the test status and offer a way to opt out.

See: [references/experiment-patterns.md](references/experiment-patterns.md)

### 4. Collaborative Design

**Core concept:** Design is a team sport. Lean UX replaces the solitary designer-then-handoff model with cross-functional design sessions where developers, product managers, and designers sketch solutions together.

**Why it works:** When the whole team participates in design, shared understanding replaces documentation. Developers who helped sketch the solution do not need a 40-page spec to build it. Diverse perspectives generate more creative solutions. Handoff waste drops dramatically.

**Key insights:**
- Design Studio method: diverge (individual sketching), present, critique, converge (refined sketch), iterate
- Shared understanding is the currency of Lean UX; it replaces heavy documentation
- Style guides and pattern libraries are living documents, not static PDFs
- The goal is not consensus but informed commitment: the team agrees on what to test, not what is "right"
- Cross-functional participation means engineers, QA, data analysts, and stakeholders sketch too
- Reduce UX deliverables to the minimum needed for shared understanding (often a whiteboard photo)

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **Sprint kick-off** | Design Studio session (90 minutes) | Whole team sketches solutions to the sprint's hypothesis |
| **Feature exploration** | Collaborative sketching workshop | 6-up sketches: each person draws 6 ideas in 5 minutes |
| **Design system maintenance** | Living style guide updates | Engineers and designers update the guide together as they build |
| **Remote teams** | Virtual whiteboard sessions | FigJam or Miro board with timed sketch rounds |

**Ethical boundary:** Collaboration must not become design by committee. A designated designer synthesizes input; the team does not vote on pixels.

See: [references/collaborative-design.md](references/collaborative-design.md)

### 5. Feedback and Research

**Core concept:** Continuous, lightweight research replaces big-bang usability studies. Lean UX embeds research into every sprint so teams learn from real user behavior constantly rather than quarterly.

**Why it works:** Feedback that arrives months after a design decision is too late to influence it. By running small research activities every sprint, teams correct course incrementally. The cost of each research activity is low, so the team can afford to test frequently.

**Key insights:**
- Research types: usability tests (5 users), customer interviews, A/B tests, analytics review, surveys, diary studies
- Five users uncover approximately 85% of usability problems (Nielsen)
- Continuous research cadence: recruit weekly, test weekly, synthesize weekly
- Research is not a phase; it is an ongoing activity embedded in every sprint
- The whole team should observe at least some research sessions to build empathy
- Proto-personas are refined and eventually replaced by evidence-based personas

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **Weekly usability testing** | Test prototype with 3-5 users every Thursday | "Testing Thursday" ritual with rotating facilitators |
| **Post-launch learning** | Monitor analytics + run 3 follow-up interviews | Identify drop-off points, interview users who churned |
| **Persona validation** | Compare proto-persona assumptions to interview data | "We assumed power users are marketers; data shows they are ops managers" |
| **Competitive research** | Lightweight competitive teardown each quarter | Team reviews 3 competitors for 30 minutes, captures patterns |

**Ethical boundary:** User research must be conducted with informed consent. Participants should understand how their data will be used and have the right to withdraw.

See: [references/experiment-patterns.md](references/experiment-patterns.md)

### 6. Integration with Agile

**Core concept:** Lean UX is designed to work inside Agile development. Dual-track agile separates discovery (learning what to build) from delivery (building it), running both tracks in parallel.

**Why it works:** Traditional UX struggles in Agile because design work does not fit neatly into a sprint. Dual-track solves this by running discovery one sprint ahead of delivery. The discovery track generates validated hypotheses and tested prototypes; the delivery track turns them into shippable software.

**Key insights:**
- Dual-track agile: discovery track (research + design) feeds the delivery track (engineering + QA)
- Discovery runs one sprint ahead, so validated designs are ready when the delivery sprint begins
- Staggered sprints prevent the "sprint zero" anti-pattern where design is always catching up
- User stories gain a hypothesis and success metric alongside acceptance criteria
- "Definition of Done" for UX includes validated learning, not just shipped pixels
- Backlog items from invalidated hypotheses are removed, not deferred

**Product applications:**

| Context | Application | Example |
|---------|-------------|---------|
| **Sprint planning** | Include hypothesis validation in sprint goals | "Sprint goal: validate that inline editing reduces task time by 20%" |
| **Backlog refinement** | Attach experiment results to stories | Story moves to delivery only after hypothesis is validated |
| **Retrospectives** | Review learning velocity alongside delivery velocity | "We validated 4 hypotheses and invalidated 2 this sprint" |
| **Roadmap updates** | Adjust roadmap based on experiment outcomes | Invalidated feature removed from Q3 roadmap |

**Ethical boundary:** Do not use Lean UX as an excuse to skip accessibility, security, or compliance work. These are non-negotiable quality standards, not assumptions to be tested.

See: [references/agile-integration.md](references/agile-integration.md)

## Common Mistakes

| Mistake | Why It Fails | Fix |
|---------|-------------|------|
| **Treating MVPs as launches** | Team over-builds because they conflate "minimum viable product" with "first release" | Reframe: MVP = learning tool, not product launch |
| **Skipping assumption declaration** | Hidden assumptions become expensive surprises | Run a 30-minute assumption mapping session at kick-off |
| **Hypothesis without success criteria** | Cannot determine if experiment passed or failed | Pre-commit to metric, threshold, and sample size |
| **Designer-only design** | Handoff waste, misalignment, slow iteration | Run Design Studio sessions with the full cross-functional team |
| **Research as a phase** | Feedback arrives too late to influence decisions | Embed lightweight research into every sprint |
| **Ignoring invalidated hypotheses** | Team builds features that failed testing | Remove invalidated items from backlog; pivot or drop |
| **Documenting instead of collaborating** | 40-page specs nobody reads | Replace specs with shared understanding from collaborative sessions |
| **Measuring outputs not outcomes** | Shipping features that do not change behavior | Define success as behavior change, not feature delivery |

## Quick Diagnostic

Audit any UX process or design plan:

| Question | If No | Action |
|----------|-------|--------|
| Are assumptions explicitly declared? | Hidden assumptions drive decisions | Run assumption mapping workshop |
| Is there a testable hypothesis? | Team is building on opinion | Write hypothesis in standard format before designing |
| Is the experiment the lowest fidelity that can answer the question? | Over-investing before learning | Downgrade to paper prototype or smoke test |
| Does the whole team participate in design? | Handoff waste and misalignment | Schedule a Design Studio session |
| Is research happening every sprint? | Feedback loop is too slow | Establish weekly testing cadence |
| Are you tracking outcomes, not just outputs? | Shipping without learning | Define behavior-change metrics for each feature |
| Does UX work feed into Agile smoothly? | Design bottleneck or sprint zero trap | Implement dual-track agile with staggered sprints |
| Can you point to a hypothesis you invalidated recently? | Team is not learning; confirmation bias | Review experiment log and celebrate a pivot |

## Reference Files

- [hypothesis-canvas.md](references/hypothesis-canvas.md): Hypothesis statement format, assumption prioritization matrix, business vs. user assumptions, sub-hypotheses
- [experiment-patterns.md](references/experiment-patterns.md): UX experiment types, choosing the right experiment, experiment design template, minimum viable tests
- [collaborative-design.md](references/collaborative-design.md): Design Studio method, collaborative sketching, cross-functional design, living style guides
- [agile-integration.md](references/agile-integration.md): Dual-track agile, fitting UX into sprints, staggered sprints, Definition of Done for UX
- [outcome-metrics.md](references/outcome-metrics.md): Outcomes vs. outputs, leading vs. lagging indicators, OKRs for UX, vanity metrics to avoid
- [case-studies.md](references/case-studies.md): Enterprise product team, startup, agency, and internal tools team scenarios

## Further Reading

This skill is based on Lean UX principles developed by Jeff Gothelf and Josh Seiden. For the complete methodology, research, and case studies:

- [*"Lean UX: Designing Great Products with Agile Teams"*](https://www.amazon.com/Lean-UX-Designing-Great-Products/dp/1098116305?tag=wondelai00-20) by Jeff Gothelf & Josh Seiden
- [*"Sense and Respond"*](https://www.amazon.com/Sense-Respond-Successful-Organizations-Continuously/dp/1633691888?tag=wondelai00-20) by Jeff Gothelf & Josh Seiden (scaling outcome-focused thinking across organizations)

## About the Authors

**Jeff Gothelf** is an organizational designer, coach, and author who helps companies build better products and cultivate outcome-focused cultures. He spent over 15 years as a UX designer and team leader at agencies and product companies, including TheLadders, Publicis Modem, and Neo Innovation (now Pivotal Labs). His experience watching teams waste months on unvalidated deliverables led him to develop Lean UX as a practical fusion of design thinking, Agile development, and lean startup principles. Gothelf coaches Fortune 500 companies and speaks internationally on product management, organizational agility, and evidence-based design.

**Josh Seiden** is a designer, product strategist, and coach with over 25 years of experience helping teams build digital products. He co-founded the interaction design practice at Cooper, one of the first UX consultancies, and later served as Managing Director at Neo Innovation. Seiden specializes in helping organizations shift from output-driven to outcome-driven ways of working. Together with Gothelf, he co-authored *Lean UX* and *Sense and Respond*, both of which have become essential reading for product teams adopting Agile and Lean practices.
