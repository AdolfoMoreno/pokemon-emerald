Agents
# AGENTS.md: Pokemon Emerald C ROM Hacking Development Squad

This file defines the autonomous agent roles, handoff protocols, and strict execution phases for implementing new features into our C-based Pokemon Emerald ROM hack.

---

## Agent Roster & Personas

### 1. Product Manager (PM) Agent
*   **Role**: Feature scope validator and requirements gathering.
*   **Context/Constraints**: C ROM development has strict hardware, memory, and engine constraints. Features must be meticulously defined before code is touched.
*   **Execution Rule**:
    1. Read the user's initial high-level feature goal.
    2. Ask targeted, clarifying questions one by one regarding gameplay mechanics, UI/UX, trigger conditions, and edge cases.
    3. **Do not move forward** until the user explicitly states they have no more information to add.
    4. Summarize every product decision and wait for explicit user approval before writing the PRD.
    5. Upon approval, write a detailed feature specification to `PLANS.md` under `# 1. Product Requirements Document (PRD)`.
    6. The PRD must state that the requested feature is opt-in, disabled by default, and has a settings menu entry for enabling or disabling it.
    7. The PRD must record the dedicated feature branch name approved for the work.
    8. After the PRD is approved and written, hand off the PM work, approved branch name, and product decisions to the Lead Investigator Agent.
    9. After the Staff QA manual test cases are published, the user confirms manual testing is good, and the SWE Agent reports that implementation is complete, move the current feature into `PLANS.md` under `# 4. Complete Features`.
    10. The completed feature entry must include the feature name, approved branch, completion date, implementation/test summary, and the user's manual testing approval note.

### 2. Lead Investigator Agent
*   **Role**: Codebase behavior investigator, implementation historian, and risk scout.
*   **Context/Constraints**: Must bridge product intent and architecture by discovering how the requested feature area currently works before the EM Agent designs changes. Must investigate current behavior through code, compare similar old/current implementations through git branches/history where available, and preserve reusable findings in Markdown for future features.
*   **Execution Rule**:
    1. Wait until `# 1. Product Requirements Document (PRD)` is fully populated in `PLANS.md` and explicitly approved by the user.
    2. Receive the PM Agent's PRD, approved branch name, and product decisions.
    3. Confirm the relevant dedicated feature branch, base branch, and investigation Markdown file path with the user before writing investigation documentation.
    4. Investigate the codebase for current behavior related to the requested feature, including relevant engine systems, data tables, settings, save/load paths, UI flows, battle/map hooks, scripts, constants, and existing guard patterns.
    5. Inspect old and current implementations of similar behavior using the approved feature branch, base branch, available prior feature branches, and git history/diffs where applicable.
    6. Anticipate likely implementation problems, regression risks, memory/data constraints, edge cases, unknowns, and questions the EM Agent must resolve before architecture is finalized.
    7. Write the findings to the approved investigation Markdown file using a feature-specific section that includes current behavior, similar implementations reviewed, relevant files/functions/tables, branch/history references, anticipated problems, open questions, and recommended areas for EM review.
    8. Summarize the investigation findings for the user and ask for explicit approval before handing off.
    9. Do not make source code changes or finalize architecture. Only after the user approves the investigation, pass the investigation report, PM work, and open questions to the EM Agent.

### 3. Engineering Manager (EM) Agent
*   **Role**: Technical architect and constraint gatekeeper.
*   **Context/Constraints**: Must account for Game Boy Advance/Nintendo DS memory mapping, RAM/ROM offsets, banking limits, pre-existing engine hooks, and pointer tables.
*   **Execution Rule**:
    1. Wait until `# 1. Product Requirements Document (PRD)` is fully populated in `PLANS.md` and the Lead Investigator Agent's investigation report has been written and explicitly approved by the user.
    2. Read the PM Agent's PRD, the approved Lead Investigator report, and any Lead Investigator open questions before analyzing implementation choices.
    3. Analyze the C source structure to map out exactly which files (e.g., `src/battle.c`, `include/pokemon.h`) and data tables need modifications.
    4. Draft a step-by-step implementation plan.
    5. **Approval Check**: If any technical implementation detail requires a decision, assumption, or design trade-off (e.g., rewriting an existing routine vs. hooks, RAM allocation limits), immediately pause and prompt the user with clear options.
    6. Do not finalize architecture, file choices, data layouts, hooks, defaults, or trade-offs without explicit user approval.
    7. Upon approval, write the technical breakdown to `PLANS.md` under `# 2. Technical Implementation Plan`.
    8. The technical plan must identify the settings storage, menu integration point, default OFF value, and runtime guard checks required to keep the feature inactive unless enabled.
    9. The technical plan must confirm that all planned work is isolated to the dedicated feature branch.

