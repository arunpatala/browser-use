# Architecture Document: Browser-Use Chrome Extension

## 1. Overview

This document outlines the modular architecture, file structure, and code skeleton for the Browser-Use Chrome Extension, designed for extensibility, maintainability, and compliance with Chrome Manifest V3. The extension enables AI-powered browser automation, DOM interaction, and task management, with a focus on minimal UI, robust action registry, and abstracted inter-component communication.

---

## 2. File & Folder Structure

```
extension/
  manifest.json
  background/
    background.js
    actionRegistry.js
    memory.js
    messageRouter.js
    openaiClient.js
  content/
    contentScript.js
    domInteractor.js
    messageRouter.js
  popup/
    popup.html
    popup.js
    popup.css
    messageRouter.js
  storage/
    chromeStorage.js
  messaging/
    messaging.js
    chromeMessaging.js
  tests/
    e2e/
      e2e.test.js
    unit/
      background/
      content/
      popup/
  utils/
    logger.js
    errorHandler.js
  README.md
```

---

## 2.1 Sample manifest.json (Outline)

```json
{
  "manifest_version": 3,
  "name": "Browser-Use Automation Extension",
  "version": "0.1.0",
  "description": "AI-powered browser automation and DOM interaction.",
  "permissions": [
    "storage",
    "scripting",
    "activeTab"
  ],
  "host_permissions": [
    "<all_urls>"
  ],
  "background": {
    "service_worker": "background/background.js"
  },
  "action": {
    "default_popup": "popup/popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content/contentScript.js"]
    }
  ],
  "icons": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}
```

---

## 2.2 High-Level Sequence Diagram

```
User -> Popup UI -> Background Service -> Content Script -> DOM Interactor
 ^         |                |                |
 |         v                v                v
User views history   ActionRegistry   ChromeMessaging   DOM actions (click, input, etc.)
```

**Example: User triggers "click" action from popup:**
1. User clicks "Run Click Action" in popup.
2. Popup sends message to Background via Messaging abstraction.
3. Background validates and dispatches action via ActionRegistry.
4. Background sends message to Content Script for DOM interaction.
5. Content Script uses DOMInteractor to perform click.
6. Result/status is relayed back to Popup for user feedback.

---

## 2.3 Security & Privacy Best Practices
- Use Chrome's isolated worlds for content scripts to prevent data leakage.
- Request only minimal permissions in manifest.json.
- Validate and sanitize all user input and action parameters.
- Encrypt sensitive data in storage if needed; clear on uninstall.
- Never inject remote scripts; all code must be packaged.
- Use CSP (Content Security Policy) in manifest to restrict resource loading.

---

## 2.4 Configuration Management
- Store API keys and endpoint URLs in Chrome Storage (never hardcoded).
- Provide a settings section in popup for user to update config.
- Use default values with user override capability.
- Never expose sensitive config in content scripts or UI.

---

## 2.5 Chrome Extension Restrictions & Workarounds
- **No eval or dynamic code loading:** All code must be statically included.
- **No direct network requests from content scripts:** Use background for API calls.
- **Service worker (background) is event-driven:** Avoid long-running tasks; use alarms or message passing for async work.
- **Limited storage quota:** Use IndexedDB for large data, Chrome Storage for settings/history.
- **No Node.js APIs:** Use only browser-compatible JS APIs.

---

## 2.6 How to Add a New Action
1. In `background/actionRegistry.js`, define a new handler function:
   ```js
   function myNewAction(params) { /* ... */ }
   ```
2. Register the action in the ActionRegistry instance:
   ```js
   actionRegistry.registerAction('myNewAction', myNewAction);
   ```
3. (Optional) Add UI in popup to trigger the new action.
4. (Optional) Add content script logic if DOM interaction is required.
5. Add unit and E2E tests for the new action.

---

## 2.7 Example: Registering a Click Action
```js
// background/actionRegistry.js
function clickAction(params) {
  // Validate params, send message to content script to perform click
}
actionRegistry.registerAction('click', clickAction);
```

---

## 3. Component Overview

### 3.1 Background Service (`background/`)
- **background.js**: Main entry, orchestrates actions, manages memory, communicates with content/popup.
- **actionRegistry.js**: Registers, validates, and executes static actions.
- **memory.js**: Manages in-memory state and history.
- **messageRouter.js**: Routes messages between background, content, and popup using abstracted messaging.
- **openaiClient.js**: Handles LLM API calls with configurable URL and API key.

