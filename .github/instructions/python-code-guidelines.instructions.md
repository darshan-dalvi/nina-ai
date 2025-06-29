# Nina AI Project - Python Code Guidelines for AI Coding Assistants

You are an AI coding assistant contributing to the **Nina AI** project, an intelligent personal assistant system. Your primary role is to generate, refactor, and review Python code that is clean, efficient, maintainable, and strictly adheres to the architectural principles for building robust AI assistant applications.

**Primary Directive:** Follow these guidelines meticulously. When in doubt, prioritize clarity, explicitness, and consistency with the existing codebase and the project's AI assistant vision.

## 1. Core Principles

1.  **Readability Counts:** Write code for humans first, machines second. Follow "The Zen of Python" (`import this`).
2.  **Explicit is Better than Implicit:** Avoid magic. Clearly declare dependencies, configurations, and data flows.
3.  **AI-First Architecture:** Respect the boundaries between the user interface, AI orchestration, conversation management, and backend integration layers.
4.  **Developer-First:** Code should be easy to understand, debug, and extend. This includes comprehensive documentation and clear error messages.
5.  **Conversation-Aware:** Design with conversational AI patterns in mind - context management, state persistence, and response generation.

---

## 2. Code Formatting & Linting (Non-Negotiable)

We enforce a consistent style using a modern toolchain. All Python code **MUST** be formatted with `black` and linted with `ruff`.

-   **Formatter:** `black` (the uncompromising code formatter).
-   **Linter:** `ruff` (for speed and comprehensive checks, replacing `isort`, `flake8`, etc.).

**Configuration (`pyproject.toml`):** Use the following configuration for all Python code.

```toml
[tool.black]
line-length = 88
target-version = ['py39'] # Or the project's minimum Python version

[tool.ruff]
line-length = 88
select = [
    "E",  # pycodestyle errors
    "W",  # pycodestyle warnings
    "F",  # pyflakes
    "I",  # isort
    "B",  # flake8-bugbear
    "C4", # flake8-comprehensions
    "UP", # pyupgrade
    "A",  # flake8-builtins
    "T20" # flake8-print
]
ignore = ["E501"] # Handled by `black`

[tool.ruff.isort]
known-first-party = ["nina"]
```

**Instruction:** Before finalizing any code, ensure it has been processed by `black` and passes all `ruff` checks.

---

## 3. Naming Conventions

Adhere strictly to PEP 8 naming conventions, with Nina AI-specific clarity.

-   **Modules/Packages:** `lowercase_with_underscores` (e.g., `conversation_manager.py`, `nina.core.agents`).
-   **Classes:** `PascalCase` (e.g., `NinaAssistant`, `ConversationHandler`, `ModelNotFoundException`).
-   **Functions & Methods:** `lowercase_with_underscores` (e.g., `process_user_input`, `_parse_intent`).
-   **Variables:** `lowercase_with_underscores` (e.g., `user_message`, `response_context`).
-   **Constants:** `UPPERCASE_WITH_UNDERSCORES` (e.g., `DEFAULT_MODEL_NAME`, `MAX_CONVERSATION_HISTORY`).
-   **Private Members:** Use a single leading underscore (`_`) for internal methods or attributes (e.g., `self._conversation_state`). Avoid double leading underscores (`__`) unless you specifically need name mangling.

---

## 4. Documentation and Docstrings (Google Style)

All public modules, classes, and functions **MUST** have a docstring. We use the **Google Python Style Guide** for docstrings, as it is highly readable and supported by tools like Sphinx.

**Key Requirements:**
-   Start with a concise, one-line summary of the object's purpose.
-   Follow with a more detailed description if necessary.
-   Use `Args:`, `Returns:`, and `Raises:` sections to document parameters, return values, and exceptions.
-   Type hints are mandatory in the function signature, not in the docstring.

**Example:**

