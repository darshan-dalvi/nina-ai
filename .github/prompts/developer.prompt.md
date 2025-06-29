# NINA Project - Master Development Prompt

**Objective:** To implement a new feature or fix a bug within the NINA project, ensuring strict adherence to all established project standards for architecture, code quality, testing, and user experience.

**Primary Directive:** You are a developer working on the NINA project. Before writing any code, you must internalize the project's core principles. Your work must be a seamless, high-quality addition to the existing codebase. You are responsible for the full lifecycle of your contribution: design, implementation, testing, and documentation.

Refer to the following authoritative documents for any questions:

1.  [`PRD.md`]: The "what" and "why." Your feature must align with the project vision.
2.  [`architecture.instructions.md`]: The "how." Your code must fit within the defined layers and boundaries.
3.  [`python-code-guidelines.instructions.md`]: For all Python code.
4.  [`cpp-code-guidelines.instructions.md`]: For all C++ code.
5.  [`cli.instructions.md`]: For any changes affecting the user-facing CLI.
6.  [`testcase.instructions.md`]: For all testing requirements.

---

## Phase 1: Pre-Implementation (Analysis & Design)

Before you write a single line of implementation code, complete the following analysis.

### 1.1. Understand the Requirements
-   **[ ] Review the PRD:** Clearly identify the user story or technical requirement you are addressing. What problem are you solving? Who is the user?
-   **[ ] Define Acceptance Criteria:** What specific, measurable outcomes will define this task as "done"?
    -   *Example: "The user can optimize a model to 4-bit using `nina model optimize`. The resulting file is created in the cache, and the CLI shows a success message."*

### 1.2. Architectural Placement
-   **[ ] Identify the Layers:** In which layer(s) of the architecture does this feature belong?
    -   *Is it a CLI presentation change? (Layer 1)*
    -   *Is it core business logic? (Layer 2: Orchestration)*
    -   *Is it a performance-critical computation? (Layers 4 & 5: C++ Engine/HAL)*
    -   *Is it a change to the Python-C++ boundary? (Layer 3)*
-   **[ ] Define the Interfaces:** What new functions, methods, or classes are needed? How will they interact with existing components?
    -   **Dependency Injection:** How will your new component get its dependencies? It **MUST NOT** create them itself.
    -   **Data Contracts:** What data structures (Pydantic models, C++ structs) will be passed between layers? Define them first.

### 1.3. Plan the Tests
-   **[ ] Outline Unit Tests:** What are the individual units of logic? How will you test them in isolation? What mocks are needed?
-   **[ ] Outline Integration Tests:** Which components need to be tested together? How will you test the "seams" between them?
-   **[ ] Outline E2E Scenario (if applicable):** What critical user journey does this feature enable? How can it be tested from the command line?

---

## Phase 2: Implementation (Code & Documentation)

Now, begin writing the code, adhering strictly to the project's guidelines.

### 2.1. Python Implementation Checklist (`nina/`)
-   **[ ] Adhere to Guidelines:** All code is formatted with `black`, passes `ruff`, and follows all rules in `python-code-guidelines.instructions.md`.
-   **[ ] Type Hinting:** All functions and methods have 100% type hint coverage.
-   **[ ] Docstrings:** All public modules, classes, and functions have Google-style docstrings.
-   **[ ] Asynchronous Code:** All I/O is handled with `async`/`await`. No blocking calls.
-   **[ ] Configuration:** Access configuration via dependency injection, not global state.
-   **[ ] Error Handling:** Use custom, specific exceptions. No generic `except Exception:`.
-   **[ ] Logging:** Use the `logging` module. No `print()` statements in library code.

### 2.2. C++ Implementation Checklist (`src/`)
-   **[ ] Adhere to Guidelines:** All code is formatted with `clang-format` and follows all rules in `cpp-code-guidelines.instructions.md`.
-   **[ ] C++17 Standard:** Use modern C++17 features correctly.
-   **[ ] RAII and Smart Pointers:** Zero raw owning pointers. Use `std::unique_ptr` by default.
-   **[ ] Const Correctness:** `const` is used everywhere it is applicable.
-   **[ ] Interface-Driven:** Code against HAL interfaces, not concrete hardware implementations.
-   **[ ] Documentation:** All public APIs are documented with Doxygen-style comments.
-   **[ ] `pybind11` Layer:** The binding layer is thin, releases the GIL for long tasks, and translates exceptions correctly.

### 2.3. CLI Implementation Checklist (If Affecting CLI)
-   **[ ] Adhere to Guidelines:** The command follows the structure and UX patterns in `cli.instructions.md`.
-   **[ ] `rich` Output:** All user-facing output uses the `rich` library (panels, tables, spinners, etc.).
-   **[ ] Helpful Feedback:** The command provides clear success, warning, and error messages with suggested next steps.
-   **[ ] Scriptable:** The command includes `--json` and `--quiet` flags where appropriate.
-   **[ ] Comprehensive Help:** The `--help` message is detailed and includes examples.

---

## Phase 3: Post-Implementation (Validation & Review)

Your code is not done until it is proven to work and is reviewed.

### 3.1. Testing Checklist (`tests/`)
-   **[ ] Run All Tests:** Execute the entire test suite locally to ensure your changes have not caused regressions.
-   **[ ] Write New Tests:** Implement the unit, integration, and E2E tests you planned in Phase 1.
-   **[ ] Achieve Coverage Target:** Run `pytest-cov` and ensure your contribution meets the **>90% coverage** requirement.
-   **[ ] Test "Sad Paths":** Confirm that your code handles invalid inputs, edge cases, and simulated failures gracefully.
-   **[ ] Test Hardware-Specific Code (if applicable):** Mark hardware-dependent tests appropriately (`@pytest.mark.cuda`) and test on the relevant hardware if available.

### 3.2. Final Review Checklist

Before opening a pull request, perform a self-review.

-   **[ ] Code is Clean:** You have removed all commented-out code, `print()` statements, and temporary debugging artifacts.
-   **[ ] Commits are Atomic:** Your git history is clean, with small, logical commits and clear commit messages.
-   **[ ] Documentation is Updated:** If you changed an API or added a CLI command, you have updated the relevant documentation (`README.md`, `cli.instructions.md`, etc.).
-   **[ ] PR Description is Thorough:** Your pull request description clearly explains the "what" and "why" of your changes, links to the relevant issue, and provides clear steps for the reviewer to test your feature.

**By following this master prompt, you ensure that every contribution maintains the high standard of quality, consistency, and excellence that defines the NINA project.**