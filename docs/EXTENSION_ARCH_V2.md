# Architecture Document: Browser-Use Chrome Extension V2

## 1. Overview

This document describes the modular, extensible architecture for Browser-Use Chrome Extension V2, supporting automation recording/replay, task management, data integration, continual learning, and shadow credentials. The design is based on the requirements in EXTENSION_PRD_V2.md and is optimized for maintainability, security, and future extensibility.

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
    automationRecorder.js
    automationReplayer.js
    taskManager.js
    ragEngine.js
    shadowCredentials.js
    dataIntegration.js
  content/
    contentScript.js
    domInteractor.js
    messageRouter.js
    recorderProxy.js
    shadowCredentialInjector.js
  popup/
    popup.html
    popup.js
    popup.css
    messageRouter.js
    tasksTab.js
    dataTab.js
    notifications.js
    controls.js
    settings.js
  storage/
    chromeStorage.js
    indexedDb.js
    sessionManager.js
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
    versioning.js
    fileUtils.js
  README.md
```

---

## 3. Component Overview

### 3.1 Background Service (`background/`)
- **background.js**: Main entry, orchestrates actions, manages memory, coordinates modules.
- **actionRegistry.js**: Registers, validates, and executes static/dynamic actions.
- **memory.js**: Manages in-memory state, automation/session history.
- **automationRecorder.js**: Captures user/browser actions as high-level logs and code.
- **automationReplayer.js**: Replays, modifies, and optimizes automations.
- **taskManager.js**: Manages automations as tasks, versioning, grouping, and status.
- **ragEngine.js**: Retrieval-augmented generation for continual learning from past sessions.
- **shadowCredentials.js**: Securely manages and injects shadow credentials.
- **dataIntegration.js**: Handles Google Sheets, file upload/download, PDF integration.
- **openaiClient.js**: Handles LLM API calls.
- **messageRouter.js**: Routes messages using abstracted messaging.

### 3.2 Content Service (`content/`)
- **contentScript.js**: Injected into web pages, interacts with DOM, relays info.
- **domInteractor.js**: Encapsulates DOM actions (click, input, scroll, extract, highlight).
- **recorderProxy.js**: Captures user actions/events for recording.
- **shadowCredentialInjector.js**: Injects credentials securely into page context.
- **messageRouter.js**: Handles messaging with background.

### 3.3 Popup UI (`popup/`)
- **popup.html/js/css**: Minimal UI, with tabs for Tasks, Data, and Controls.
- **tasksTab.js**: Displays automations grouped by website/page, with expand/collapse, versioning, and replay/edit controls.
- **dataTab.js**: UI for uploading/downloading data files, Google Sheets, PDFs.
- **notifications.js**: User notifications for automation status, errors, and interventions.
- **controls.js**: Start/Stop/Pause/Continue/Exit automation controls.
- **settings.js**: Manage API keys, endpoints, and shadow credentials.
- **messageRouter.js**: Messaging abstraction for popup.

### 3.4 Storage (`storage/`)
- **chromeStorage.js**: Wrapper for Chrome Storage API (settings, small data).
- **indexedDb.js**: For large automation histories, session data, and file storage.
- **sessionManager.js**: Handles session versioning, rollback, and local/remote session management.

### 3.5 Messaging Abstraction (`messaging/`)
- **messaging.js**: Abstract interface for messaging.
- **chromeMessaging.js**: Chrome-specific implementation.

### 3.6 Utilities (`utils/`)
- **logger.js**: Logging utility.
- **errorHandler.js**: Centralized error handling.
- **versioning.js**: Automation versioning and rollback.
- **fileUtils.js**: File parsing, download, and upload helpers.

### 3.7 Tests (`tests/`)
- **e2e/**: End-to-end tests with mock pages, data, and user actions.
- **unit/**: Unit tests for each module.

---

## 4. Sequence Diagrams

### 4.1 Recording an Automation
```
User -> Popup (Start Recording) -> Background (automationRecorder) -> Content (recorderProxy)
User performs actions -> recorderProxy captures events -> automationRecorder logs actions
User -> Popup (Stop Recording) -> Background (automationRecorder saves session)
```

### 4.2 Replaying an Automation
```
User -> Popup (Replay) -> Background (automationReplayer) -> Content (domInteractor)
AutomationReplayer steps through actions -> domInteractor executes on page
Status/results relayed back to Popup
```

### 4.3 Data Integration (Google Sheets/File Upload)
```
User -> Popup (Upload Data) -> Background (dataIntegration)
Background (OAuth if needed) -> Fetch/process data -> Store in session
Automation uses data rows for actions
Results -> Background (dataIntegration) -> Popup (Download/Export)
```

---

## 5. Skeleton Classes & Methods

### 5.1 Automation Recorder
```js
// background/automationRecorder.js
export class AutomationRecorder {
  startRecording(context) { /* Begin capturing events */ }
  stopRecording() { /* Stop and save session */ }
  logAction(action) { /* Add action to current session */ }
  getSession() { /* Return current session */ }
}
```

### 5.2 Automation Replayer
```js
// background/automationReplayer.js
export class AutomationReplayer {
  loadSession(sessionId) { /* Load automation session */ }
  replay(session, options) { /* Step through actions */ }
  pause() { /* Pause replay */ }
  continue() { /* Continue replay */ }
  exit() { /* Exit replay */ }
}
```

### 5.3 Task Manager
```js
// background/taskManager.js
export class TaskManager {
  addTask(session) { /* Add new automation */ }
  getTasks(filter) { /* List/group automations */ }
  updateTask(taskId, updates) { /* Edit automation */ }
  versionTask(taskId) { /* Version/rollback */ }
}
```

### 5.4 Data Integration
```js
// background/dataIntegration.js
export class DataIntegration {
  importGoogleSheet(sheetId, auth) { /* OAuth, fetch, parse */ }
  importFile(file) { /* Parse CSV/JSON */ }
  exportResults(format, data) { /* Download as CSV/JSON/PDF */ }
}
```

### 5.5 Shadow Credentials
```js
// background/shadowCredentials.js
export class ShadowCredentials {
  setCredential(site, creds) { /* Store securely */ }
  getCredential(site) { /* Retrieve for injection */ }
  injectCredential(tabId, site) { /* Use content script to inject */ }
}
```

### 5.6 RAG Engine (Continual Learning)
```js
// background/ragEngine.js
export class RAGEngine {
  retrieveRelevantSessions(context) { /* Retrieve past sessions */ }
  updateWithOutcome(session, outcome) { /* Learn from success/failure */ }
}
```

---

## 6. Security & Privacy Best Practices
- Shadow credentials are never exposed to LLMs or external APIs; only injected in page context.
- All sensitive data is encrypted in storage and cleared on uninstall.
- Content scripts run in isolated worlds; only minimal permissions are requested.
- No user data is uploaded except high-level, non-identifiable metrics.
- OAuth tokens and credentials are never stored in content scripts or UI.
- Use CSP in manifest to restrict resource loading.

---

## 7. Configuration Management
- API keys, endpoints, and shadow credentials are stored in Chrome Storage (never hardcoded).
- Settings UI in popup/settings.js for user to update config.
- Default values with user override capability.
- Sensitive config is never exposed to content scripts or UI.

---

## 8. Chrome Extension Restrictions & Workarounds
- No eval or dynamic code loading; all code must be statically included.
- No direct network requests from content scripts; use background for API calls.
- Service worker (background) is event-driven; avoid long-running tasks.
- Use IndexedDB for large data, Chrome Storage for settings/history.
- No Node.js APIs; use only browser-compatible JS APIs.

---

## 9. How to Add New Automations or Data Integrations
1. Implement new action/data handler in background (e.g., in actionRegistry.js or dataIntegration.js).
2. Register the handler in the appropriate registry/module.
3. Add UI in popup if user interaction is needed.
4. Add content script logic if DOM or credential injection is required.
5. Add unit and E2E tests for the new feature.

---

## 10. Future Extensibility
- Modular design allows for new data sources, automation types, and sharing features.
- RAG engine and versioning support continual improvement and rollback.
- Data integration layer can be extended for new file types/APIs.
- Shadow credentials system can be enhanced for more complex auth flows.
- UI tabs/components can be expanded for new features (e.g., scheduling, notifications, template gallery).

---

## 11. Example: Registering a New Data-Driven Automation
```js
// background/actionRegistry.js
function fillFormWithSheetData(params) {
  // Use DataIntegration to fetch data, then DOMInteractor to fill form
}
actionRegistry.registerAction('fillFormWithSheetData', fillFormWithSheetData);
``` 