```python
# GOOD
import logging
from typing import Dict, List, Optional
from datetime import datetime

from .exceptions import ConversationError, ModelError

logger = logging.getLogger(__name__)

async def process_user_message(
    message: str,
    conversation_id: str,
    context: Optional[Dict[str, any]] = None,
) -> Dict[str, any]:
    """Processes a user message and generates an AI response.

    This function handles the complete conversation flow including intent
    recognition, context management, and response generation using the
    configured AI model.

    Args:
        message: The user's input message text.
        conversation_id: Unique identifier for the conversation session.
        context: Optional conversation context from previous interactions.

    Returns:
        A dictionary containing the AI response, updated context, and metadata.
        Keys include 'response', 'context', 'intent', and 'timestamp'.

    Raises:
        ConversationError: If the conversation state is invalid or corrupted.
        ModelError: If the AI model fails to generate a response.
    """
    logger.info(
        "Processing message for conversation %s: %s...",
        conversation_id,
        message[:50],
    )
    # ... processing logic here ...
    if processing_failed:
        raise ConversationError("Failed to process user message.")
    
    response_data = {
        "response": ai_response,
        "context": updated_context,
        "intent": detected_intent,
        "timestamp": datetime.utcnow().isoformat(),
    }
    return response_data
```

---

## 5. Type Hinting (PEP 484+)

Type hinting is **MANDATORY** for all new code. It is critical for developer productivity, static analysis, and bug prevention.

-   **Function Signatures:** All function and method arguments and return values must be type-hinted.
-   **Variables:** Use type hints for complex variables where the type is not immediately obvious.
-   **Modern Types:** Use types from the `typing` module (`List`, `Dict`, `Optional`, `Tuple`, `Callable`, `Any`). For Python 3.9+, prefer built-in generics (`list`, `dict`).
-   **Clarity:** Use `TypeAlias` (from `typing`) for complex or repetitive type signatures.
-   **AI-Specific Types:** Define clear type aliases for AI-related data structures.

**Example:**

```python
# GOOD
from typing import Dict, List, Optional, TypeAlias, Union
from datetime import datetime

# Use TypeAlias for Nina AI-specific types
ConversationContext = Dict[str, Union[str, int, List[str]]]
MessageHistory = List[Dict[str, str]]
IntentConfidence = Dict[str, float]

class ConversationManager:
    """Manages conversation state and context for Nina AI."""

    def __init__(self, max_history: int = 50):
        self._max_history = max_history
        self._conversations: Dict[str, MessageHistory] = {}
        self._contexts: Dict[str, ConversationContext] = {}

    def get_conversation_context(
        self, conversation_id: str
    ) -> Optional[ConversationContext]:
        """Retrieves the context for an active conversation.

        Args:
            conversation_id: The unique identifier for the conversation.

        Returns:
            The conversation's context dictionary, or None if not found.
        """
        return self._contexts.get(conversation_id)
```

---

## 6. Asynchronous Code (`asyncio`)

Nina AI's orchestration layer relies heavily on `asyncio` for responsive AI interactions.

-   **Use `async def`:** All I/O-bound operations (API calls, file access, model inference) **MUST** be defined as `async` functions and called with `await`.
-   **Avoid Blocking Calls:** Never call blocking I/O functions (e.g., `requests.get()`, `time.sleep()`) directly in an `async` function. Use `asyncio`-compatible libraries (e.g., `aiohttp`, `aiofiles`) or run blocking code in a separate thread using `asyncio.to_thread()`.
-   **AI Model Inference:** AI model calls should be async to prevent blocking the conversation flow when processing multiple users.
-   **Real-time Features:** Use `asyncio` for real-time features like streaming responses or live conversation updates.

---

## 7. Error Handling & Logging

A reliable AI assistant requires robust error handling with graceful degradation.

-   **Custom Exceptions:** Define specific, custom exceptions that inherit from a base `NinaException`. This provides granular error handling.
    -   Examples: `ConversationError`, `ModelNotAvailableError`, `IntentRecognitionError`, `ContextError`.
