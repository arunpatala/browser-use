# Browser-Use Architecture

## Overview

**Browser-Use** is a Python library that enables AI agents to control and automate web browsers. It is designed to make websites accessible for AI agents, supporting advanced browser automation, DOM extraction, memory, and extensibility for custom workflows. The architecture is modular, with clear separation between the agent logic, browser control, DOM extraction, action registry, memory, telemetry, and utilities.

---

## High-Level Architecture Diagram

```mermaid
graph TD
    subgraph Agent
        A1[Agent Service]
        A2[Message Manager]
        A3[Memory]
        A4[Agent Views]
    end
    subgraph Browser
        B1[Browser]
        B2[Browser Context]
        B3[Browser Views]
    end
    subgraph DOM
        D1[DOM Service]
        D2[DOM Views]
    end
    subgraph Controller
        C1[Controller Service]
        C2[Action Registry]
        C3[Controller Views]
    end
    subgraph Telemetry
        T1[Telemetry Service]
        T2[Telemetry Views]
    end
    subgraph Utils
        U1[Utilities]
    end

    A1 --controls--> B1
    B1 --manages--> B2
    B2 --extracts--> D1
    D1 --provides--> A1
    A1 --executes actions via--> C1
    C1 --uses--> C2
    A1 --stores state in--> A3
    A1 --logs events to--> T1
    U1 --used by all-->
```

---

## Main Components

### 1. Agent
- **Location:** `browser_use/agent/`
- **Key Files:** `service.py`, `views.py`, `memory/service.py`, `message_manager/`
- **Responsibilities:**
  - Orchestrates the entire automation process.
  - Manages the task, state, and step execution.
  - Interacts with the LLM (via LangChain) to plan and decide actions.
  - Maintains conversation and procedural memory (via Mem0 or embeddings).
  - Handles agent history, error handling, and extensibility hooks.
  - Uses the Message Manager to manage prompt history and context.
- **Extensibility:**
  - Custom actions can be added via the controller registry.
  - Memory and planner LLMs can be swapped/configured.

### 2. Browser
- **Location:** `browser_use/browser/`
- **Key Files:** `browser.py`, `context.py`, `views.py`, `chrome.py`, `utils/`
- **Responsibilities:**
  - Wraps and extends Playwright (via Patchright) for browser automation.
  - Manages browser sessions, contexts, tabs, navigation, and actions.
  - Handles browser configuration (headless, security, proxy, etc.).
  - Provides APIs for tab management, navigation, screenshots, and more.
- **Extensibility:**
  - Supports custom browser binaries, remote debugging, and context configs.

### 3. DOM Extraction
- **Location:** `browser_use/dom/`
- **Key Files:** `service.py`, `buildDomTree.js`, `views.py`, `clickable_element_processor/`
- **Responsibilities:**
  - Extracts and represents the DOM structure for the agent.
  - Identifies clickable/interactable elements and their properties.
  - Uses injected JavaScript (`buildDomTree.js`) for efficient DOM parsing.
  - Provides a structured, token-efficient representation for LLM input.
- **Extensibility:**
  - Custom DOM extraction logic can be added for special elements.

### 4. Controller & Action Registry
- **Location:** `browser_use/controller/`
- **Key Files:** `service.py`, `registry/`, `views.py`
- **Responsibilities:**
  - Maintains a registry of all possible actions the agent can perform.
  - Maps LLM-decided actions to concrete browser operations (click, input, scroll, etc.).
  - Handles action execution, error handling, and dynamic action models.
- **Extensibility:**
  - New actions can be registered easily via the registry pattern.

### 5. Memory
- **Location:** `browser_use/agent/memory/`
- **Key Files:** `service.py`, `views.py`
- **Responsibilities:**
  - Implements procedural memory using embeddings (Mem0, HuggingFace, OpenAI, etc.).
  - Summarizes and compresses agent history to optimize context window usage.
  - Supports RAG and other memory strategies.
- **Extensibility:**
  - Pluggable memory backends and configuration.

### 6. Telemetry
- **Location:** `browser_use/telemetry/`
- **Key Files:** `service.py`, `views.py`
- **Responsibilities:**
  - Captures anonymized usage and error telemetry (via Posthog).
  - Can be disabled via environment variable.
  - Helps improve the product and monitor agent runs.

### 7. Utilities
- **Location:** `browser_use/utils.py`
- **Responsibilities:**
  - Provides cross-cutting utilities: signal handling, timing, singleton, environment checks, etc.
  - Used throughout the codebase for robustness and developer ergonomics.

---

## How a Typical Agent Run Works

1. **Initialization:**
   - The user creates an `Agent` with a task and an LLM (e.g., OpenAI, Gemini, etc.).
   - The agent initializes the browser, controller, memory, and message manager.

2. **Step Loop:**
   - For each step, the agent:
     - Extracts the current browser/DOM state.
     - Updates available actions based on the page.
     - Prepares the prompt and context for the LLM.
     - Calls the LLM to decide the next action(s).
     - Executes the chosen actions via the controller (which uses the browser and DOM services).
     - Updates memory and history.
     - Logs telemetry and handles errors.
     - Optionally generates a GIF of the step.

3. **Completion:**
   - The agent stops when the task is done, max steps are reached, or an error occurs.
   - Final results, history, and artifacts are saved.

---

## Extensibility Points

- **Actions:** Add new browser actions by registering them in the controller registry.
- **Memory:** Swap or configure memory backends and summarization strategies.
- **LLM:** Use any LangChain-compatible LLM for planning and extraction.
- **UI:** Integrate with UI frontends (e.g., Gradio demo, web UI repo).
- **Telemetry:** Can be disabled or extended for custom analytics.

---

## File/Module Map

- `browser_use/agent/` — Agent logic, memory, message management, prompts, and views.
- `browser_use/browser/` — Browser automation, context/session management, browser config.
- `browser_use/dom/` — DOM extraction, clickable element processing, JS injection.
- `browser_use/controller/` — Action registry, controller logic, action views.
- `browser_use/telemetry/` — Telemetry and analytics.
- `browser_use/utils.py` — Utilities and helpers.
- `examples/` — Example scripts and use cases.
- `tests/` — Test suite.

---

## Summary

Browser-Use is a modular, extensible framework for AI-driven browser automation. Its architecture separates agent logic, browser control, DOM extraction, action execution, memory, and telemetry, making it easy to extend and adapt for new use cases, LLMs, and workflows. The codebase is designed for both research and production, with a focus on developer experience and robust automation. 