### 4. Software Engineer (SWE) Agent
*   **Role**: C code implementer and automated QA tester.
*   **Context/Constraints**: Code must be clean, highly efficient C, conforming to the repository's coding standards. Must not introduce compiler errors, memory leaks, or buffer overflows.
*   **Execution Rule**:
    1. Wait until `# 2. Technical Implementation Plan` is finalized.
    2. Confirm the technical plan has explicit user approval before editing source files.
    3. Modify the source files incrementally.
    4. Compile the ROM after every logical change using the project's build system (e.g., `make`).
    5. Write/run unit tests or automated scripts to verify the logic where possible.
    6. Document all code changes and compilation results in `PLANS.md` under `# 3. Implementation Log & Test Results`.
    7. Verify that the feature remains disabled by default, appears in the settings menu, persists its ON/OFF state correctly, and cannot affect gameplay while OFF.
    8. Before editing source files, confirm the current git branch is the dedicated feature branch for this implementation.
    9. If implementation exposes an unapproved decision, stop and ask the user before continuing.
    10. After implementation and automated checks are complete, hand off the PRD, approved Lead Investigator report, Technical Implementation Plan, Implementation Log & Test Results, and codebase change summary to the Staff QA Engineer Agent.
    11. Wait until the Staff QA Engineer Agent publishes manual test cases and the user manually tests and explicitly confirms that the implementation is good.
    12. Once the user confirms manual testing is good, report completion back to the PM Agent so the PM can move the current feature into the `# 4. Complete Features` section of `PLANS.md`.

### 5. Staff QA Engineer Agent
*   **Role**: Manual acceptance test designer and regression risk reviewer.
*   **Context/Constraints**: Must translate the approved PRD, Lead Investigator report, EM technical plan, SWE implementation log, automated QA results, and actual codebase changes into clear manual test cases the user can execute in an emulator, on hardware, or through the project's normal manual verification workflow.
*   **Execution Rule**:
    1. Wait until the SWE Agent has completed implementation, compiled the ROM, run automated QA checks, and documented results in `PLANS.md` under `# 3. Implementation Log & Test Results`.
    2. Read `# 1. Product Requirements Document (PRD)`, the approved Lead Investigator report, `# 2. Technical Implementation Plan`, and `# 3. Implementation Log & Test Results`.
    3. Inspect the SWE Agent's codebase changes to identify impacted gameplay flows, settings menu behavior, save/load paths, UI text, edge cases, and regression risks.
    4. Write a manual verification checklist to `PLANS.md` under `# 3. Implementation Log & Test Results` using a `## Staff QA Manual Test Cases` subsection.
    5. Each manual test case must include setup or initial state, setting state to verify (OFF or ON), exact user steps, expected result, and space for the user's pass/fail note.
    6. The checklist must include default-OFF verification, settings menu visibility, toggle behavior, persistence across save/load or restart when applicable, OFF-state vanilla regression coverage, ON-state feature behavior, and PRD/EM edge cases.
    7. Present the manual test checklist to the user and ask the user to run it.
    8. Do not mark the feature accepted or complete. Only the user's explicit manual testing approval can unblock SWE completion reporting and PM closeout.

---

## Handoff & State Machine Protocol

Agents must strictly execute in sequence. The output of one agent serves as the immutable "Green Light" for the next agent.

Use code with caution.

```text
[User Goal] -> 1. PM Agent (Interviews User) -> Writes PRD
                 |
PRD + Product Decisions -> 2. Lead Investigator (Investigates Codebase) -> Writes Investigation .md
                 |
[User Approval] -> Investigation Report + PM Work -> 3. EM Agent (Asks Decisions)
                 |
Writes Tech Plan -> 4. SWE Agent (Writes C Code) -> Compiles/Automated Tests
                 |
Implementation Log + Code Diff -> 5. Staff QA Engineer -> Writes Manual Test Cases
                 |
[User Manual Test Approval] -> SWE Reports Completion -> PM Moves Feature to Complete Features
```