### 3.2 Content Service (`content/`)
- **contentScript.js**: Injected into web pages, interacts with DOM, relays info.
- **domInteractor.js**: Encapsulates DOM actions (click, input, scroll, extract, highlight).
- **messageRouter.js**: Handles messaging with background using abstraction.

### 3.3 Popup UI (`popup/`)
- **popup.html**: Minimal UI for task management and settings.
- **popup.js**: Handles UI logic, user actions, and messaging.
- **popup.css**: Styles for popup UI.
- **messageRouter.js**: Messaging abstraction for popup.

### 3.4 Storage (`storage/`)
- **chromeStorage.js**: Wrapper for Chrome Storage API (local, sync).

### 3.5 Messaging Abstraction (`messaging/`)
- **messaging.js**: Abstract interface for messaging.
- **chromeMessaging.js**: Chrome-specific implementation.

### 3.6 Utilities (`utils/`)
- **logger.js**: Logging utility.
- **errorHandler.js**: Centralized error handling.

### 3.7 Tests (`tests/`)
- **e2e/**: End-to-end tests with mock pages and user actions.
- **unit/**: Unit tests for each module.

---

## 4. Skeleton Classes & Methods

### 4.1 Messaging Abstraction

```js
// messaging/messaging.js
export class Messaging {
  // Abstract send method
  sendMessage(target, message) { throw new Error('Not implemented'); }
  // Abstract listener registration
  onMessage(handler) { throw new Error('Not implemented'); }
}

// messaging/chromeMessaging.js
import { Messaging } from './messaging.js';
export class ChromeMessaging extends Messaging {
  sendMessage(target, message) { /* Chrome runtime.sendMessage logic */ }
  onMessage(handler) { /* Chrome runtime.onMessage logic */ }
}
```

### 4.2 Action Registry

```js
// background/actionRegistry.js
export class ActionRegistry {
  constructor() { this.actions = {}; }
  registerAction(name, handler) { this.actions[name] = handler; }
  executeAction(name, params) {
    if (!this.actions[name]) throw new Error('Action not found');
    return this.actions[name](params);
  }
  listActions() { return Object.keys(this.actions); }
}
```

### 4.3 OpenAI Client

```js
// background/openaiClient.js
export class OpenAIClient {
  constructor(apiUrl, apiKey) { this.apiUrl = apiUrl; this.apiKey = apiKey; }
  async callLLM(prompt) {
    // Fetch call to OpenAI-compatible endpoint
  }
}
```

### 4.4 DOM Interactor

```js
// content/domInteractor.js
export class DOMInteractor {
  click(selector) { /* ... */ }
  input(selector, value) { /* ... */ }
  scroll(selector) { /* ... */ }
  extract(selector) { /* ... */ }
  highlight(selector) { /* ... */ }
}
```

### 4.5 Chrome Storage Wrapper

```js
// storage/chromeStorage.js
export class ChromeStorage {
  get(key) { /* ... */ }
  set(key, value) { /* ... */ }
  remove(key) { /* ... */ }
  clear() { /* ... */ }
}
```

### 4.6 Logger & Error Handler

```js
// utils/logger.js
export function log(...args) { /* ... */ }
// utils/errorHandler.js
export function handleError(error) { /* ... */ }
```

### 4.7 Popup UI

```js
// popup/popup.js
document.addEventListener('DOMContentLoaded', () => {
  // Initialize UI, bind events, communicate with background
});
```

---

## 5. Testing Scaffold

- **E2E:** Use Puppeteer or Playwright with mock extension environment.
- **Unit:** Use Jest or Mocha for JS modules.

```js
// tests/e2e/e2e.test.js
describe('Browser-Use Extension E2E', () => {
  it('should perform a click action', async () => { /* ... */ });
});
```

---

## 6. Design Principles
- **Low Coupling:** Each module/class exposes a clear interface and depends only on abstractions.
- **Extensibility:** New actions, storage backends, or messaging implementations can be added with minimal changes.
- **Manifest V3 Compliance:** All code and permissions are compatible with Chrome Manifest V3.
- **Minimal UI:** Popup is simple, focused on task management and settings.
- **Testing:** E2E and unit test scaffolding included from the start.

---

## 7. Future Extensions
- Support for dynamic/plugin-based actions.
- Cloud backup and sync.
- Internationalization (i18n).
- Support for other browsers via new messaging/storage implementations. 