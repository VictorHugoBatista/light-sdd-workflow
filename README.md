# Light SDD Workflow

## Main phases
### SPEC Phase (AKA upstream)
This phase reflects the upstream phase on an agile flow. It will start with your specifications and will finish with epics and stories ready to run. Use a small LLM optimized for text processing and organization, but **keep humans in control of all business logic and architecture decisions**. The SPEC phase is where you capture what the system *must* do and *why*, establishing the foundation for all downstream work.

```mermaid
timeline
    Writing the SPEC.md file: Write the file by hand or in pair with a LLM : Separate the specs in development phases : Define your quality gates : Use a LLM to review and search for inconsistencies and contradictions
    Generating the PRD.md file : Use the SPEC.md to generate this file : Use a pattern to the LLM to follow : The result must to be epics and stories well defined : Each story must to have its own quality gates
    Generating the PRD.json file : File used for the ralph-tui on development phase : It MUST to reflect the PRD.md file, but in a simpler way
```

#### Writing the SPEC.md file

Your SPEC.md file is the **contract between stakeholders and your development team**. It defines business rules, system architecture, quality gates, and constraints—the critical decisions that humans must own and verify, never delegate entirely to LLMs. Think of it as a conversation with your team that answers: "What problem are we solving? What are the system's boundaries? What must never break?"

A good specification is **complete, consistent, unambiguous, and verifiable**. Write requirements using imperative language ("the system **shall** process payments within 5 seconds") rather than soft language ("the system should be fast"). Avoid superlatives, subjective phrases, or vague terms like "user-friendly" or "as needed"—these invite interpretation and later disputes about whether requirements are met. Instead, be measurable: "display results in under 2 seconds" vs "display results quickly."

Structure your SPEC.md with sections for:
- **Purpose & Overview** — What problem does this system solve? Who uses it?
- **Business Rules** — Policies, constraints, workflows that govern behavior (especially important for LLMs to understand later)
- **System Architecture** — High-level design, key components, data flow (use mermaid diagrams for flowcharts, sequence diagrams, state machines for complex logic)
- **Quality Gates** — What does "done" mean? Define acceptance criteria, performance targets, security requirements, testing approach *upfront*
- **Assumptions & Constraints** — What are we NOT building? What dependencies exist?

Use diagrams liberally. Flowcharts clarify complex business logic, sequence diagrams show interactions between components, state machines capture transitions (e.g., order states: pending → confirmed → shipped → delivered). These visual specifications are invaluable for LLMs—they provide concrete examples of what "correct" behavior looks like.

**Separate your specs by development phases/epics early**. If you're building an e-commerce system, grouping specs as "Auth & User Management," "Product Catalog," "Shopping Cart," "Payments" makes the downstream task easier. When you generate the PRD later, this separation naturally becomes your epic structure.

**Use an LLM to review your specs for inconsistencies, contradictions, and missing requirements**. Feed the LLM your draft SPEC.md and ask: "Are there any contradictions in these business rules? Any requirements that conflict with each other? What's missing?" This catches ambiguities before they become expensive bugs.

#### Generating the PRD.md file

Once your SPEC.md is solid, use it to generate a detailed PRD (Product Requirements Document). Provide the LLM with a template or pattern to follow—structure, format, and level of detail. The Light SDD workflow works well with a pattern that organizes stories into epics, with each story including description, acceptance criteria, and quality gates.

Your PRD.md must reflect your SPEC.md exactly—if your SPEC defines "payment processing must complete in under 5 seconds," that requirement appears in the PRD either as an acceptance criterion for a story or as a non-functional requirement across the epic. This traceability ensures nothing gets lost between phases.

Keep stories small and independently verifiable. A good user story is something one developer can complete and test in a few hours to a day. Large stories hide complexity and delays feedback. Each story must have clear acceptance criteria—concrete, testable conditions that define "done" (e.g., "Given a user in the cart, when they click Checkout, then the payment page loads with order details pre-filled and a payment form").

**Include an architecture story at the end of each epic**, dedicated solely to updating README and ARCHITECTURE files with what you've learned. After implementing an auth epic, for example, document the authentication patterns, data structures, and decision rationale so future developers and LLMs understand your approach.

Review the PRD.md with your team before moving to the next phase. Get alignment on scope, priorities, and acceptance criteria. This review is your last chance to catch misunderstandings cheaply—once development starts, changes become expensive.

