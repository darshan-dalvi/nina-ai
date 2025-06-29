# Nina AI Project - C++ Code Guidelines for AI Coding Assistants

You are an AI coding assistant contributing to the high-performance C++ backend of the **Nina AI** project, an intelligent personal assistant system. The C++ code you generate forms the performance-critical inference engine and core computational components. It **MUST** be exceptionally fast, memory-efficient, robust, and portable across multiple platforms (Linux, macOS, Windows) and architectures (x86, ARM).

**Primary Directive:** Adhere to these guidelines with extreme rigor. This code must be of the highest professional quality. Prioritize performance, safety, and clarity in that order.

## 1. Core Principles

1.  **Performance is Paramount:** The C++ layer exists for one reason: speed. Every line of code should be written with performance in mind for AI inference, text processing, and real-time conversation handling.
2.  **Resource Safety via RAII:** We do not manually manage memory or resources. Use smart pointers and other RAII types exclusively. No raw `new`, `delete`, `malloc`, or `free` for owning pointers.
3.  **AI-Optimized Design:** Design with AI workloads in mind - tensor operations, batch processing, memory-efficient model loading, and optimal hardware utilization.
4.  **Cross-Platform by Design:** All code must be written to be portable. Avoid platform-specific APIs outside of the designated Hardware Abstraction Layer (HAL).
5.  **Real-Time Responsiveness:** Optimize for low-latency AI interactions suitable for conversational experiences.

---

## 2. Language Standard & Compiler

-   **Standard:** **C++17**. This provides a strong balance of modern features (`std::optional`, `std::string_view`, structured bindings) and wide compiler support.
-   **Compilers:** Code must compile cleanly (zero warnings) with:
    -   GCC 9+
    -   Clang 10+
    -   MSVC v142+ (Visual Studio 2019+)
-   **Compiler Flags:** Always compile with high warning levels (`-Wall -Wextra -Wpedantic` for GCC/Clang, `/W4` for MSVC) and treat warnings as errors (`-Werror`).

---

## 3. Code Formatting (Non-Negotiable)

We enforce a uniform code style using `clang-format`. All C++ code **MUST** be formatted according to the project's `.clang-format` file.

**Instruction:** Before finalizing any C++ code, ensure it has been formatted with `clang-format` using the configuration below.

**.clang-format Configuration:**

```yaml
# Based on Google's C++ Style Guide
Language: Cpp
BasedOnStyle: Google
# --- Customizations for Nina AI ---
# Indentation
IndentWidth: 4
TabWidth: 4
UseTab: Never
# Pointer and Reference Alignment
PointerAlignment: Left
# Column Limit
ColumnLimit: 100
# Braces
BreakBeforeBraces: Allman
AllowShortFunctionsOnASingleLine: None
AllowShortIfStatementsOnASingleLine: false
# Include Blocks
SortIncludes: true
IncludeBlocks: Regroup
IncludeCategories:
  - Regex:           '^<.*\.h>'
    Priority:        1  # C System Headers
  - Regex:           '^<.*>'
    Priority:        2  # C++ Standard Library
  - Regex:           '.*'
    Priority:        3  # Other libraries (e.g., pybind11, ONNX, PyTorch)
  - Regex:           '^nina/.*'
    Priority:        4  # Our project's headers
# Other
AllowAllParametersOfDeclarationOnNextLine: false
```

---

## 4. Naming and Organization

Follow the [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) conventions with Nina AI-specific clarity.

-   **Namespaces:** `lowercase_with_underscores` (e.g., `nina::core`, `nina::inference`, `nina::conversation`).
-   **Types (Classes, Structs, Enums, Type Aliases):** `PascalCase` (e.g., `InferenceEngine`, `ConversationProcessor`, `ModelConfig`).
-   **Functions & Methods:** `PascalCase` (e.g., `ProcessMessage`, `ExecuteInference`, `LoadModel`).
-   **Variables (including function parameters):** `lowercase_with_underscores` (e.g., `user_input`, `model_path`, `response_tokens`).
-   **Class Member Variables:** `lowercase_with_underscores_` (trailing underscore, e.g., `model_config_`, `conversation_state_`).
-   **Constants & Enum Values:** `kPascalCase` (e.g., `kDefaultMaxTokens`, `kConversationTimeout`).
-   **Macros:** `NINA_UPPERCASE_WITH_UNDERSCORES`. Avoid macros; prefer `constexpr` variables or inline functions.

