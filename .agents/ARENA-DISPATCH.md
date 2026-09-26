# Arena Dispatch

Compile decided work into a self-sufficient GitHub Issue for Arena.

Do not assume Arena automatically reads `AGENTS.md` or has access to ChatGPT conversation history, model memory, or unstated human intent. Include task-critical context and explicitly name any repository files it must read.

The Issue must contain the task-critical context Arena needs and point precisely to repository sources it must read.

Do not dispatch unresolved product or architecture decisions. If a missing decision could materially change intent, resolve it before assigning the Issue to Arena.

## Planning inputs

Dispatch may compile work directly from settled human intent or from planning artifacts produced by capabilities such as `to-spec`, `to-tickets`, or `wayfinder`.

Those artifacts do not replace this dispatch contract.

- `to-spec` may produce the durable specification that dispatch uses as a source.
- `to-tickets` may decompose larger work into executable slices.
- A ticket produced by `to-tickets` is not Arena-ready until this dispatch procedure has compiled and validated its execution context.
- Prefer enriching the existing implementation ticket rather than creating a duplicate Arena Issue.
- Planning capabilities remain ChatGPT-side. Do not instruct Arena to load `to-spec`, `to-tickets`, `grilling`, or `wayfinder`.

Arena assignment is the final dispatch gate regardless of any upstream tracker label or readiness terminology.

## Compilation rules

- Restate task-critical decisions, constraints, and expected outcomes in the Issue.
- Point to canonical repository sources instead of duplicating their full contents.
- Name exact repository paths in `Read first`.
- Name the exact adopted Arena capabilities to load.
- Use observable acceptance criteria.
- Provide concrete verification commands or checks where known.
- Include likely files or directories only when reasonably knowable.
- Include out-of-scope boundaries only when they materially constrain the work.
- Do not require Arena to infer product or architecture intent.

Routine implementation choices within decided boundaries may be left to Arena.

If Arena discovers missing, contradictory, or ambiguous product or architecture intent that could materially affect the result, it must stop and surface the blocker rather than invent a decision.

## Execution routing

Execution environment is part of the work order. Arena must distinguish security-audit execution from ordinary implementation/build verification instead of treating limitations of its development sandbox as repository blockers.

- **Live capability check:** within the selected capability's safety boundaries, probe the current session before choosing an execution route. Distinguish a tool that is not preinstalled from one that cannot be installed safely, and a registry's reachable front page from an artifact that can actually be downloaded. A failed request to one distribution host is evidence about that route only: consider safe, reputable CLI/API, signed package-manager, verified mirror, or user-local acquisition routes before concluding a toolchain cannot be obtained. **Separate toolchain acquisition from project dependency acquisition:** a working runtime does not establish a compiler or build tool; signed package metadata, a manually seeded cache, or a locally copied JAR/classpath does not establish a downloaded SDK, Maven/Gradle resolver, remote dependency resolution, or project build. Record which verified artifact was actually obtained and which bounded proof ran. Never weaken TLS, signatures, checksums, or package verification to expand the probe. Check whether a canonical repository workflow exists and is accessible before routing work to it. Record untested paths as unknown rather than generalizing from another session.
- **Security-audit boundary:** when `security-audit` is loaded, any target-controlled build, test, process, emulator, browser, fuzzer, or fixture execution used as audit evidence must obey the security-audit skill's sandbox contract. Static source inspection remains read-only. Do not substitute an ordinary GitHub-hosted runner for a required audit sandbox unless the selected capability explicitly permits an equivalent environment and every required control can be enforced. If the audit controls cannot be enforced, record the exact `needs_validation` blocker rather than weakening them.
- **Implementation and CI work:** outside that audit-evidence boundary, verify locally when the required tools are available or can be installed safely. When local verification is not viable, prefer an existing, trusted canonical repository runner if Arena can access and execute its workflow. Do not assume a runner or workflow exists, or that Arena has permission to use it. If neither route is available, report the exact unverified checks and blocker; missing preinstalled tools alone do not prove the repository implementation is broken.
- **Supply-chain regeneration:** when a reviewed dependency or build-tool change requires lockfile, checksum, verification-metadata, generated manifest, or similar network-dependent regeneration that the development sandbox cannot perform, Arena may use a temporary branch-scoped GitHub Actions workflow. It must use immutable action pins, minimum permissions, no repository secrets unless explicitly required by the Issue, write only the intended generated artifacts, verify the generated state under the repository's normal strict checks, and be removed before merge unless the Issue explicitly adopts it as permanent infrastructure.
- **Repository settings:** if a required GitHub setting needs repository-administration permission that Arena does not have, establish that with one authoritative permission/API check and report it as an owner action. Do not churn workflow YAML or weaken checks to work around an administration-only setting.
- **Final acceptance:** repository implementation acceptance is based on the repository's actual canonical verification surface, when one exists, plus any explicitly required local checks. A local Arena-sandbox success is not a substitute for required CI; when CI does not exist or is inaccessible, report the verification gap rather than assuming CI passed. A local failure caused only by missing sandbox tooling is not by itself evidence that the repository implementation is broken.
- **Mixed audit + implementation Issues:** when one Issue contains both a security audit and subsequent implementation, its `Special execution requirements` must state this routing explicitly: audit evidence stays within the security-audit sandbox contract; implementation/build verification moves to the repository runner where appropriate after the audit reaches its required terminal state.

