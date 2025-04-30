# Controller Integration with Agent

This document explains how the controller functionality works with the agent in the browser-use project.

## Overview

The controller provides a structured way for the agent to interact with the browser by defining a set of actions that can be performed. It acts as a bridge between the agent's high-level instructions and the browser's low-level operations.

## Core Components

### 1. Controller Class

The `Controller` class is the main entry point for action management. It provides:

- Action registration and management
- Action execution
- Result handling
- Context management

Key features:
- Generic type support for custom contexts
- Action registry integration
- Flexible output model support
- Built-in action logging

### 2. Action Registry

The action registry manages all available actions and their parameters. It provides:

- Action registration
- Parameter validation
- Action lookup
- Action execution

## Available Actions

### Navigation Actions

1. **Search Google**
   ```python
   class SearchGoogleAction(BaseModel):
       query: str
   ```
   - Searches Google with the provided query
   - Uses Google's search interface

2. **Go to URL**
   ```python
   class GoToUrlAction(BaseModel):
       url: str
   ```
   - Navigates to a specific URL
   - Handles page load states

3. **Go Back**
   - Navigates to the previous page
   - No parameters required

### Element Interaction Actions

1. **Click Element**
   ```python
   class ClickElementAction(BaseModel):
       index: int
       xpath: Optional[str] = None
   ```
   - Clicks an element by index or XPath
   - Handles file uploaders
   - Manages new tab creation

2. **Input Text**
   ```python
   class InputTextAction(BaseModel):
       index: int
       text: str
       xpath: Optional[str] = None
   ```
   - Inputs text into form elements
   - Supports sensitive data handling

3. **Send Keys**
   ```python
   class SendKeysAction(BaseModel):
       keys: str
   ```
   - Sends keyboard shortcuts
   - Supports special keys and combinations

### Tab Management Actions

1. **Switch Tab**
   ```python
   class SwitchTabAction(BaseModel):
       page_id: int
   ```
   - Switches between browser tabs
   - Handles tab loading states

2. **Open Tab**
   ```python
   class OpenTabAction(BaseModel):
       url: str
   ```
   - Opens a new tab with specified URL
   - Manages tab creation

3. **Close Tab**
   ```python
   class CloseTabAction(BaseModel):
       page_id: int
   ```
   - Closes specified tab
   - Handles tab cleanup

### Scrolling Actions

1. **Scroll Down**
   ```python
   class ScrollAction(BaseModel):
       amount: Optional[int] = None
   ```
   - Scrolls page down
   - Supports custom scroll amounts

2. **Scroll Up**
   - Scrolls page up
   - Supports custom scroll amounts

3. **Scroll to Text**
   - Scrolls to specific text on page
   - Handles text visibility

### Advanced Actions

1. **Drag and Drop**
   ```python
   class DragDropAction(BaseModel):
       element_source: Optional[str]
       element_target: Optional[str]
       element_source_offset: Optional[Position]
       element_target_offset: Optional[Position]
       coord_source_x: Optional[int]
       coord_source_y: Optional[int]
       coord_target_x: Optional[int]
       coord_target_y: Optional[int]
       steps: Optional[int]
       delay_ms: Optional[int]
   ```
   - Supports element-based and coordinate-based drag and drop
   - Handles complex UI interactions

2. **Extract Content**
   - Extracts specific information from pages
   - Supports structured data extraction

3. **Save PDF**
   - Saves current page as PDF
   - Handles media emulation

## Usage with Agent

### Basic Usage

```python
from browser_use.controller import Controller
from browser_use.browser import BrowserContext

# Create controller instance
controller = Controller()

# Execute action
result = await controller.act(
    action=ActionModel(
        name="go_to_url",
        params={"url": "https://example.com"}
    ),
    browser_context=browser_context
)
```

### Action Execution Flow

1. Agent determines required action
2. Controller validates action parameters
3. Action is executed with browser context
4. Result is returned to agent
5. Agent processes result and determines next action

## Best Practices

1. **Action Selection**
   - Choose appropriate action for task
   - Consider action limitations
   - Handle action failures gracefully

2. **Parameter Validation**
   - Validate parameters before execution
   - Handle missing or invalid parameters
   - Use appropriate parameter types

3. **Error Handling**
   - Handle action execution errors
   - Provide meaningful error messages
   - Implement retry mechanisms

4. **Performance**
   - Minimize unnecessary actions
   - Use efficient action sequences
   - Handle action timeouts

## Limitations

1. Actions may fail due to page state changes
2. Some actions require specific page conditions
3. Action execution time varies
4. Some actions may be blocked by websites
5. Complex actions may require multiple steps

## Troubleshooting

1. **Action Failures**
   - Check action parameters
   - Verify page state
   - Check element visibility
   - Handle timeouts

2. **Navigation Issues**
   - Verify URLs
   - Check network connectivity
   - Handle redirects
   - Manage page loads

3. **Element Interaction**
   - Verify element existence
   - Check element visibility
   - Handle dynamic content
   - Manage iframes

4. **Tab Management**
   - Verify tab existence
   - Handle tab creation
   - Manage tab switching
   - Clean up closed tabs 