---

## 5. Headers and Includes

-   **Include Guards:** Use `#pragma once` in all header files. It is standard, less error-prone, and faster than traditional include guards.
-   **Include Order:** Strictly follow the order defined in the `.clang-format` configuration to prevent hidden dependencies.
    1.  Related header (e.g., in `conversation_engine.cpp`, the first include is `conversation_engine.h`)
    2.  C system headers (`<cstddef>`)
    3.  C++ standard library headers (`<vector>`, `<string>`, `<memory>`)
    4.  Other library headers (`<cuda_runtime.h>`, `<pybind11/pybind11.h>`, AI framework headers)
    5.  Our project's headers (`"nina/core/tensor.h"`, `"nina/inference/model.h"`)
-   **Forward Declarations:** Prefer forward declarations over full `#include` directives in header files to reduce compile times and coupling.

---

## 6. Resource Management and RAII

-   **Ownership:** Use `std::unique_ptr` for exclusive ownership. This is the default choice for AI models, tensors, and conversation state. Transfer ownership with `std::move`.
-   **Shared Ownership:** Use `std::shared_ptr` only when shared ownership is genuinely required (e.g., a model shared by multiple conversation threads). Be aware of the performance overhead.
-   **No Owning Raw Pointers:** Raw pointers (`*`) and references (`&`) may be used as non-owning observers of an object whose lifetime is guaranteed to be longer than the pointer/reference.
-   **Custom Deleters:** Use custom deleters with smart pointers for AI framework resources (e.g., CUDA memory, model weights).

**Example:**

```cpp
// GOOD - AI model resource management
#include <memory>
#include <vector>

class AIModel
{
public:
    // Factory function to enforce smart pointer usage
    static std::unique_ptr<AIModel> LoadFromFile(const std::string& model_path);
    
    // Process conversation input efficiently
    std::vector<float> ProcessInput(const std::vector<int>& tokens) const;
    
private:
    // Custom deleter for AI framework resources
    struct ModelDeleter {
        void operator()(void* ptr) const { /* AI framework cleanup */ }
    };
    
    AIModel(void* model_data, size_t model_size) 
        : model_data_(model_data, ModelDeleter{}), model_size_(model_size) {}

    std::unique_ptr<void, ModelDeleter> model_data_;
    size_t model_size_;
};

// BAD
class AIModel
{
private:
    void* model_data_; // Raw owning pointer, leaks if not deleted
};
```

---

## 7. Class Design

-   **Rule of Zero/Five:** Design classes so the default compiler-generated special member functions are correct (Rule of Zero). If you must write one, review all five.
-   **Interfaces:** Use abstract base classes with pure virtual functions for AI model interfaces, conversation processors, and hardware abstraction.
-   **`struct` vs. `class`:** Use `struct` for passive data objects (message data, model configuration). Use `class` for objects with behavior (inference engines, conversation handlers).
-   **Const Correctness:** Use `const` aggressively. AI inference operations that don't modify model state **MUST** be marked `const`.

---

## 8. Error Handling

-   **Exceptions for Critical Errors:** Use exceptions for errors that indicate serious problems (model loading failure, out of memory, hardware initialization failure).
-   **`std::optional` for Recoverable Operations:** For functions that may or may not return a value (e.g., conversation context lookup, optional model features), return a `std::optional<T>`.
-   **Error Codes for Performance-Critical Paths:** In high-frequency inference loops where exception overhead is unacceptable, use error codes. This should be rare and well-justified.

---

## 9. Performance Best Practices for AI Workloads