## Issue structure

Use this core structure for every Arena Issue.

```md
# <task title>

## Objective

<The single outcome Arena is being asked to produce.>

## Read first

- `<path>` — <what Arena must take from this source>
- `<path>` — <what Arena must take from this source>

## Capabilities to load

- `<capability>` — `<path>`
- `<capability>` — `<path>`

## Task

<Concrete description of the work to perform.>

<Include all task-critical decisions and constraints already settled by the human and ChatGPT.>

## Acceptance criteria

- <observable condition>
- <observable condition>

## Verification

- `<command or check>` — <what it proves>
- `<command or check>` — <what it proves>

## Completion

Return a pull request containing the completed work.

In the PR or completion report, include:

- what changed,
- verification performed and its results,
- any deviations from the Issue,
- any unresolved concerns,
- the PR reference.
```

## Conditional sections

Add these only when materially useful.

### Files likely to change

Use when the expected implementation surface is reasonably knowable.

```md
## Files likely to change

- `<path or directory>` — <expected reason>
```

This is guidance, not a prohibition on touching another file when the task legitimately requires it.

### Out of scope / Do not change

Use when explicit boundaries are needed.

```md
## Out of scope / Do not change

- <boundary>
```

### Additional context

Use when Arena needs concise task-specific facts that do not already have a better canonical source.

Do not use this section as a substitute for `Read first` or for recording durable project truth in its owning file.

### Special execution requirements

Use when the selected capability or task requires additional execution constraints, such as:

- agreed test seams,
- prototype questions,
- migration sequencing,
- security-audit scope,
- compatibility requirements,
- required ordering or rollback constraints.

## Pre-dispatch check

Before assigning the Issue to Arena, confirm:

- the objective is decided,
- required repository sources are named,
- selected Arena capabilities are explicit,
- task-critical decisions are stated,
- acceptance criteria are observable,
- verification is defined,
- meaningful boundaries are included,
- execution routing is explicit when the task mixes security-audit work with implementation/build verification or depends on runner-only tooling,
- no unresolved product or architecture decision has been delegated accidentally.

If any of these cannot be satisfied because human intent is still unresolved, return to decision-making instead of dispatching.

## Arena blocker rule

If execution reveals an ambiguity or contradiction that could materially change product or architecture intent:

1. Stop before making the speculative decision.
2. Preserve completed work that does not depend on the unresolved decision.
3. Report the exact blocker, the conflicting evidence, and the decision required.
4. Do not substitute an assumption for the missing decision.
