# Page Context in Browser-Use

This document explains how Browser-Use prepares and sends page context to the LLM, including both vision and non-vision modes.

## Overview

When the agent needs to make a decision about the next action, it needs to understand the current state of the page. This is done by:
1. Extracting the DOM structure and clickable elements
2. Capturing the page state (text, screenshots, etc.)
3. Formatting this information for the LLM
4. Sending it as part of the prompt

## Page Context Components

### 1. DOM Structure
The DOM structure is extracted using the `DomService` class (`browser_use/dom/service.py`). It:
- Injects and executes `buildDomTree.js` to analyze the page
- Creates a tree representation of the DOM
- Identifies clickable/interactive elements
- Assigns indices to clickable elements for reference

### 2. Page State
The page state is captured by the `BrowserContext` class (`browser_use/browser/context.py`) and includes:
- Current URL
- Page title
- Screenshot (if vision is enabled)
- Tab information
- DOM state with clickable elements

## Context Formatting

### Without Vision

When vision is disabled (`use_vision=False`), the context includes:

1. **Text Content:**
   ```text
   Current URL: https://example.com
   Page Title: Example Website
   
   Clickable Elements:
   [1] Button: "Add to Cart" (type="submit", class="btn-primary")
   [2] Link: "View Details" (href="/details/123")
   [3] Input: "Search" (type="text", placeholder="Search products...")
   
   Page Content:
   Welcome to our store!
   Featured Products:
   - Product 1: $99.99
   - Product 2: $149.99
   ```

2. **Element Attributes:**
   - Only essential attributes are included to save tokens
   - Default attributes: title, type, name, role, aria-label, placeholder, value, alt, aria-expanded
   - Custom attributes can be added via `include_attributes` in `AgentSettings`

### With Vision

When vision is enabled (`use_vision=True`), the context includes:

1. **Base64 Screenshot:**
   ```text
   Current URL: https://example.com
   Page Title: Example Website
   
   Screenshot: [base64_encoded_image]
   
   Clickable Elements:
   [1] Button: "Add to Cart" (type="submit", class="btn-primary")
   [2] Link: "View Details" (href="/details/123")
   [3] Input: "Search" (type="text", placeholder="Search products...")
   
   Page Content:
   Welcome to our store!
   Featured Products:
   - Product 1: $99.99
   - Product 2: $149.99
   ```

2. **Element Highlighting:**
   - Clickable elements are highlighted in the screenshot
   - Each element's index corresponds to its position in the clickable elements list
   - The LLM can reference elements by their index in actions

## Code Flow

1. **State Extraction:**
   ```python
   # In BrowserContext.get_state()
   state = await self._get_updated_state()
   # This calls DomService.get_clickable_elements()
   # and captures screenshots if vision is enabled
   ```

2. **Message Preparation:**
   ```python
   # In MessageManager.add_state_message()
   state_message = self._format_state_message(state, use_vision)
   self._add_message_with_tokens(state_message)
   ```

3. **LLM Input:**
   ```python
   # In Agent.step()
   input_messages = self._message_manager.get_messages()
   model_output = await self.get_next_action(input_messages)
   ```

## Example Context

### Without Vision
```text
Current URL: https://shopping.com
Page Title: Online Store

Clickable Elements:
[1] Input: "Search" (type="text", placeholder="Search products...")
[2] Button: "Search" (type="submit")
[3] Link: "Cart" (href="/cart")
[4] Button: "Add to Cart" (type="button", class="btn-primary")

Page Content:
Welcome to our online store!
Featured Products:
- Organic Apples: $2.99/lb
- Fresh Milk: $3.49
- Whole Grain Bread: $4.99
```

### With Vision
```text
Current URL: https://shopping.com
Page Title: Online Store

Screenshot: [base64_encoded_image]

Clickable Elements:
[1] Input: "Search" (type="text", placeholder="Search products...")
[2] Button: "Search" (type="submit")
[3] Link: "Cart" (href="/cart")
[4] Button: "Add to Cart" (type="button", class="btn-primary")

Page Content:
Welcome to our online store!
Featured Products:
- Organic Apples: $2.99/lb
- Fresh Milk: $3.49
- Whole Grain Bread: $4.99
```

## Best Practices

1. **Token Optimization:**
   - Only include essential attributes
   - Use `viewport_expansion` to control how many elements are included
   - Set `max_input_tokens` to prevent context overflow

2. **Vision Usage:**
   - Enable vision for complex UIs or when element identification is difficult
   - Disable vision for simpler tasks to save tokens and processing time
   - Use `use_vision_for_planner` to enable vision only for planning steps

3. **Element Selection:**
   - Use clear, unique indices for clickable elements
   - Include relevant attributes for element identification
   - Consider using `aria-label` and other accessibility attributes

## Configuration

The page context can be configured through `AgentSettings`:

```python
agent = Agent(
    task="Your task",
    llm=ChatOpenAI(model="gpt-4o"),
    use_vision=True,  # Enable/disable vision
    use_vision_for_planner=False,  # Enable vision only for planning
    include_attributes=[  # Custom attributes to include
        'title',
        'type',
        'name',
        'role',
        'aria-label',
        'placeholder',
        'value',
        'alt',
        'aria-expanded',
    ],
    max_input_tokens=128000,  # Maximum tokens for context
    viewport_expansion=0,  # How many pixels to expand viewport
)
```

This configuration allows fine-tuning of the page context to balance between completeness and efficiency. 