-   **Move Semantics:** Use `std::move` to transfer ownership of large AI resources (models, tensors) to avoid expensive copies.
-   **Value Passing:** Pass large objects (tensors, model configs) by reference-to-const (`const T&`). Pass small objects by value.
-   **`std::string_view`:** Prefer `std::string_view` over `const std::string&` for text processing functions to avoid allocations.
-   **`constexpr` and `constinit`:** Use `constexpr` for compile-time AI configuration constants.
-   **Memory Layout:** Optimize for cache locality in tensor operations. Prefer contiguous memory layouts for SIMD processing.
-   **Batch Processing:** Design APIs to handle batched operations for improved throughput.
-   **Final and Override:** Mark AI model interfaces appropriately to aid compiler optimization.

---

## 10. Hardware Abstraction Layer (HAL) for AI

This is crucial for supporting multiple AI acceleration platforms.

-   **Interface-Based:** Define common C++ interfaces for AI operations (e.g., `MatrixMultiply`, `Attention`, `Softmax`).
-   **Backend Implementations:** Provide separate implementations for each acceleration platform:
    -   CUDA (NVIDIA GPUs)
    -   ROCm (AMD GPUs)
    -   Metal (Apple Silicon)
    -   CPU/SIMD (fallback)
    -   OpenCL (cross-platform)
-   **Factory Pattern:** Use runtime hardware detection to select the optimal backend.
-   **Conditional Compilation:** Use `#ifdef` guards for backend-specific code (e.g., `#ifdef NINA_ENABLE_CUDA`).

**Example:**

```cpp
namespace nina::inference
{

class InferenceBackend
{
public:
    virtual ~InferenceBackend() = default;
    virtual std::vector<float> ExecuteInference(
        const std::vector<int>& input_tokens,
        size_t max_output_tokens
    ) const = 0;
};

class CudaInferenceBackend : public InferenceBackend
{
public:
    std::vector<float> ExecuteInference(
        const std::vector<int>& input_tokens,
        size_t max_output_tokens
    ) const override;
};

// Factory function
std::unique_ptr<InferenceBackend> CreateOptimalBackend();

} // namespace nina::inference
```

---

## 11. Python/C++ Interface (`pybind11`) for AI Integration

-   **Keep it Thin:** The `pybind11` layer is for binding, not AI logic. It should only translate between Python and C++ AI operations.
-   **Release the GIL:** For all AI inference calls, release the Global Interpreter Lock to allow Python concurrency.
    ```cpp
    m.def("process_conversation", &ProcessConversation, 
          py::call_guard<py::gil_scoped_release>());
    ```
-   **Efficient Data Transfer:** Use `py::array_t` for efficient tensor/token data transfer between Python and C++.
-   **AI-Specific Exception Mapping:** Map C++ AI exceptions to appropriate Python exceptions for graceful error handling.

---

## 12. Documentation (Doxygen) for AI Components

All public AI APIs **MUST** be documented with clear descriptions of their AI-specific behavior.

```cpp
/**
 * @brief Processes a conversation message and generates an AI response.
 *
 * This function handles tokenization, model inference, and response generation
 * in a thread-safe manner optimized for real-time conversation.
 *
 * @param user_input The raw text input from the user.
 * @param conversation_context Optional context from previous messages.
 * @param max_response_tokens Maximum number of tokens to generate.
 * @return A unique pointer to the response containing generated text and metadata.
 * @throws nina::inference::ModelNotLoadedException if no model is loaded.
 * @throws nina::inference::InferenceException if generation fails.
 */
std::unique_ptr<ConversationResponse> ProcessMessage(
    const std::string& user_input,
    const ConversationContext* conversation_context = nullptr,
    size_t max_response_tokens = 512
);
```

---

## 13. Testing (`GoogleTest`) for AI Components

-   **Framework:** Use `GoogleTest` for all unit and integration tests.
-   **AI-Specific Testing:**
    -   Unit tests for individual AI components (tokenizers, model loaders)
    -   Integration tests for complete inference pipelines
    -   Performance benchmarks for response time requirements
    -   Conversation flow tests with mock interactions
-   **Test Data:** Use small, deterministic test models for consistent testing.
-   **Hardware Mocking:** Mock hardware backends for testing without requiring specific GPUs.

---

