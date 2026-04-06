# GSoC 2026 — Apache Software Foundation / Apache Airflow
# Airflow Contribution & Verification Agent Skills (Breeze-Aware AI Skills)

---

## Contact Details

| Field | Details |
|---|---|
| **Full Name** | Shashwati Bhattacharya |
| **Email** | starsb1402@gmail.com |
| **GitHub** | [github.com/shashbha14](https://github.com/shashbha14) |
| **LinkedIn** | [linkedin.com/in/shashwati-bhattacharya](https://www.linkedin.com/in/shashwati-bhattacharya-197589269/) |
| **Location** | Naya Raipur, Chhattisgarh, India |
| **University** | International Institute of Information Technology, Naya Raipur |
| **Degree** | B.Tech in Electronics and Communication Engineering |
| **Graduation** | June 2026 |

---

## Project Synopsis

### About Apache Airflow

Apache Airflow is the world's leading open-source workflow orchestration platform, used by thousands of organizations globally to author, schedule, and monitor data pipelines. With over 35,000 GitHub stars, hundreds of active contributors, and a provider ecosystem spanning 80+ integrations, Airflow represents one of the most active and impactful projects in the Apache Software Foundation. The platform's success depends critically on its contributor community — yet today, contributing to Airflow is significantly harder than it needs to be.

The reason is **Breeze**.

### The Problem: AI Assistants Don't Understand Breeze

Breeze is Airflow's containerized development environment built on Docker. It encapsulates the entire Airflow development toolchain — running tests, executing static checks, building documentation, managing integrations — inside a container. Every contributor who wants to fix a bug or add a feature must use Breeze correctly.

The problem is that modern AI coding assistants — **Claude Code, GitHub Copilot, Gemini CLI** — treat Airflow like a generic Python project. They have no awareness of Breeze. When a contributor asks "run the unit tests," the assistant issues a plain `pytest` command on the host machine, which either fails outright or produces results that differ from CI. When asked to "fix static check failures," the assistant runs `pre-commit` outside the container, producing different results from what Airflow's CI actually runs.

This context mismatch is the single largest source of confusion for contributors using AI tools with Airflow. There is also no machine-readable encoding of Airflow's contribution workflows — no structured definition of what "fix a bug in a provider" or "update the documentation" looks like as a precise sequence of commands in the right context. AI assistants must guess, and they frequently guess wrong.

### The Solution: Breeze-Aware Agent Skills

This project proposes to build a comprehensive library of **Breeze-aware agent skills** — structured, machine-readable workflow definitions that teach AI coding assistants exactly how Airflow's development environment works.

The system will:

1. **Detect execution context** — reliably determine whether the agent is running on the host, inside a Breeze container, or in CI (GitHub Actions)
2. **Select correct commands** — route every action to the right command for the current context automatically
3. **Encode contribution workflows** — define the full contributor lifecycle (bug fix, docs update, provider enhancement, PR review) as reusable, machine-readable skills
4. **Auto-sync with Breeze** — automatically regenerate skills when the Breeze CLI changes, so the library never becomes stale
5. **Verify agent behavior** — provide a full test harness that validates every skill in all three execution contexts

### Expected Impact

A successful implementation will:
- Lower the barrier to contribution for every developer using an AI coding assistant with Airflow
- Reduce contributor onboarding friction significantly
- Create a self-maintaining infrastructure layer for AI-assisted Airflow development
- Serve as a reference implementation that other Apache projects can adopt

---

## Previous Work on the Project

*All pull requests listed below have been merged into apache:main. These contributions were undertaken independently before writing this proposal — to develop the deep codebase familiarity needed to build skills that accurately model Airflow's contributor workflows.*

### Merged Pull Request 1 — Bug Fix (Apache Airflow #62364)

**Title:** Fix HiveServer2Hook password handling for PLAIN auth
**PR:** [apache/airflow#62364](https://github.com/apache/airflow/pull/62364)
**Merged by:** @jscheffl — February 25, 2026
**Branch:** `shashbha14:fix-hive-thrift-password-62338`
**Lines changed:** +22 / -2

**Problem investigated:** When creating a HiveServer2 Thrift connection using PLAIN authentication (the default mechanism), Airflow was silently discarding the user-provided password and always sending the placeholder value `"x"` to the HiveServer2 service. This caused authentication failures even when users provided correct credentials. The root cause was in `HiveServer2Hook.get_conn()` — `"PLAIN"` was missing from the list of mechanisms that retrieve and pass the password to the underlying `pyhive` library.

**Fix implemented:** Added `"PLAIN"` to the authentication mechanism list. Added unit test `test_get_conn_with_password_plain` to verify the fix and prevent regression.

**What this contribution taught me:** Navigating the providers codebase under real reviewer scrutiny. This PR went through two rounds of reviewer-requested rebases before being approved. I learned how to properly rebase (`git rebase origin/main`), isolate commits, and satisfy the high review standards of the Airflow maintainer team. Reviewer @dabla wrote: *"Nicely done!"*

**Relevance to this project:** The fix involves `subprocess`/socket-level Python behavior — precisely the kind of low-level knowledge required to implement the Breeze context detection module and the Smart Command Router.

---

### Merged Pull Request 2 — Documentation Fix (Apache Airflow #63634)

**Title:** docs: clarify plugins folder sys.modules registration behavior
**PR:** [apache/airflow#63634](https://github.com/apache/airflow/pull/63634)
**Merged by:** @potiuk — March 2026
**Branch:** `shashbha14:docs/plugins-syspath-warning`
**Lines changed:** +26 / -0

**Problem investigated:** The Modules Management documentation incorrectly implied that `plugins/`, `dags/`, and `config/` are equivalent — that they are simply added to `sys.path`. In reality, the `plugins/` folder has fundamentally different semantics: `plugins_manager.load_plugins_from_plugin_directory()` actively imports all `.py` files at startup and registers them in `sys.modules` under the **bare filename** (`path.stem`), completely ignoring subdirectory structure. A file at `plugins/my_project/operators/hdfs.py` gets registered as `sys.modules["hdfs"]` — not `sys.modules["my_project.operators.hdfs"]` — silently shadowing the PyPI `hdfs` package and causing cryptic `ImportError`s in providers that depend on it.

**Fix implemented:** Added two warning sections to `modules_management.rst` with a concrete reproduction example and cross-reference to `plugins_manager.py`. This fixed issue #63548.

**What this contribution taught me:** How to trace Airflow internals from documentation to source code. I read `airflow/plugins_manager.py` line by line to understand the loading lifecycle. This is the kind of deep internals knowledge required to build skills that accurately describe what happens when an AI agent runs `breeze start-airflow` or `breeze shell`.

**Relevance to this project:** Understanding the Airflow runtime loading sequence is prerequisite knowledge for encoding accurate contribution workflow skills. A skill that describes "start Airflow" must reflect what actually happens at the Python level.

---

### Merged Pull Request 3 — Bug Fix (Apache Arrow #48896)

**Title:** GH-48853: Fix bytes to string comparison in download_rc_binaries.py
**PR:** [apache/arrow#48896](https://github.com/apache/arrow/pull/48896)
**Merged by:** @raulcd — January 20, 2026
**Branch:** `shashbha14:GH-48853-fix-bytes-string-comparison`
**Lines changed:** +1 / -1

**Problem investigated:** `subprocess.Popen().communicate()` returns `bytes` objects, but the Apache Arrow release script compared `stderr` (bytes) against the string `"OpenSSL"`, causing `TypeError: a bytes-like object is required, not 'str'` whenever a download failed. This broke the entire release download pipeline for the Arrow project.

**Fix implemented:** Changed the comparison to use the bytes literal `b"OpenSSL"`.

**What this contribution taught me:** Diagnosing bugs across large, unfamiliar OSS codebases quickly. This is a transferable skill — the same mental model applies when investigating Airflow CI failures.

---

### Active Work

**Branch:** `fix/login-wrong-credentials-error-64280`
**Title:** Fix wrong error message displayed when user logs in with incorrect credentials
**Status:** In progress (commits: `15de2c0e56`)

This fix touches the React UI layer (TypeScript/TSX). It demonstrates frontend familiarity alongside backend Python work — directly applicable to the integration guide work in Proposal 7 (configuring Claude Code and GitHub Copilot, which involve `.claude/` directory structures and workspace instruction files).

---

### Reflection on Pre-GSoC Contributions

Working through these contributions gave me direct, hands-on experience with the exact system this project aims to improve. I now know:
- What it means to run the wrong command outside Breeze and debug the consequences
- How to navigate CI failure output and identify root causes
- How Airflow's internal loading pipeline works at the Python level
- What the Airflow reviewer community expects from a PR

I am not designing the skills in this proposal theoretically. I am encoding workflows I have actually followed, mistakes I have actually made, and knowledge I have actually earned.

---

## Project in Detail

The project is structured around eight concrete deliverables, ordered by dependency.

---

### Proposal 1: Breeze Context Detection Module

**What it is:**
A Python module — `airflow.tools.breeze_context` — that reliably determines whether the current process is executing on the host machine, inside a Breeze Docker container, or in a CI environment such as GitHub Actions. This is the foundational layer on which every other skill depends.

**Why it is needed:**
An AI agent cannot choose the correct command without knowing where it is running. `pytest tests/unit/...` on the host and inside Breeze produce different results because Breeze configures the test database, environment variables, and PYTHONPATH. Without context detection, every skill is ambiguous.

**Implementation approach:**

The detection logic checks three signals in priority order:

```python
import enum, os, pathlib

class ExecutionContext(enum.Enum):
    BREEZE_CONTAINER = "breeze_container"
    HOST             = "host"
    CI               = "ci"

def detect_context() -> ExecutionContext:
    # 1. CI check first (GitHub Actions always sets GITHUB_ACTIONS=true)
    if os.environ.get("GITHUB_ACTIONS") == "true":
        return ExecutionContext.CI

    # 2. Breeze sets BREEZE_INITIALIZED on container startup
    if os.environ.get("BREEZE_INITIALIZED"):
        return ExecutionContext.BREEZE_CONTAINER

    # 3. All Docker containers have /.dockerenv as a filesystem marker
    if pathlib.Path("/.dockerenv").exists():
        return ExecutionContext.BREEZE_CONTAINER

    return ExecutionContext.HOST
```

**CLI exposure:**
```
$ breeze context detect
ExecutionContext.HOST
```

**Tools:** Python stdlib only — `os`, `pathlib`, `enum`. Zero external dependencies, which ensures this module can be imported anywhere in the codebase without dependency conflicts.

**Tests:** Unit tests using `unittest.mock` to patch `os.environ` and `pathlib.Path.exists` for all three contexts. 100% branch coverage required.

---

### Proposal 2: Skill Definition Schema and Initial Skill Library

**What it is:**
A YAML-based schema for defining Airflow contributor skills, validated with Pydantic v2, plus an initial library of **15+ skills** covering the complete contribution lifecycle.

**Why it is needed:**
There is currently no machine-readable encoding of "how to contribute to Airflow." AI assistants must infer contribution procedures from README files written for humans, which are frequently outdated and lack the context-specific command variants that Breeze requires. A structured schema enables programmatic validation, auto-generation, and cross-assistant portability.

**Skill Schema (YAML format):**

```yaml
name: run-unit-tests
description: Run unit tests for a specific Airflow module inside Breeze
context: [breeze_container, ci]
command:
  breeze_container: "python -m pytest {test_path} -v"
  ci: "python -m pytest {test_path} -v --no-header"
parameters:
  - name: test_path
    description: Path to the test file or directory
    required: true
    example: "tests/unit/providers/apache/hive/hooks/test_hive.py"
prerequisites:
  - breeze-start
on_success: "Tests passed. Check coverage output for uncovered lines."
on_failure: >
  Check assertion errors in the output. Use 'investigate-issue' skill
  if the root cause is unclear. Ensure the test database is initialized.
examples:
  - description: Run hive provider hook tests
    invocation: "run-unit-tests tests/unit/providers/apache/hive/"
  - description: Run a single test file
    invocation: "run-unit-tests tests/unit/core/test_dag.py"
```

**Initial Skill Library (15+ skills):**

| Skill | Context | Description |
|---|---|---|
| `breeze-start` | host | Start the Breeze development environment |
| `breeze-shell` | host | Enter an interactive Breeze shell |
| `run-unit-tests` | container / ci | Run pytest for a specific module |
| `run-static-checks` | container / ci | Execute pre-commit checks on changed files |
| `run-static-checks-all` | container / ci | Run all static checks |
| `build-docs` | container | Build Airflow RST documentation |
| `spellcheck-docs` | container | Run spell checker on documentation |
| `create-newsfragment` | host / container | Add changelog news fragment |
| `rebase-branch` | host | Rebase branch on latest main |
| `investigate-issue` | container | Reproduce a reported GitHub issue |
| `fix-provider-bug` | container | Full workflow for a provider bug fix |
| `add-unit-test` | container | Scaffold a new unit test file |
| `check-mypy` | container | Run mypy type checks |
| `check-pr-checks` | host | Identify which CI checks are failing |
| `submit-pr` | host | Full PR submission checklist |
| `update-provider-deps` | container | Update provider dependency versions |

**Format compatibility:**
- Claude Code: `.claude/skills/` directory (YAML with markdown front-matter)
- GitHub Copilot: referenced from `.github/copilot-instructions.md`
- Gemini CLI: JSON tool definition format

**Tools:** PyYAML, Pydantic v2, `jsonschema`.

---

### Proposal 3: Breeze CLI Auto-Sync Mechanism

**What it is:**
A GitHub Actions workflow combined with a Python script that automatically parses the Breeze CLI source code, detects when commands or parameters have changed, and opens a PR to update the skill library accordingly.

**Why it is needed:**
The #1 failure mode of any documentation or skill library is becoming stale. Breeze is actively developed — new commands are added, parameters change, deprecated flags are removed. A skill that diverges from the real Breeze CLI is worse than no skill at all because it confidently gives wrong instructions. Auto-sync makes this a self-maintaining system.

**Implementation approach:**

```python
# dev/scripts/sync_breeze_skills.py
import ast, pathlib, yaml

BREEZE_COMMANDS_DIR = pathlib.Path("dev/breeze/src/airflow_breeze/commands/")
SKILLS_DIR = pathlib.Path(".claude/skills/")

def extract_click_commands(source_file: pathlib.Path) -> list[dict]:
    """Use Python AST to extract Click command definitions from Breeze source."""
    tree = ast.parse(source_file.read_text())
    commands = []
    for node in ast.walk(tree):
        if isinstance(node, ast.FunctionDef):
            for decorator in node.decorator_list:
                if (isinstance(decorator, ast.Call)
                        and hasattr(decorator.func, 'attr')
                        and decorator.func.attr in ('command', 'group')):
                    commands.append({
                        'name': node.name.replace('_', '-'),
                        'docstring': ast.get_docstring(node) or "",
                    })
    return commands

def find_stale_and_missing(
    existing: set[str], current: set[str]
) -> tuple[set[str], set[str]]:
    return existing - current, current - existing
```

**GitHub Actions workflow (triggered on push to `dev/breeze/`):**
1. Run `sync_breeze_skills.py` to compare current Breeze commands against skill library
2. If stale or missing skills are detected: automatically open a PR with the diff, tagged `area:skills auto-sync`
3. Assign to the skills maintainer (the GSoC contributor, post-program)

**Tools:** Python `ast`, Click `get_help()` introspection, PyYAML, `PyGithub`, GitHub Actions.

---

### Proposal 4: Smart Command Router

**What it is:**
A thin CLI wrapper — `airflow-skill <skill-name> [parameters]` — that routes skill execution to the correct context, handling the host↔container boundary transparently so contributors never need to think about it.

**Why it is needed:**
Even with a correct skill library, a contributor on the host machine who runs a container-scoped skill should get a helpful result — not an error. The router should detect context and automatically wrap the command with `breeze shell --` when necessary, making the correct behavior the default.

**Routing logic:**

```
$ airflow-skill run-unit-tests tests/unit/providers/apache/hive/

  → detect_context() returns ExecutionContext.HOST
  → skill requires: breeze_container
  → wrap: breeze shell -- python -m pytest tests/unit/providers/apache/hive/ -v
  → stream output to terminal with Rich formatting
```

**Fallback handling:**
- `breeze` not installed on host → print installation instructions
- Docker not running → detect and print actionable error with fix command
- Already inside container → execute directly, no wrapping

**Tools:** Python `Click` (CLI framework), `subprocess.Popen` with `pty` for proper TTY passthrough, `rich` for formatted terminal output.

**Example output:**
```
$ airflow-skill run-unit-tests tests/unit/providers/apache/hive/
[Airflow Skills] Context detected: HOST
[Airflow Skills] Skill requires: BREEZE_CONTAINER
[Airflow Skills] Routing via: breeze shell --
─────────────────────────────────────────
platform linux -- Python 3.11.8
collected 23 items

tests/unit/providers/apache/hive/hooks/test_hive.py ..........  [100%]
23 passed in 4.21s
```

---

### Proposal 5: Test Harness for Agent Skills

**What it is:**
A pytest-based framework that verifies every skill in the library behaves correctly in all three execution contexts — without requiring Docker to be running for unit tests.

**Why it is needed:**
Skills that are not tested break silently. A skill that tells an AI agent to run the wrong command is worse than no skill. The test harness is what separates a reliable, production-quality skill library from an artifact that looks good on paper but breaks in practice.

**Directory structure:**

```
tests/
  skills/
    conftest.py                  ← context mock fixtures
    test_context_detection.py    ← tests for ExecutionContext module
    test_skill_schema.py         ← validates every YAML against schema
    test_breeze_start.py
    test_run_unit_tests.py
    test_static_checks.py
    test_build_docs.py
    ... (one file per skill)
```

**Core fixtures:**

```python
# tests/skills/conftest.py
import pytest
from unittest.mock import patch

@pytest.fixture
def mock_breeze_context():
    """Simulate execution inside a Breeze container."""
    with patch.dict('os.environ', {'BREEZE_INITIALIZED': '1'}):
        yield

@pytest.fixture
def mock_host_context():
    """Simulate execution on the host machine."""
    with patch.dict('os.environ', {}, clear=True):
        with patch('pathlib.Path.exists', return_value=False):
            yield

@pytest.fixture
def mock_ci_context():
    """Simulate GitHub Actions CI environment."""
    with patch.dict('os.environ', {'GITHUB_ACTIONS': 'true'}):
        yield
```

**CI integration:** A `skill-tests` job added to the Airflow GitHub Actions matrix that runs on every PR touching `dev/breeze/` or `.claude/skills/`.

**Coverage target:** ≥80% at midterm evaluation; 100% at project end.

**Tools:** pytest, pytest-mock, `unittest.mock`, Docker SDK for Python (integration tests only).

---

### Proposal 6: Contribution Workflow Recipes

**What it is:**
End-to-end workflow recipes that chain multiple skills into complete, named contribution scenarios. Where skills are functions, recipes are programs.

**Why it is needed:**
A contributor fixing a bug needs to know not just "how to run tests" but the complete sequence: reproduce → root cause → fix → test → lint → newsfragment → PR. Recipes make this knowledge explicit, sequential, and executable.

**Recipe 1: `new-bug-fix`**
```
Step 1:  investigate-issue {issue_number}    → Reproduce the reported bug
Step 2:  fix-provider-bug {file_path}        → Implement the fix
Step 3:  add-unit-test {test_path}           → Write a regression test
Step 4:  run-unit-tests {test_path}          → Verify tests pass
Step 5:  run-static-checks                  → Fix any lint issues
Step 6:  create-newsfragment {issue_number} → Add changelog entry
Step 7:  submit-pr                          → Complete PR checklist
```

**Recipe 2: `documentation-update`**
```
Step 1:  investigate-issue {issue_number}   → Understand what is incorrect
Step 2:  [Edit RST files directly]
Step 3:  build-docs                         → Verify docs build cleanly
Step 4:  spellcheck-docs                    → Fix spelling errors
Step 5:  submit-pr
```

**Recipe 3: `provider-enhancement`**
```
Step 1:  [Implement feature in provider file]
Step 2:  add-unit-test
Step 3:  run-unit-tests
Step 4:  run-static-checks
Step 5:  update-provider-deps               → If new dependencies added
Step 6:  create-newsfragment
Step 7:  submit-pr
```

**Recipe 4: `fix-failing-ci`**
```
Step 1:  check-pr-checks                    → Identify which check is failing
Step 2:  run-static-checks                  → Reproduce locally
Step 3:  [Apply fix]
Step 4:  run-static-checks                  → Verify fix works
Step 5:  rebase-branch                      → Ensure branch is up-to-date
Step 6:  [Push updated branch]
```

**Format:** Each recipe is a Markdown file with embedded skill references. Compatible with Claude Code custom slash commands (`/new-bug-fix`) and GitHub Copilot workspace instructions.

---

### Proposal 7: AI Assistant Integration Guides

**What it is:**
Ready-to-use configuration files and integration guides for the three major AI coding assistants so contributors can use the Airflow skill library with zero manual setup.

**Claude Code integration:**
- Skills placed in `.claude/skills/` (one YAML file per skill)
- New section in `CLAUDE.md`: *"Using Airflow Skills with Claude Code"* explaining Breeze, execution contexts, and available skills
- Recipes exposed as custom slash commands: `/new-bug-fix`, `/fix-failing-ci`, `/documentation-update`

**GitHub Copilot integration:**
- `.github/copilot-instructions.md` additions: Breeze context explanation + skill invocation patterns
- Workspace-level instruction file for VS Code users

**Gemini CLI integration (stretch):**
- JSON tool definition files for Gemini CLI's tool-use format
- Setup script that registers the Airflow skills as Gemini tools

**Documentation:**
- `CONTRIBUTING_WITH_AI.md` in the repo root — step-by-step setup guide for each assistant
- This file will be linked from the main `CONTRIBUTING.rst`

---

### Proposal 8: Contributor Onboarding Command (Stretch Goal)

**What it is:**
A new `breeze onboard` command that runs an AI-guided interactive checklist for new contributors, using the skill test harness to verify their environment before they write their first line of code.

**Onboarding checklist:**
```
Airflow Contributor Onboarding
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Docker is running
✅ Python 3.11 detected
✅ pre-commit installed
✅ Breeze starts successfully
✅ Sample unit test passes inside Breeze
⚠️  GitHub SSH key not found
    → Fix: ssh-keygen -t ed25519 -C "your@email.com"

4/5 checks passed. Fix the warning above and you're ready to contribute!
```

**Tools:** Click, Rich, skills test harness from Proposal 5.

---

## Implementation Plan and Timeline

### Minimal Deliverables (guaranteed)

- Breeze context detection module with full test coverage
- YAML skill schema with Pydantic v2 validation
- Initial library of 15+ skills
- Breeze CLI auto-sync GitHub Actions workflow
- Smart Command Router (`airflow-skill` CLI)
- Test harness with ≥80% skill coverage
- Claude Code + GitHub Copilot integration
- Four contribution workflow recipes
- `CONTRIBUTING_WITH_AI.md` documentation

### Optional Deliverables (if time permits)

- Gemini CLI integration guide
- `breeze onboard` command
- Expand skill library to 25+ skills
- I plan to continue maintaining this library after GSoC ends, with the auto-sync mechanism keeping it current as Breeze evolves

### Detailed Timeline

| Period | Dates | Deliverables |
|---|---|---|
| **Community Bonding** | May 8 – Jun 1 | Deep-dive Breeze codebase; study Click AST introspection; finalize skill schema design with mentor; set up blog; explore all files in `dev/breeze/src/airflow_breeze/commands/` |
| **Week 1–2** | Jun 2 – Jun 15 | Implement `ExecutionContext` detection module; write unit tests for all 3 contexts; open PR #1 for review |
| **Week 3–4** | Jun 16 – Jun 29 | Design and finalize YAML schema with Pydantic v2; implement schema validator; write first 5 skills; mentor review checkpoint |
| **Week 5–6** | Jun 30 – Jul 13 | Expand skill library to 15+; implement skill loader and YAML parser; write all 4 recipe definitions; open PR #2 |
| **Midterm Evaluation** | **Jul 14** | **Context detection ✓ Schema ✓ 15 skills ✓ 4 recipes ✓** |
| **Week 7–8** | Jul 14 – Jul 27 | Build auto-sync: AST Breeze CLI parser + GitHub Actions workflow; validate against current Breeze commands; open PR #3 |
| **Week 9** | Jul 28 – Aug 3 | Implement Smart Command Router; TTY passthrough with `pty`; Rich terminal output; open PR #4 |
| **Week 10** | Aug 4 – Aug 10 | Build test harness; context mock fixtures; CI `skill-tests` job; achieve ≥80% skill coverage; open PR #5 |
| **Week 11** | Aug 11 – Aug 17 | Write Claude Code + Copilot integration guides; `CONTRIBUTING_WITH_AI.md`; open PR #6 |
| **Week 12 (Buffer)** | Aug 18 – Aug 24 | Buffer for review iteration delays; stretch goals if ahead of schedule |
| **Final Week** | Aug 25 – Sep 1 | Final polish; GSoC report; project blog post; maintainer handoff documentation |

---

## Plan for Communication with Mentors

- **Two video syncs per week:** Monday (plan the week, align on priorities) and Friday (demo progress, surface blockers early)
- **Daily async updates:** Short comment on the tracking GitHub issue — what I completed, what is next, any blockers
- **All work in small, incremental PRs:** Mentors can review code progressively rather than waiting for large batches. I will keep each PR focused on a single deliverable.
- **Weekly blog post** on Dev.to / Medium: (1) what I accomplished this week, (2) technical challenge I found interesting, (3) plan for next week. This also serves as a public progress record for the community.
- **Response SLA:** I will respond to all reviewer comments within 24 hours on weekdays
- **Design decisions:** When a non-trivial design choice needs mentor input, I will raise it as a GitHub Discussion with my proposed approach and the tradeoffs — not just a question. This respects mentor time.

---

## Past Experience

### Visaire CLI — *Most Directly Relevant*
**Tech:** Node.js, JavaScript, NPM | **Period:** Jun 2025 – Sep 2025
**Link:** [github.com/shashbha14](https://github.com/shashbha14) (check pinned repos)

I built an agentic CLI tool for NLP-based LLM interaction with file I/O, shell execution, and dependency management. Published on NPM with API integration for Claude, GPT, and Gemini; implemented JSON-based configuration persistence for managing agent state across sessions.

**Why this is directly relevant to the GSoC project:** Visaire is architecturally similar to what the Breeze-Aware Skills project requires — defining AI agent capabilities as structured, executable artifacts with context-aware routing. Building Visaire taught me the core design insight that informs this entire proposal: **the hardest part of building AI agent tools is not the AI — it is the context problem.** An agent that does not know where it is running and what tools are available cannot act correctly. Visaire taught me this at a small scale. The GSoC project solves this problem at production scale for one of the world's most active open source projects.

---

### AI Engineer Intern — Codec Technologies
**Tech:** Python, React.js, Gemini API | **Period:** May 2025 – Jul 2025

Engineered a web-based AI platform integrating Amazon, Flipkart, and Myntra for automated product comparison (40% efficiency improvement per user feedback). Built a carbon footprint calculator with a custom-trained ML model and Gemini 1.5 API (95% accuracy on test data).

**Relevance:** Production-grade experience integrating the exact AI APIs this project targets (Claude, Gemini, Copilot). I understand integration patterns, tool-calling conventions, and the practical challenges of making AI assistants reliable in real-world environments.

---

### CVision: Cataract Detection App
**Tech:** Python, TensorFlow Lite, Flutter, Gemini API | **Period:** Jan 2025 – May 2025

Developed an AI-powered cataract detection app using TensorFlow Lite CNNs for real-time mobile inference. Integrated a 3-model ensemble with averaging for accuracy, deployed on Google Play Store, implemented a Gemini API chatbot for eye health guidance.

**Publication:** IEEE CCNCPS, Dubai, June 2025 — *CVision: CNN-based cataract detection using TensorFlow Lite with 95%+ accuracy.*

**Relevance:** End-to-end software delivery experience from design through production deployment and publication. GSoC projects that produce lasting impact require this kind of delivery discipline.

---

### Visualization Tool for Pathfinding Algorithms
**Tech:** Python, HTML, CSS, React, JavaScript | **Period:** Aug 2023 – Dec 2023

Built an interactive web visualization tool for DFS/BFS pathfinding algorithms, reducing learning complexity for CS students by 40% through an interactive grid-based UI.

**Relevance:** React.js frontend experience — directly applicable to the active UI PR (#64280) and to writing the Claude Code integration guide, which involves working with `.claude/` directory structures.

---

### Apache Contributions — Direct Skill Mapping

| Contribution | What I Learned | Skill it Informs |
|---|---|---|
| Airflow #62364 (HiveServer2Hook) | Python providers internals, subprocess, testing, rebasing under review | Proposal 4 (Smart Router uses subprocess); Proposal 6 (provider-fix recipe) |
| Airflow #63634 (plugins docs) | `plugins_manager.py` runtime loading, RST docs | Proposal 2 (skill docs accuracy); Proposal 3 (understanding what Breeze does at startup) |
| Arrow #48896 (bytes fix) | Cross-project OSS debugging, subprocess.Popen bytes handling | Proposal 1 (context detection uses subprocess signals) |

---

### Other Achievements and Recognition

- **IEEE RASSE, Singapore (November 2025):** Trust-region optimization of T-shaped monopole antennas using ANN, KNN, LASSO
- **GHCI'24 Scholar** — awarded by AnitaB.org for outstanding academic performance, leadership, and contributions to technology
- **Runner-Up — Hack-o-Harbour, Technovate IIIT-NR 2025:** 2nd place for Inventory Management System with chatbot-powered ticketing, automated listings, gamification, and sentiment analysis
- **350+ LeetCode problems solved**
- **GPA:** 8.0 / 10.0 (CGPA)

---

## Motivation

I have wanted to do GSoC since I was a first-year student. I would watch seniors from other colleges get selected and feel something I can only describe as a mix of admiration and quiet determination — not envy, but a firm belief that I wanted to be in that position someday. I always wanted to stand out, to do something beyond what everyone around me was doing. DSA and placements were table stakes — necessary, but not enough for me. GSoC represented something more: a chance to contribute to software that real people depend on, to work with engineers from across the world, and to prove my ability in the most merit-based arena there is — open source.

That belief was tested hard during placement season. Despite having skills I was genuinely confident in — production-grade projects, published research, an internship — I watched opportunities go to candidates who, in my honest assessment, were less prepared. I do not say this with bitterness. I say it because it clarified something important: placement outcomes often depend on factors outside your control. The quality of your work is not always visible in a forty-five minute interview. I refused to accept that as the final verdict on what I was capable of. I wanted a platform where the work speaks for itself, where contribution is visible and verifiable, where effort and skill are directly rewarded. GSoC — and open source broadly — is that platform.

That is why I did not wait to apply. I went into the Airflow codebase, found real issues, fixed them, and merged PRs. Not to pad a resume, but because I needed to know I could hold my own in a serious engineering community. When @jscheffl merged my HiveServer2Hook fix and @potiuk merged my documentation PR, it confirmed something that no placement result had: that my work meets the standard of one of the most rigorous open source communities in the world.

The technical motivation runs just as deep. When I first tried to contribute to Airflow, I spent hours debugging a test failure that had nothing to do with my code. The root cause: I was running `pytest` on my host machine, not inside Breeze. An AI assistant had confidently given me the wrong command. Nothing in the documentation warned me. That frustration gave me a precise picture of what this project should build.

My Visaire CLI had already taught me that the hardest problem in AI-assisted development is not model intelligence — it is the context problem. An agent that does not know where it is cannot act correctly. This GSoC project is the most important real-world instance of that problem I have encountered, and it is one I am uniquely positioned to solve: I have felt the pain personally, I have studied the Airflow internals that any solution must model accurately, and I have the skills to build it properly.

I want to build something that helps the next contributor avoid what I went through. I want to build something that keeps working after I am done. And I want to do it here, in this community, with these standards. That is my motivation.

---

## Working Time and Other Commitments

**Graduation:** June 2026. The Airflow GSoC coding period (June–September) falls after my graduation — I will have **no academic obligations** for the full coding period.

**Community bonding period (May):** Final semester exams in May. I will be available approximately **15–20 hours/week** for design discussions, codebase exploration, and mentor sync calls.

**Coding period (June–September): 45–50 hours/week.** Weekdays: minimum 8 hours/day. Weekends: as needed to stay on schedule.

**Other commitments:** No internship, no job offer accepted, no planned vacation during the coding period.

**Single project focus:** This is the **only GSoC project I am applying for.** I am choosing depth over breadth. This project deserves my full attention, and I intend to give it.

**Post-GSoC commitment:** I plan to maintain and expand the Airflow skill library after the program ends. The auto-sync system (Proposal 3) makes this sustainable — as Breeze evolves, skills update automatically. I will handle cases requiring human judgment: new skill categories, breaking Breeze changes, integration with new AI assistant tools as the ecosystem evolves.

---

*Thank you for considering this proposal.*

---

**Shashwati Bhattacharya**
starsb1402@gmail.com | [github.com/shashbha14](https://github.com/shashbha14)
