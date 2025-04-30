# Browser Integration with Agent

This document explains how the browser functionality works with the agent in the browser-use project.

## Overview

The browser integration provides a powerful way for the agent to interact with web pages using Playwright. It offers a rich set of features for web automation, including:

- Browser session management
- Page navigation and interaction
- DOM manipulation
- Screenshot capture
- Cookie management
- Tab management
- Network request monitoring
- Element highlighting and interaction

## Available Tools

The browser integration provides several tools for interacting with web pages:

1. **Navigation Tools**
   - `navigate_to(url)`: Navigate to a specific URL
   - `go_back()`: Navigate to the previous page
   - `go_forward()`: Navigate to the next page
   - `refresh_page()`: Reload the current page

2. **Element Interaction Tools**
   - `click_element(index)`: Click an element by its index
   - `input_text(index, text)`: Input text into a form element
   - `send_keys(keys)`: Send keyboard shortcuts
   - `scroll_down(amount)`: Scroll the page down
   - `scroll_up(amount)`: Scroll the page up
   - `scroll_to_text(text)`: Scroll to specific text on the page

3. **Tab Management Tools**
   - `create_new_tab(url)`: Open a new tab with optional URL
   - `switch_to_tab(page_id)`: Switch to a specific tab
   - `close_tab(page_id)`: Close a specific tab
   - `get_tabs_info()`: Get information about all open tabs

4. **State Management Tools**
   - `get_state()`: Get the current page state
   - `get_page_html()`: Get the page's HTML content
   - `get_page_structure()`: Get the page's DOM structure
   - `take_screenshot(full_page)`: Capture a screenshot

5. **Network Tools**
   - `wait_for_network_idle()`: Wait for network requests to complete
   - `get_network_requests()`: Get information about network requests
   - `save_cookies()`: Save browser cookies
   - `load_cookies()`: Load saved cookies

6. **DOM Tools**
   - `get_element_by_index(index)`: Get an element by its index
   - `get_element_by_xpath(xpath)`: Get an element by XPath
   - `get_element_by_css_selector(selector)`: Get an element by CSS selector
   - `get_element_by_text(text)`: Get an element by its text content

7. **Advanced Tools**
   - `drag_drop(source, target)`: Perform drag and drop operations
   - `save_pdf()`: Save the current page as PDF
   - `execute_javascript(script)`: Execute JavaScript code
   - `extract_content(goal)`: Extract specific information from the page

## Core Components

### 1. Browser Class

The `Browser` class is the main entry point for browser automation. It provides:

- Browser instance management
- Context creation and management
- Connection to remote browsers (via CDP or WSS)
- Support for different browser types (Chromium, Firefox, WebKit)

Key features:
- Configurable browser settings (headless mode, security settings, etc.)
- Support for custom browser binaries
- Remote debugging capabilities
- Proxy support

### 2. BrowserContext

The `BrowserContext` class manages individual browser sessions and provides:

- Page management
- Navigation controls
- DOM interaction
- State management
- Screenshot capabilities
- Cookie handling
- Tab management

Key features:
- Configurable window size and viewport
- Network request monitoring
- Page load state management
- Element highlighting
- JavaScript execution
- Screenshot capture
- Cookie persistence

## Configuration

### Browser Configuration

The `BrowserConfig` class allows customization of browser behavior:

```python
config = BrowserConfig(
    headless=False,  # Run in visible mode
    disable_security=False,  # Security features enabled
    browser_class='chromium',  # Use Chromium browser
    browser_binary_path=None,  # Use default browser
    chrome_remote_debugging_port=9222,  # Default debugging port
    extra_browser_args=[],  # Additional browser arguments
    keep_alive=False,  # Close browser after use
    deterministic_rendering=False  # Consistent rendering across platforms
)
```

### Context Configuration

The `BrowserContextConfig` class provides fine-grained control over browser contexts:

```python
context_config = BrowserContextConfig(
    cookies_file=None,  # Cookie persistence
    minimum_wait_page_load_time=0.25,  # Minimum page load wait
    wait_for_network_idle_page_load_time=0.5,  # Network idle wait
    maximum_wait_page_load_time=5.0,  # Maximum page load wait
    wait_between_actions=0.5,  # Action delay
    browser_window_size=BrowserContextWindowSize(width=1280, height=1100),
    highlight_elements=True,  # Element highlighting
    viewport_expansion=0,  # Viewport expansion
    allowed_domains=None,  # Domain restrictions
    include_dynamic_attributes=True,  # Dynamic attribute inclusion
    http_credentials=None,  # HTTP authentication
    is_mobile=None,  # Mobile emulation
    has_touch=None,  # Touch events
    geolocation=None,  # Location spoofing
    permissions=None,  # Browser permissions
    timezone_id=None  # Timezone setting
)
```

## Usage with Agent

### Basic Usage

```python
from browser_use.browser import Browser, BrowserConfig

# Create browser instance
browser = Browser(config=BrowserConfig())

# Create context
context = await browser.new_context()

# Navigate to page
await context.navigate_to("https://example.com")

# Get page state
state = await context.get_state(cache_clickable_elements_hashes=True)

# Take screenshot
screenshot = await context.take_screenshot()

# Close context
await context.close()
```

### Advanced Features

1. **Element Interaction**
   - Click elements
   - Input text
   - Scroll pages
   - Handle file uploads

2. **Tab Management**
   - Create new tabs
   - Switch between tabs
   - Close tabs
   - Get tab information

3. **Network Monitoring**
   - Track network requests
   - Monitor response times
   - Handle authentication
   - Manage cookies