## Final Check: Nina AI Example

Here is a template for AI-specific C++ code following all guidelines.

**`nina/conversation/processor.h`**

```cpp
#pragma once

#include <memory>
#include <string>
#include <string_view>
#include <vector>

#include "nina/conversation/context.h"
#include "nina/conversation/response.h"
#include "nina/inference/model.h"

namespace nina
{
namespace conversation
{

/**
 * @brief High-performance conversation processor for Nina AI.
 * 
 * Handles the complete conversation pipeline from user input to AI response,
 * optimized for real-time interactive experiences.
 */
class ConversationProcessor
{
public:
    explicit ConversationProcessor(std::unique_ptr<inference::Model> model);
    ~ConversationProcessor();

    /**
     * @brief Processes user input and generates an AI response.
     *
     * @param user_input The user's message text.
     * @param context Optional conversation context for continuity.
     * @return A unique pointer to the response containing generated text.
     */
    std::unique_ptr<ConversationResponse> ProcessInput(
        std::string_view user_input,
        const ConversationContext* context = nullptr
    );

    /**
     * @brief Checks if the processor is ready for inference.
     */
    bool IsReady() const;

private:
    class ConversationProcessorImpl;
    std::unique_ptr<ConversationProcessorImpl> impl_;
};

} // namespace conversation
} // namespace nina
```

**`nina/conversation/processor.cpp`**

```cpp
#include "nina/conversation/processor.h"

#include <algorithm>
#include <chrono>

#include "nina/tokenization/tokenizer.h"
#include "nina/inference/engine.h"

namespace nina
{
namespace conversation
{

class ConversationProcessor::ConversationProcessorImpl
{
public:
    explicit ConversationProcessorImpl(std::unique_ptr<inference::Model> model)
        : model_(std::move(model))
        , tokenizer_(tokenization::CreateTokenizer(model_->GetVocabPath()))
        , inference_engine_(inference::CreateEngine(model_.get()))
    {
    }

    std::unique_ptr<ConversationResponse> ProcessInput(
        std::string_view user_input,
        const ConversationContext* context
    );

    bool IsReady() const 
    { 
        return model_ && tokenizer_ && inference_engine_; 
    }

private:
    std::unique_ptr<inference::Model> model_;
    std::unique_ptr<tokenization::Tokenizer> tokenizer_;
    std::unique_ptr<inference::Engine> inference_engine_;
};

std::unique_ptr<ConversationResponse> 
ConversationProcessor::ConversationProcessorImpl::ProcessInput(
    std::string_view user_input,
    const ConversationContext* context
)
{
    // Tokenize user input
    auto input_tokens = tokenizer_->Encode(user_input);
    
    // Add conversation context if available
    if (context) {
        auto context_tokens = context->GetTokens();
        input_tokens.insert(input_tokens.begin(), 
                           context_tokens.begin(), context_tokens.end());
    }

    // Execute AI inference
    auto start_time = std::chrono::high_resolution_clock::now();
    auto output_tokens = inference_engine_->Generate(input_tokens, 512);
    auto end_time = std::chrono::high_resolution_clock::now();

    // Convert tokens back to text
    auto response_text = tokenizer_->Decode(output_tokens);

    // Create response with metadata
    auto response = std::make_unique<ConversationResponse>();
    response->SetText(std::move(response_text));
    response->SetGenerationTime(
        std::chrono::duration_cast<std::chrono::milliseconds>(
            end_time - start_time
        ).count()
    );

    return response;
}

// Public interface implementation
ConversationProcessor::ConversationProcessor(std::unique_ptr<inference::Model> model)
    : impl_(std::make_unique<ConversationProcessorImpl>(std::move(model)))
{
}

ConversationProcessor::~ConversationProcessor() = default;

std::unique_ptr<ConversationResponse> ConversationProcessor::ProcessInput(
    std::string_view user_input,
    const ConversationContext* context
)
{
    return impl_->ProcessInput(user_input, context);
}

bool ConversationProcessor::IsReady() const
{
    return impl_->IsReady();
}

} // namespace conversation
} // namespace nina
```