#### Generating the PRD.json file

The PRD.json file is the execution format for ralph-tui. It distills your PRD.md into a simpler structure: epics, stories, steps, and quality gates. The JSON must accurately reflect the PRD.md — if a detail appears in the PRD but not in the JSON, developers might miss it. Think of it as the "runbook" version of the PRD: clear, structured, and directly executable.

Each story in the JSON should include: title, description, acceptance criteria, steps to implement (if known), quality gates to validate, and any dependencies on other stories. Ralph-tui reads this file and presents stories one at a time, tracking progress and ensuring nothing is skipped.

Review the PRD.json against the PRD.md side-by-side. Verify every requirement from the PRD appears somewhere in the JSON. Check for consistency—if two stories mention the same business rule, confirm they align. Once you've agreed on the JSON, it becomes the source of truth for development.

### Development phase (AKA downstream)
#### Developement
```mermaid
flowchart LR
    1@{ shape: sm-circ, label: "Small start" }
    2@{ shape: subproc, label: "Review" }
    a("Run a story with LLM")
    b("Review and test (if possible)")
    c{"Is there some error?"}
    d("Fix errors (by hand or with LLM)")
    e{"Are there more stories on epic?"}

    1 --> a
    subgraph Development
        a --> b
        b ---> c
        c --->|"Yes"| d
        d --> c
        c --->|"No"| e
        e --->|"yes"| b
    end

    e -->|"no"| 2
```

This is the moment where you assist the LLM to run all the plan prepared on SPEC phase. If you are using the ralph-tui application, you can run the commnad `ralph-tui run --prd prd.json` for starting the ui.

You can run all the stories automatically and test only in the end, but this has a potential to take more time than save it. Just prefer to run one small story by time and review / test when possible.

Create a script that runs build/lint/security/scan/automated tests at once. Then, you will need to run only one command to test the project integrity after running a story. If possible, test manually too. **Be sure your product works**.

If some fix be necessary, you can fixit or ask a LLM to fix for you. Again, small LLMs are doing a great job on this minor fixes and refactor to me.

Repeat this proccess until all stories be finished.

#### Review
```mermaid
flowchart LR
    1@{ shape: subproc, label: "Development" }
    2@{ shape: framed-circle, label: "Stop" }
    f("Create a pull request")
    g("Add copilot as reviewer / review with your human pairs")
    h("Resolve the comments by hand or with LLM")
    i("Make sure all the quality gates still are ok")

    1 --> f
    subgraph Review
        f --> g --> h --> i
    end
    i --> 2
```
There's no much to say about the review phase. Just create your pull request as you always done. But, if you use github, you can ask copilot to review the code for you. Ask your pairs to review it too, **humans are not obsolete**.

If copilot return something relevant, you can solve it or ask a small LLM to solve for you. Again, be sure about your project integrity, run all your checks after each change and test manually if possible.

Now your ready to restart the cycle on the next epic.

## Epic workflows
```mermaid
stateDiagram
    classDef spec fill:yellow
    classDef dev color: white,fill:green

    class specPhaseComplete spec
    class specPhaseOnyEpicByTime spec
    class oneEpicByTime dev
    class oneStoryByTime dev
    class developEpic dev
    class developStory dev

    specPhaseComplete: 1- Write the specs for all your epics
    oneEpicByTime: 2- Run an epic on a separated branch
    oneStoryByTime: 3- Run a epic story by time

    specPhaseOnyEpicByTime: 1- Write the spec for one epic a time
    developEpic: 2- Develop the written epic
    developStory: 3- Develop one epic story by time

    state AllIn {
        specPhaseComplete --> oneEpicByTime : 1
        oneEpicByTime --> oneStoryByTime : 2
        oneStoryByTime --> oneStoryByTime : 3
        oneStoryByTime --> oneEpicByTime : 4
        oneEpicByTime --> oneEpicByTime : 5
    }

    state AgileSdd {
        specPhaseOnyEpicByTime --> developEpic : 1
        developEpic --> developStory : 2
        developStory --> developEpic : 3
        developStory --> developStory : 4
        developEpic --> specPhaseOnyEpicByTime : 5
        specPhaseOnyEpicByTime --> specPhaseOnyEpicByTime : 6
    }
```