-   **Be Specific:** Catch specific exceptions, not generic `Exception` or `BaseException`.
-   **Graceful Degradation:** When possible, provide fallback responses instead of complete failures.
-   **Logging:** Use the standard `logging` module. **NEVER** use `print()` for logging or debugging output.
    -   `logger.debug()`: For detailed diagnostic information (conversation flow, model decisions).
    -   `logger.info()`: For high-level information about user interactions and system state.
    -   `logger.warning()`: For unexpected but recoverable events (fallback model usage, context cleanup).
    -   `logger.error()`: For serious errors that prevent proper response generation.
    -   `logger.critical()`: For system-breaking errors that require immediate attention.

---

## 8. AI Model Integration

The Python layer handles AI model orchestration and must provide clean abstractions.

-   **Model Abstraction:** Create abstract base classes for different types of AI models (chat, completion, embedding, etc.).
-   **Provider Agnostic:** Design interfaces that can work with multiple AI providers (OpenAI, Anthropic, local models, etc.).
-   **Resource Management:** Implement proper model loading, unloading, and memory management.
-   **Response Streaming:** Support streaming responses for real-time conversation experiences.
-   **Rate Limiting:** Implement rate limiting and quota management for external AI services.

**Example:**

```python
from abc import ABC, abstractmethod
from typing import AsyncIterator, Dict, Any

class AIModel(ABC):
    """Abstract base class for AI model implementations."""

    @abstractmethod
    async def generate_response(
        self, 
        prompt: str, 
        context: Optional[ConversationContext] = None
    ) -> str:
        """Generate a text response to the given prompt."""
        pass

    @abstractmethod
    async def stream_response(
        self, 
        prompt: str, 
        context: Optional[ConversationContext] = None
    ) -> AsyncIterator[str]:
        """Stream a text response token by token."""
        pass
```

---

## 9. Configuration Management

Nina AI requires flexible configuration for different deployment scenarios.

-   **Environment-Based:** Use environment variables for sensitive data (API keys, database URLs).
-   **Configuration Files:** Support YAML/JSON configuration files for model settings, feature flags, and behavior tuning.
-   **Validation:** Use Pydantic models to validate configuration data with clear error messages.
-   **Hot Reloading:** Support configuration updates without full system restart where possible.

---

## 10. Testing (`pytest`)

Comprehensive testing is essential for AI assistant reliability.

-   **Framework:** Use `pytest` for all tests.
-   **Coverage:** Every new feature or bug fix **MUST** be accompanied by tests.
-   **Test Categories:**
    -   Unit tests for individual components
    -   Integration tests for AI model interactions
    -   Conversation flow tests with mock interactions
    -   Performance tests for response time requirements
-   **Fixtures:** Use `pytest` fixtures for setup (mock conversations, test models, temporary config).
-   **Mocking:** Mock external AI services, file systems, and databases during testing.
-   **AI-Specific Testing:** Test conversation flows, context management, and response quality.

---

## 11. Dependencies & Packaging

-   **`pyproject.toml`:** All project metadata and dependencies **MUST** be defined in `pyproject.toml` (PEP 621).
-   **Dependency Groups:** Use optional dependency groups for different use cases:
    -   `dev`: Development tools (testing, linting, documentation)
    -   `openai`: OpenAI API integration
    -   `anthropic`: Anthropic API integration
    -   `local`: Local model support
    -   `web`: Web interface dependencies
-   **Version Pinning:** Pin direct dependencies for reproducible builds while allowing flexibility for library consumers.

---

## Final Check: Nina AI Example

Here is a comprehensive example following all guidelines for a Nina AI conversation handler:

