Of course. Here is a comprehensive `testcase.instructions.md` tailored for the NINA project. This document provides a detailed framework for ensuring the software's quality, correctness, and performance through rigorous testing.

---

# NINA Project - Test Case and Quality Assurance Instructions

**Primary Directive:** Code without tests is considered incomplete and broken. All contributions to NINA, whether in Python or C++, **MUST** be accompanied by a comprehensive suite of tests that validate their correctness, robustness, and performance. This document is the single source of truth for the project's testing philosophy and practices.

## 1. Core Testing Philosophy: The Testing Pyramid

We adhere to the "Testing Pyramid" model to ensure a fast, reliable, and maintainable test suite. The strategy is to have many small, fast unit tests at the base, fewer and slightly slower integration tests in the middle, and a very small number of slow end-to-end tests at the peak.

```
      / \
     / ▲ \
    /  |  \   <-- End-to-End (E2E) Tests (Few, Slow, Cover full user scenarios)
   /---+---\
  /    |    \  <-- Integration Tests (More, Test interactions between components)
 /-----+-----\
/      |      \ <-- Unit Tests (Lots, Fast, Test individual classes/functions in isolation)
---------------
```

**Our Goals:**
1.  **Correctness:** The software does what the PRD says it should do.
2.  **Robustness:** The software gracefully handles errors, invalid inputs, and unexpected conditions.
3.  **Performance:** The software meets or exceeds the performance and efficiency targets defined in the PRD.

## 2. Testing Tools and Infrastructure

-   **Python Testing Framework:** `pytest`
-   **Python Test Utilities:** `pytest-cov` (for coverage), `pytest-mock` (for mocking), `pytest-benchmark` (for performance).
-   **C++ Testing Framework:** `GoogleTest` (`gtest`) and `GoogleMock` (`gmock`).
-   **Continuous Integration (CI):** All tests **MUST** run and pass in the CI pipeline (e.g., GitHub Actions) before a pull request can be merged.
-   **Code Coverage:** We mandate a minimum of **90% line and branch coverage** for all new code. Pull requests that decrease overall coverage will be rejected.

---

## 3. Unit Tests

**Goal:** To verify that a single, isolated piece of code (a function, method, or class) works as intended.
**Scope:** One module/class at a time. All external dependencies (network, filesystem, other classes, hardware) **MUST** be mocked or stubbed.
**Characteristics:** Fast (<50ms per test), isolated, and repeatable.

### Python Unit Test Guidelines:

*   **Structure:** All tests **MUST** follow the **Arrange-Act-Assert (AAA)** pattern.
*   **Location:** Tests for `nina/core/component.py` should be in `tests/core/test_component.py`.
*   **Mocking:** Use `pytest-mock`'s `mocker` fixture to replace dependencies.
*   **Test Both Paths:** For every feature, test the "happy path" (correct inputs and expected outcome) and multiple "sad paths" (invalid inputs, exceptions, edge cases).

**NINA-Specific Python Examples:**
*   **`ConfigManager`:** Test that it correctly parses a sample YAML string. Test that it raises a `ConfigurationError` for a malformed file.
*   **`ModelManager`:** Mock the `huggingface_hub` download function. Test that the `ModelManager` correctly constructs file paths. Test the logic for checking if a model is already cached.
*   **`CLI`:** Test argument parsing logic. For a command `nina chat --model foo`, verify that the CLI layer calls the `Orchestrator` with the correct arguments (`start_chat_session(model_id='foo')`). The Orchestrator itself should be mocked.

### C++ Unit Test Guidelines:

*   **Structure:** Use `TEST_F` (for fixtures) or `TEST` macros. Use `ASSERT_*` for fatal checks and `EXPECT_*` for non-fatal checks.
*   **Location:** Tests for `src/nina/core/tensor.cpp` should be in `tests/core/tensor_test.cpp`.
*   **Mocking:** Use `gmock` to create mock implementations of abstract interfaces (like the HAL backends).
*   **Focus on Logic:** Test algorithms, data structures, and state transitions.

**NINA-Specific C++ Examples:**
*   **`Tensor` class:** Test constructor logic, data accessors, and metadata management.
*   **`KVCache` class:** Test insertion, retrieval, and eviction logic under various conditions (e.g., cache full, sequence lengths changing).
*   **Sampling Algorithms:** For a `TopKSampler`, test it with known logit distributions and verify that it returns the correct token IDs.
*   **`ModelParser`:** Test GGUF parsing logic using a small, valid, hand-crafted binary file. Test that it fails gracefully on a corrupted file.

## 4. Integration Tests

**Goal:** To verify that multiple components interact correctly.
**Scope:** Two or three components working together. Mocks are used sparingly, only for external systems (e.g., live cloud APIs) or very slow components.
**Characteristics:** Slower than unit tests, may involve real filesystem I/O.

### Integration Test Guidelines:

*   Use small, real, self-contained test data (e.g., a tiny 1M-parameter model file checked into the repo for testing).
*   Focus on the "seams" or interfaces between layers.
*   Clean up any artifacts created during the test (e.g., using `pytest` fixtures with `tmp_path`).