### AllIn Workflow
This workflow is ideal when you have a clear vision of the entire system from the start. You begin by writing the specifications for all your epics at once (`specPhaseComplete`, step 1). Then, for each epic, you create a separate branch and develop it independently (`oneEpicByTime`, step 2). Within each epic, you work through the stories one by one (`oneStoryByTime`, step 3), iterating on them until the epic is complete. After finishing an epic, you move on to the next, repeating the process. This approach is structured and works well for greenfield projects or when requirements are stable and well-understood.

### AgileSdd Workflow
This workflow is more incremental and adaptive, suitable for projects where requirements may evolve or are not fully known upfront. Here, you write the specification for one epic at a time (`specPhaseOnyEpicByTime`, step 1). You then develop that epic (`developEpic`, step 2), working through its stories individually (`developStory`, step 3). After completing a story, you can either continue with the next story or revisit the epic for adjustments. Once the epic is done, you return to the specification phase for the next epic. This cycle allows for continuous feedback and adaptation, making it ideal for agile teams and projects with changing needs.

## Notes
### What is ralph loop and the ralph-tui and why use it
The **ralph-tui** is a task orchestration and execution tool designed for the Development phase of the Light SDD Workflow. It reads your `prd.json` file (the JSON representation of your Product Requirements Document) and provides an interactive UI for running and tracking user stories one at a time. You invoke it with `ralph-tui run --prd prd.json`, which loads all your defined stories and guides you through their execution with built-in progress tracking. This tool is what makes the Development phase practical—it eliminates manual bookkeeping of which stories are done, in-progress, or blocked.

The **ralph loop** is the iterative development cycle that ralph-tui facilitates: **run story → review and test → detect errors → fix errors → repeat**. Rather than running all stories at once and testing at the end (which leads to discovering many bugs simultaneously and delays feedback), the ralph loop emphasizes incremental execution. After each story completes, you immediately review, test, and fix issues before moving to the next story. This approach catches errors early when they're fresh in context, reduces integration surprises, and keeps the codebase in a testable state throughout development.

```mermaid
flowchart LR
    Start("📖 Story from ralph-tui")
    Run("▶ Run story with LLM<br/>(code generation)")
    Review("🔍 Review output &<br/>test if possible")
    ErrorCheck{"❌ Issues<br/>found?"}
    Fix("🔧 Fix errors<br/>(by hand or LLM)")
    NextStory{"📋 More<br/>stories?"}
    Complete("✅ Epic complete")
    
    Start --> Run
    Run --> Review
    Review --> ErrorCheck
    ErrorCheck -->|Yes| Fix
    Fix --> ErrorCheck
    ErrorCheck -->|No| NextStory
    NextStory -->|Yes| Start
    NextStory -->|No| Complete
    
    style Start fill:#e1f5ff
    style Run fill:#fff3e0
    style Review fill:#f3e5f5
    style ErrorCheck fill:#ffebee
    style Fix fill:#ffe0b2
    style NextStory fill:#e8f5e9
    style Complete fill:#c8e6c9
```