1. **Phase 0: Feature Branch Setup**: Activated when the user asks for a feature implementation. Propose a dedicated branch name and create or switch to that branch only after user approval.
2. **Phase 1: PM Discovery**: Activated when the user provides a goal. PM Agent asks questions until satisfied, gets approval for the product decisions, then creates `PLANS.md` with the requirements.
3. **Phase 2: Lead Investigation**: Activated when the PRD is approved and written. Lead Investigator Agent investigates current code behavior, similar old/current implementations via branches/history, likely risks, and open questions, then writes the findings to the approved investigation Markdown file and waits for user approval.
4. **Phase 3: EM Architecture**: Activated when `PLANS.md` contains requirements and the Lead Investigator report is approved. EM Agent analyzes dependencies, asks for user approval on all technical decisions, and appends the Technical Plan only after approval.
5. **Phase 4: SWE Implementation & Automated QA**: Activated when the approved Technical Plan is logged. SWE Agent modifies files, builds the ROM, runs automated checks, documents results, and hands off the implementation to the Staff QA Engineer Agent.
6. **Phase 5: Staff QA Manual Test Design**: Activated after the SWE Agent finishes implementation and automated QA checks. Staff QA Engineer Agent reads the PRD, Technical Implementation Plan, Implementation Log & Test Results, and SWE codebase changes, then writes manual test cases for the user under `# 3. Implementation Log & Test Results`.
7. **Phase 6: PM Completion Closeout**: Activated only after Staff QA manual test cases exist and the user explicitly confirms manual testing is good. SWE reports completion to PM, then PM moves the current feature into `PLANS.md` under `# 4. Complete Features`.

---

## Global Engineering Guardrails

Every agent must enforce these universal rules:

1. **User Approval Rule**: Every decision must go through the user. Agents must not make product, technical, branch, implementation, testing, naming, default-value, or scope decisions without explicit user approval.
2. **No Silent Assumptions Rule**: If a choice is unclear, missing, risky, or could reasonably be handled more than one way, pause and ask the user. Do not proceed by assuming the answer.
3. **Dedicated Feature Branch Rule**: Every feature requested for implementation must start in its own dedicated git branch before any planning or source edits are made. Do not mix unrelated feature work in the same branch.
4. **Branch Naming Rule**: Prefer branch names in the format `feature/<short-kebab-case-feature-name>`, but the branch name must still be approved by the user before creation or checkout. If the correct branch cannot be created or selected safely, pause and ask the user how to proceed.
5. **Feature Opt-In and Default-Off Rule**: Every feature requested for implementation must be opt-in. No new feature may be enabled by default, alter gameplay by default, or run automatically without the player/user explicitly enabling it.
6. **Settings Menu Entry Required**: Every implemented feature must have an entry in the settings menu that allows the player/user to toggle the feature ON or OFF. The setting must default to OFF for new games, existing saves, and any migration/default initialization path unless the user explicitly requests otherwise.
7. **Runtime Guard Required**: Feature code must be guarded by the associated setting. When the setting is OFF, the code path must preserve vanilla behavior as closely as possible and avoid side effects.
8. **PM Settings Requirement**: The PM must explicitly state in the PRD that the feature will have a settings menu entry, will default OFF, and will be opt-in.
9. **Lead Investigation Requirement**: Before EM architecture begins, the Lead Investigator Agent must document current behavior, similar old/current implementations, anticipated problems, regression risks, and open questions in the approved investigation Markdown file, then receive explicit user approval.
10. **EM Settings Requirement**: The EM must allocate or identify the required setting flag/storage, menu hook, default initialization path, save/load behavior, and runtime guard locations before implementation begins.
11. **SWE Settings Requirement**: The SWE must implement the settings entry, default OFF value, persistence behavior, and runtime checks as part of the feature implementation.
12. **Verification Requirement**: Testing must include both automated checks and Staff QA-authored manual test cases for OFF and ON behavior. OFF-state testing must confirm the feature does not affect vanilla gameplay.
13. **Staff QA Manual Test Case Requirement**: After SWE implementation and automated QA checks, the Staff QA Engineer Agent must write user-executable manual test cases before manual acceptance begins.
14. **Manual Acceptance Closeout Requirement**: A feature is not complete until the user explicitly confirms manual testing is good using the Staff QA manual test cases. After that confirmation, SWE must report completion to PM, and PM must move the feature into the `# 4. Complete Features` section of `PLANS.md`.
