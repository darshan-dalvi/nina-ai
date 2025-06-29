# NINA Project - CLI Design and UX Instructions

**Primary Directive:** The NINA Command Line Interface (CLI) is the primary entry point for our users. It **MUST** be more than just a functional tool; it must be an intuitive, helpful, and empowering experience. Our goal is to set a new standard for AI developer tools. Every design decision **MUST** prioritize clarity, productivity, and user delight.

**Guiding Principles:**
1.  **Discoverable:** Users should be able to figure out how to use the CLI without constantly referring to documentation.
2.  **Interactive & Guided:** When a user is unsure, the CLI should guide them with prompts, suggestions, and auto-completion.
3.  **Informative & Concise:** Output should be well-structured, relevant, and easy to parse, using color and formatting to draw attention to key information.
4.  **Powerful for Experts:** While friendly for beginners, the CLI **MUST** provide powerful, scriptable options for advanced users and automation.
5.  **Consistent:** The command structure, flags, and output format **MUST** be consistent across all commands.

## 1. CLI Framework and Technology

-   **Framework:** **`Typer`**. It is built on `Click` but provides modern features like automatic help generation from type hints and a simpler API. It encourages best practices.
-   **Rich Output:** **`rich`**. This library is non-negotiable for all CLI output. It provides beautiful formatting for tables, progress bars, spinners, markdown, syntax highlighting, and more.
-   **Interactivity:** **`rich.prompt`** or a similar library for guided input.
-   **Auto-completion:** The CLI **MUST** ship with shell completion scripts (`nina --install-completion`). `Typer` can generate these automatically.

## 2. Command Structure and Naming

We will use a `noun verb` structure. It's intuitive and easy to remember.

**Correct Structure:** `nina <NOUN> <VERB> [ARGS] [--FLAGS]`
-   `nina model download ...`
-   `nina config set ...`
-   `nina profile create ...`

**Avoid:** `nina <VERB>-<NOUN> ...` (e.g., `nina download-model`).

### Core Command Groups (`NOUNS`):

-   `nina model`: For managing models (download, list, optimize, benchmark).
-   `nina config`: For managing user configuration and profiles.
-   `nina profile`: A specific subcommand group for managing hardware/use-case profiles.
-   `nina serve`: For running the OpenAI-compatible server.
-   `nina monitor`: For real-time performance monitoring.

### Top-Level Commands (`VERBS` at root level):

These are the primary actions a user will take.

-   `nina chat`: The main interactive chat mode.
-   `nina complete`: For one-shot, scriptable completions.
-   `nina batch`: For batch processing a file of prompts.

## 3. The Interactive Experience (`nina chat`)

This is the flagship experience. It must be polished and feel intelligent.

### Key Features:

1.  **Welcome Message:** On launch, display a clean, helpful welcome message.
    ```
    NINA 🤖 - Nonstop Autonomy Engine v0.1.0
    Model: llama-3.1-8b-instruct (local, 4-bit, cuda:0)
    Type '/help' for a list of commands, '/exit' to quit.
    ---
    You:
    ```
2.  **Rich, Streaming Output:**
    -   User prompts (`You:`) should be a distinct color (e.g., bold blue).
    -   The AI's response (`NINA:`) should be another color (e.g., bold green).
    -   Responses **MUST** stream token-by-token. Use a `rich` spinner or a subtle typing indicator while waiting for the first token.
    -   **Syntax Highlighting:** If the model generates code, it **MUST** be automatically detected and syntax-highlighted using `rich.syntax`.
    -   **Markdown Rendering:** If the model generates Markdown (e.g., lists, tables, bold text), it **MUST** be rendered beautifully in the terminal using `rich.markdown`.

3.  **Slash Commands (`/`):** A powerful, discoverable way to control the session.
    -   `/help`: Display a table of available slash commands.
    -   `/model <model_id>`: Hot-swap to another loaded model. This should have auto-completion for available models.
    -   `/context`: Show current context size and token count.
    -   `/reset`: Clear the conversation context.
    -   `/save <filename>`: Save the current session to a file.
    -   `/load <filename>`: Load a previous session.
    -   `/benchmark`: Show a quick performance summary of the last response (tokens/sec, TTFT).
    -   `/exit` or `/quit`: Exit the chat session.

4.  **Intelligent Prompting:** Use `rich.prompt.Prompt` for a superior input experience, including history navigation (up/down arrows).

## 4. Visual Feedback and UI Elements

Never leave the user staring at a blank screen.

-   **Spinners:** For long-running operations like model downloads or initial loading, use a `rich` spinner with descriptive text.
    ```
    [bold cyan]⠹[/] Downloading llama-3.1-8b-instruct... (2.1/4.7 GB)
    ```