Why use this? Because **integration and testing frequency dramatically reduces risk and effort**. Developers using the ralph loop catch semantic conflicts (where code compiles but doesn't work logically) immediately after a story completes, not weeks later during final integration. This short feedback loop means less rework, clearer error context, and—critically—keeps both the codebase and your confidence in code quality high throughout development.

### The importance of automated tests and security scans
Automated tests and security scans are the foundation of maintaining quality gates throughout the Light SDD Workflow. Every story completion should trigger a consistent, automated verification: **does the code build? Do tests pass? Are there security vulnerabilities?** This prevents the workflow's biggest risk: shipping broken or insecure code because defects weren't caught early.

In Light SDD, create a single script (e.g., `test.sh` or equivalent) that chains together: linting, build compilation, unit tests, integration tests, and security scanning. You can use libs/applications like `mocha` for testing and `opensecurity/njsscan` for SAST (Static Application Security Testing), so your script might run `mocha test/**/*.js` followed by `njsscan --output json .` to catch logic errors and security flaws in one pass. After completing each story with the ralph loop, is a good practice to run this single command once—if it passes, you know the story is solid (and never forget about the linter and its importance to maintain a pattern in the written code); if it fails, you fix it immediately with fresh context. This is far more efficient than discovering dozens of test failures after all stories complete.

The specific types of checks matter: **unit tests** verify individual functions work correctly, **integration tests** confirm different components work together, and **security scans** detect patterns like hardcoded secrets, injection vulnerabilities, or dependency risks before code reaches production. By placing these in an automated script run after each story, you align your development practice with the workflow's core principle—continuous verification and early error detection. Documentation of your automated test approach should be in your ARCHITECTURE file so future developers understand the testing strategy.

### Green fields vs brown fields
**Greenfield** development means building a new system from scratch—you have no existing codebase to work with, requirements are typically well-defined upfront, and you can architect systems optimally from the beginning. Greenfield projects are ideal for the **AllIn workflow** in Light SDD: write specifications for all epics first, then develop them in sequence. There are no legacy constraints, no existing code to understand, and no integration points with mature systems. Examples: building a new microservice, a new product line, or a complete rewrite of a system.

**Brownfield** development means working within an existing system—you inherit legacy code, architectural decisions you didn't make, and constraints from production systems. Brownfield projects benefit from the **AgileSDD workflow**: write specifications one epic at a time, develop and deliver each epic, then repeat. Why? Because existing systems often have hidden requirements that only surface once you start working in them. By cycling through spec → development → delivery one epic at a time, you discover these constraints early and adapt your approach incrementally rather than midway through a months-long development cycle. Examples: adding features to a mature application, maintaining legacy systems, or integrating with existing infrastructure.

The Light SDD Workflow structure (with AllIn and AgileSDD variants) explicitly acknowledges this distinction because greenfield and brownfield have fundamentally different risk profiles. Greenfield risk is *completeness and architectural correctness*—did you design the system right? Brownfield risk is *integration safety and unforeseen constraints*—does your new code break existing functionality? By choosing the workflow that matches your context, you address the right risks at the right time.

### The importance of README and ARCHITECTURE files on AI written projects
When using AI assistance in development, documentation becomes your **bridge of understanding**—both for the LLM and for future developers. A README that explains "what this system does and why" and an ARCHITECTURE file that captures "how it's built and why we chose this design" are not optional polish; they're essential inputs for effective AI collaboration.

LLMs work best with clear context. When you ask an LLM to implement a story, it needs to understand your system's design patterns, business rules, and data flow. Without this, LLMs generate code that might work technically but violates your system's conventions, duplicates logic, or creates hidden bugs. A well-maintained ARCHITECTURE file describing your core patterns, data structures, and key decisions provides the grounding LLMs need to generate coherent, consistent code. Your Light SDD workflow's emphasis on including an **"architecture story" at the end of each epic**—dedicated to updating README and ARCHITECTURE files—reflects this necessity: after each epic, you capture what you've learned about the system's design, making that knowledge available for the next epic's LLM assistance.

Documentation also protects you from vendor lock-in or LLM dependency. Six months from now, if you need to refactor or extend code, a well-documented ARCHITECTURE file lets you (or another developer) understand intent without reconstructing it from code alone. For AI-written projects especially, this documentation is your insurance: it ensures knowledge survives beyond the LLM session and remains accessible to humans.

## Resources

### SPEC Phase
- [Wikipedia - Software Requirements Specification](https://en.wikipedia.org/wiki/Software_requirements_specification) — Comprehensive overview of SRS standards, structure, requirement quality characteristics, and IEEE/ISO standards (IEEE 830, ISO/IEC/IEEE 29148)
- [TechWhirl - Writing Software Requirements Specifications](https://techwhirl.com/writing-software-requirements-specifications/) — Practical guide on SRS templates, requirement quality indicators, avoiding ambiguous language, establishing traceability matrices, and best practices for writing unambiguous requirements

### Development Phase & Ralph Loop
- [Martin Fowler - Patterns for Managing Source Code Branches](https://martinfowler.com/articles/branching-patterns.html) — Understanding greenfield vs brownfield development contexts, branching strategies for sustainable development, and integration frequency patterns
- [Martin Fowler - Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — High-frequency integration practices, testing strategies, and maintaining code quality through automation—directly aligned with the ralph loop approach

### Notes (Quality Gates & Testing)
- [Mocha Testing Framework](https://mochajs.org/) — JavaScript test framework for unit and integration testing, used in this project for story validation
- [NJSScan by OpenSecurity](https://github.com/ajinabraham/njsscan) — Static security scanner for Node.js applications, detects vulnerabilities and code quality issues as part of automated quality gates
- [Continuous Integration Best Practices](https://martinfowler.com/articles/continuousIntegration.html) — Testing, automation, and high-frequency integration fundamentals that support the ralph loop and quality gates