4. **State Management**
   - Track page state
   - Cache element information
   - Handle navigation events
   - Manage browser sessions

## Best Practices

1. **Resource Management**
   - Always close contexts when done
   - Use `async with` for automatic cleanup
   - Monitor memory usage
   - Handle browser crashes gracefully

2. **Performance**
   - Use appropriate wait times
   - Cache element information when possible
   - Minimize unnecessary page reloads
   - Use efficient selectors

3. **Security**
   - Be cautious with `disable_security`
   - Validate URLs before navigation
   - Handle sensitive data carefully
   - Use appropriate permissions

4. **Error Handling**
   - Handle navigation errors
   - Manage timeouts
   - Handle element not found cases
   - Implement retry mechanisms

## Limitations

1. Browser automation may be detected by websites
2. Some websites may block automated access
3. Performance depends on network conditions
4. Memory usage can grow with multiple contexts
5. Some features may not work in headless mode

## Troubleshooting

1. **Browser Connection Issues**
   - Check browser binary path
   - Verify debugging port availability
   - Check network connectivity
   - Verify browser version compatibility

2. **Element Interaction Problems**
   - Verify element visibility
   - Check for iframe contexts
   - Handle dynamic content
   - Use appropriate wait times

3. **Performance Issues**
   - Monitor memory usage
   - Check network conditions
   - Optimize wait times
   - Use efficient selectors

4. **Security Concerns**
   - Review security settings
   - Validate URLs
   - Handle sensitive data
   - Use appropriate permissions

## Chrome Extension Compatibility

The following table analyzes which tools can be implemented in a Chrome extension (either background.js or content.js) and which cannot:

| Tool | Implementable in Extension | Location | Reason |
|------|---------------------------|----------|---------|
| **Navigation Tools** |
| `navigate_to(url)` | ✅ Yes | Background | Can use `chrome.tabs.update()` |
| `go_back()` | ✅ Yes | Background | Can use `chrome.tabs.goBack()` |
| `go_forward()` | ✅ Yes | Background | Can use `chrome.tabs.goForward()` |
| `refresh_page()` | ✅ Yes | Background | Can use `chrome.tabs.reload()` |
| **Element Interaction Tools** |
| `click_element(index)` | ✅ Yes | Content | Can use `document.querySelector()` and `click()` |
| `input_text(index, text)` | ✅ Yes | Content | Can use `document.querySelector()` and `value` property |
| `send_keys(keys)` | ✅ Yes | Content | Can use `KeyboardEvent` |
| `scroll_down(amount)` | ✅ Yes | Content | Can use `window.scrollBy()` |
| `scroll_up(amount)` | ✅ Yes | Content | Can use `window.scrollBy()` |
| `scroll_to_text(text)` | ✅ Yes | Content | Can use `document.querySelector()` and `scrollIntoView()` |
| **Tab Management Tools** |
| `create_new_tab(url)` | ✅ Yes | Background | Can use `chrome.tabs.create()` |
| `switch_to_tab(page_id)` | ✅ Yes | Background | Can use `chrome.tabs.update()` |
| `close_tab(page_id)` | ✅ Yes | Background | Can use `chrome.tabs.remove()` |
| `get_tabs_info()` | ✅ Yes | Background | Can use `chrome.tabs.query()` |
| **State Management Tools** |
| `get_state()` | ✅ Yes | Content | Can use `document.documentElement.outerHTML` |
| `get_page_html()` | ✅ Yes | Content | Can use `document.documentElement.outerHTML` |
| `get_page_structure()` | ✅ Yes | Content | Can use `document.documentElement.outerHTML` |
| `take_screenshot(full_page)` | ❌ No | N/A | Requires browser-level permissions not available to extensions |
| **Network Tools** |
| `wait_for_network_idle()` | ❌ No | N/A | Requires browser-level network monitoring |
| `get_network_requests()` | ✅ Yes | Background | Can use `chrome.webRequest` API |
| `save_cookies()` | ✅ Yes | Background | Can use `chrome.cookies` API |
| `load_cookies()` | ✅ Yes | Background | Can use `chrome.cookies` API |
| **DOM Tools** |
| `get_element_by_index(index)` | ✅ Yes | Content | Can use `document.querySelectorAll()` |
| `get_element_by_xpath(xpath)` | ✅ Yes | Content | Can use `document.evaluate()` |
| `get_element_by_css_selector(selector)` | ✅ Yes | Content | Can use `document.querySelector()` |
| `get_element_by_text(text)` | ✅ Yes | Content | Can use `document.querySelector()` with text selectors |
| **Advanced Tools** |
| `drag_drop(source, target)` | ✅ Yes | Content | Can use `MouseEvent` and `DragEvent` |
| `save_pdf()` | ❌ No | N/A | Requires browser-level PDF generation |
| `execute_javascript(script)` | ✅ Yes | Content | Can use `eval()` or `Function()` |
| `extract_content(goal)` | ✅ Yes | Content | Can use DOM APIs and text processing |

### Implementation Notes

1. **Background vs Content Scripts**
   - Background scripts have access to Chrome extension APIs
   - Content scripts have access to the page's DOM
   - Some features require both scripts working together

2. **Permission Requirements**
   - Most features require appropriate permissions in `manifest.json`
   - Some features may require additional host permissions
   - Certain features may require special permissions (e.g., `tabs`, `cookies`)

3. **Limitations**
   - Extensions cannot access certain browser-level features
   - Some features may be restricted by Chrome's security policies
   - Performance may be different from browser automation

4. **Security Considerations**
   - Content scripts run in isolated worlds
   - Cross-origin restrictions apply
   - Some APIs require user consent 