```python
# nina/core/conversation.py

import asyncio
import logging
from datetime import datetime, timedelta
from typing import AsyncIterator, Dict, List, Optional, TypeAlias
from uuid import uuid4

from ..exceptions import ConversationError, ModelError
from ..models.base import AIModel
from ..schemas import Message, ConversationState

logger = logging.getLogger(__name__)

# Nina AI-specific type aliases
ConversationHistory = List[Message]
ResponseMetadata = Dict[str, any]

class ConversationHandler:
    """Handles conversation flow and state management for Nina AI.

    This class manages individual conversation sessions, maintaining context,
    history, and coordinating with AI models to generate appropriate responses.
    """

    def __init__(
        self, 
        model: AIModel, 
        max_history: int = 20,
        context_timeout: timedelta = timedelta(hours=2),
    ):
        self._model = model
        self._max_history = max_history
        self._context_timeout = context_timeout
        self._conversations: Dict[str, ConversationState] = {}
        logger.debug("ConversationHandler initialized with model: %s", model.name)

    async def process_message(
        self, 
        user_message: str, 
        conversation_id: Optional[str] = None
    ) -> tuple[str, str, ResponseMetadata]:
        """Processes a user message and generates Nina's response.

        Manages conversation context, calls the AI model, and updates the
        conversation state with the new interaction.

        Args:
            user_message: The user's input message.
            conversation_id: Optional conversation ID. If None, creates a new conversation.

        Returns:
            A tuple containing (response_text, conversation_id, metadata).

        Raises:
            ConversationError: If conversation state management fails.
            ModelError: If the AI model fails to generate a response.
        """
        if conversation_id is None:
            conversation_id = str(uuid4())
            logger.info("Starting new conversation: %s", conversation_id)

        try:
            # Get or create conversation state
            conversation = await self._get_conversation(conversation_id)
            
            # Add user message to history
            user_msg = Message(
                role="user",
                content=user_message,
                timestamp=datetime.utcnow(),
            )
            conversation.add_message(user_msg)

            # Generate AI response
            response_text = await self._model.generate_response(
                user_message, 
                context=conversation.get_context()
            )

            # Add AI response to history
            ai_msg = Message(
                role="assistant",
                content=response_text,
                timestamp=datetime.utcnow(),
            )
            conversation.add_message(ai_msg)

            # Update conversation state
            conversation.last_activity = datetime.utcnow()
            self._conversations[conversation_id] = conversation

            metadata: ResponseMetadata = {
                "conversation_id": conversation_id,
                "model_name": self._model.name,
                "message_count": len(conversation.messages),
                "response_time": datetime.utcnow().isoformat(),
            }

            logger.info(
                "Generated response for conversation %s (length: %d chars)",
                conversation_id,
                len(response_text),
            )

            return response_text, conversation_id, metadata

        except Exception as e:
            logger.error(
                "Failed to process message for conversation %s: %s",
                conversation_id,
                e,
            )
            raise ConversationError(
                f"Failed to process message: {str(e)}"
            ) from e

    async def stream_response(
        self, 
        user_message: str, 
        conversation_id: str
    ) -> AsyncIterator[str]:
        """Streams AI response tokens for real-time conversation experience.

        Args:
            user_message: The user's input message.
            conversation_id: The conversation identifier.

        Yields:
            Response text tokens as they are generated.

        Raises:
            ConversationError: If conversation state management fails.
            ModelError: If the AI model fails during streaming.
        """
        conversation = await self._get_conversation(conversation_id)
        
        try:
            async for token in self._model.stream_response(
                user_message, 
                context=conversation.get_context()
            ):
                yield token
        except Exception as e:
            logger.error(
                "Streaming failed for conversation %s: %s",
                conversation_id,
                e,
            )
            raise ModelError(f"Response streaming failed: {str(e)}") from e

    async def _get_conversation(self, conversation_id: str) -> ConversationState:
        """Retrieves or creates a conversation state."""
        if conversation_id in self._conversations:
            conversation = self._conversations[conversation_id]
            
            # Check if conversation has timed out
            if datetime.utcnow() - conversation.last_activity > self._context_timeout:
                logger.info("Conversation %s timed out, creating fresh state", conversation_id)
                conversation = ConversationState(conversation_id)
        else:
            conversation = ConversationState(conversation_id)
            
        return conversation

    async def cleanup_expired_conversations(self) -> int:
        """Removes expired conversations to free memory.

        Returns:
            The number of conversations that were cleaned up.
        """
        cutoff_time = datetime.utcnow() - self._context_timeout
        expired_ids = [
            conv_id for conv_id, conv in self._conversations.items()
            if conv.last_activity < cutoff_time
        ]
        
        for conv_id in expired_ids:
            del self._conversations[conv_id]
            
        if expired_ids:
            logger.info("Cleaned up %d expired conversations", len(expired_ids))
            
        return len(expired_ids)
```