**NINA-Specific Examples:**
*   **[Python Orchestration -> C++ Binding]:** Write a test that calls the Python `Orchestrator.load_model()` method. This test will use a real (but tiny) model file, call through the real `pybind11` wrapper, and verify that the C++ `InferenceEngine` is correctly instantiated. The actual hardware can be a mocked CPU backend.
*   **[C++ Engine -> HAL]:** Test that the `InferenceEngine` can receive a request, create a plan, and correctly dispatch operations (e.g., `MatMul`, `Attention`) to a **real CPU backend implementation** from the HAL. Assert that the numerical output is correct against a pre-calculated reference value (e.g., from NumPy).
*   **[Hybrid Intelligence]:** Write a test that configures a local model and a mocked cloud API. Trigger an inference request. Mock the local C++ engine to throw an `OutOfMemoryError`. Assert that the `Orchestrator` catches this and correctly forwards the request to the mocked cloud API.

## 5. End-to-End (E2E) / Scenario Tests

**Goal:** To simulate a real user workflow from start to finish.
**Scope:** The entire application stack, from the command line to the inference result.
**Characteristics:** Very slow, potentially brittle, and used sparingly for critical user journeys.

### E2E Test Guidelines:

*   **Critical Paths Only:** Focus on the most important user scenarios (e.g., downloading a model, having a 3-turn conversation, running a batch job).
*   **Subprocess Execution:** These tests should run the `nina` CLI as a separate process to accurately simulate the user experience.
*   **Assert on Output:** Check the `stdout`, `stderr`, and exit codes of the CLI application. For a chat test, assert that the final response contains expected keywords.

**NINA-Specific Examples:**
1.  **Golden Path Chat:**
    a. **Arrange:** Ensure no models are cached.
    b. **Act:** Run `nina model download <tiny-test-model>`. Assert the command succeeds and the file exists.
    c. **Act:** Run `nina chat --model <tiny-test-model>` with a piped-in script of prompts (`cat prompts.txt | nina chat ...`).
    d. **Assert:** Check that the final `stdout` of the chat session matches a "golden" output file.
2.  **OpenAI-Compatible Server:**
    a. **Act:** Run `nina serve --model <tiny-test-model> --port 8888` in a background process.
    b. **Act:** Use an HTTP client (like `requests`) to send a request to `localhost:8888/v1/chat/completions`.
    c. **Assert:** Verify the HTTP response code is 200 and the JSON body has the expected structure and content.

## 6. Specialized Testing Strategies

### Hardware-Specific Testing

*   **Labeling:** All tests that require specific hardware **MUST** be marked with a `pytest` marker (e.g., `@pytest.mark.cuda`, `@pytest.mark.metal`).
*   **CI Configuration:** The CI pipeline will have separate jobs that run on GPU-enabled runners. The `cuda` tests will only run on the NVIDIA runners.
*   **Graceful Skip:** Tests **MUST** be written to automatically skip if the required hardware is not detected. This allows developers without GPUs to run the rest of the test suite.
*   **Reference Implementation:** The primary numerical correctness tests for HAL kernels **MUST** run against the reference CPU backend. A smaller set of tests will run on actual GPU hardware to confirm the kernels execute without crashing and produce bit-identical results to the CPU version.

### Performance & Benchmark Testing

*   **Regression Suite:** A dedicated benchmark suite (`benchmarks/`) will be created. These are not correctness tests.
*   **Tooling:** Use `pytest-benchmark` or a similar framework.
*   **Dedicated Environment:** Benchmarks **MUST** be run on a dedicated, physically consistent machine, not on variable CI runners.
*   **PR Guard:** The CI pipeline will run a fast "smoke test" benchmark on pull requests. If performance degrades by more than a set threshold (e.g., 5%), the PR is flagged for review. The full benchmark suite is run nightly on the dedicated machine, and results are tracked on a dashboard.

### Nonstop Autonomy Testing (Chaos Engineering)

*   **Goal:** To validate the "nonstop" and "self-healing" claims from the PRD.
*   **Strategy:** Intentionally inject failures and verify the system's resilience.
*   **Examples:**
    *   Write an integration test where a C++ inference call is mocked to segfault. Verify the Python `Orchestrator` catches the subprocess crash and restarts the engine or fails over.
    *   During a batch processing job, use `mocker` to simulate a disk I/O error when writing an output file. Verify the job retries or logs the failure for that specific prompt without crashing the entire batch.

## 7. A Good Test Case Checklist

Before submitting, ensure every test case meets these criteria:

-   [ ] **Clear Name:** The test function name describes what it's testing (e.g., `test_load_model_fails_on_corrupted_file`).
-   [ ] **AAA Structure:** The `Arrange`, `Act`, and `Assert` sections are clearly separated.
-   [ ] **Single Responsibility:** The test verifies one logical condition.
-   [ ] **Isolated & Repeatable:** The test does not depend on the state of other tests and gives the same result every time.
-   [ ] **Includes "Sad Paths":** You have also tested for expected errors, not just the happy path.
-   [ ] **Documented:** A brief comment explains the purpose of the test if it's not immediately obvious from the name.

---