-   **Progress Bars:** For quantifiable progress, use `rich.progress`. Downloads **MUST** show file size and download speed. Batch processing **MUST** show a progress bar of completed prompts.
-   **Tables:** For lists of items (`nina model list`), use `rich.table.Table` with clear headers and aligned columns. Use subtle colors to differentiate rows or highlight the active/default item.
-   **Panels:** Wrap important messages, warnings, or errors in a `rich.panel.Panel` to make them stand out.
    -   **Success:** Green border, checkmark emoji.
    -   **Warning:** Yellow border, warning emoji.
    -   **Error:** Red border, cross emoji.

**Example Error Message:**

```
╭─ [bold red]Error[/] ──────────────────────────────────────────╮
│ 💥 Model Not Found: 'llama-3.1-9b-instruct'      │
│                                                     │
│ NINA could not find this model locally.             │
│                                                     │
│ To see available models, run:                       │
│   [cyan]nina model list --cloud[/]                  │
│                                                     │
│ To download it, run:                                │
│   [cyan]nina model download llama-3.1-8b-instruct[/]  │
╰─────────────────────────────────────────────────────╯
```

## 5. Command Design and Help Text (`--help`)

The `--help` flag is a user's best friend. `Typer` and `rich` make this easy to perfect.

-   **Clear Descriptions:** Every command, argument, and option **MUST** have a concise, helpful description.
-   **Use `rich_help_panel`:** Group related options into panels (e.g., "Model Options", "Performance Options").
-   **Show Defaults:** The help text **MUST** clearly indicate the default value for each option.
-   **Use Examples:** The most important commands should include an `Examples:` section in their help text.

**Example `nina model optimize --help`:**

```
Usage: nina model optimize [OPTIONS] MODEL_ID

  Optimizes a model for the local hardware.

  This command applies quantization and other optimizations to create a
  smaller, faster version of the model in the NINA cache.

Arguments:
  MODEL_ID  The ID of the model to optimize (e.g., 'llama-3.1-8b') [required]

Options:
  --quantize-bits [4|8]  The target bit depth for quantization.
                                       [default: 4]
  --target-device TEXT   The hardware to optimize for (e.g., 'auto', 'cuda:0').
                                       [default: auto]
  --output-id TEXT     A custom ID for the new optimized model.
  --help                 Show this message and exit.

Examples:
  # Optimize Llama 3 to 4-bit for the best available device
  $ nina model optimize llama-3.1-8b

  # Create an 8-bit version specifically for the CPU
  $ nina model optimize llama-3.1-8b --quantize-bits 8 --target-device cpu
```

## 6. Configuration Experience (`nina config`)

Configuration should be interactive and foolproof.

-   **`nina config init`:** A guided, interactive setup wizard for first-time users.
    -   It should ask for the model cache location.
    -   It should run hardware detection and ask the user to confirm the best device.
    -   It should ask if they want to download a recommended default model.
-   **`nina config set <key> <value>`:** For scriptable configuration.
-   **`nina config get <key>`:** To view a specific setting.
-   **`nina config list`:** To view the current configuration in a clean table.

## 7. Scripting and Automation

While interactive use is key, power users need automation.

-   **Standard Exit Codes:**
    -   `0`: Success.
    -   `1`: General error (e.g., model not found, configuration error).
    -   `2`: Invalid command-line arguments.
-   **JSON Output:** Any command that produces listable output (e.g., `nina model list`) **MUST** have a `--json` flag to output raw, machine-readable JSON for piping into tools like `jq`.
-   **Quiet Mode:** A `-q` or `--quiet` flag **MUST** be available to suppress all spinners, progress bars, and other "UI" elements, printing only the final result or error. This is essential for CI/CD logs.
-   **No-Interaction Flag:** Any interactive command (like `nina config init`) **MUST** have a `--no-interaction` flag (or similar) that allows all options to be passed as flags, preventing it from hanging while waiting for user input.

---
## 8. CLI Implementation Checklist

For every new CLI command, ensure the following:

-   [ ] **Command exists in `Typer` app:** The basic command and options are defined.
-   [ ] **Help text is comprehensive:** All descriptions, defaults, and examples are present.
-   [ ] **Follows `noun verb` pattern:** The command fits into the established structure.
-   [ ] **Uses `rich` for all output:** No `print()` statements.
-   [ ] **Provides visual feedback:** Spinners/progress bars are used for any wait > 1 second.
-   [ ] **Handles success state:** A clear, positive confirmation message is shown (e.g., a green panel).
-   [ ] **Handles error states:** Errors are caught and displayed in a helpful, formatted panel with suggested next steps.
-   [ ] **Has scriptable options:** `--json` and `--quiet` flags are implemented where applicable.
-   [ ] **Has interactive fallbacks:** If a required argument is missing, the CLI prompts the user for it interactively.
-   [ ] **Has auto-completion support:** The command and its arguments are discoverable via tab-completion.