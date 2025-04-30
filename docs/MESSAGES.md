# Message Manager Architecture

## Overview

The Message Manager is a critical component of the Browser-Use agent that handles the communication between the agent and the LLM. It manages message history, token counting, sensitive data filtering, and ensures efficient context window usage. This document explains how the Message Manager works and its key features.

## Core Components

### 1. MessageManager Class
The main `MessageManager` class (`browser_use/agent/message_manager/service.py`) is responsible for:
- Managing message history
- Counting tokens
- Filtering sensitive data
- Handling different message types
- Maintaining conversation context

### 2. Message Types
The Message Manager handles several types of messages:
- System messages (initial setup and instructions)
- Human messages (user input and state information)
- AI messages (model responses)
- Tool messages (action results)

### 3. Message History
The `MessageHistory` class maintains:
- Ordered list of messages with metadata
- Token counting
- Message type tracking
- History management

## Message Flow

### 1. Initialization
```python
message_manager = MessageManager(
    task="Your task description",
    system_message=system_message,
    settings=MessageManagerSettings(
        max_input_tokens=128000,
        include_attributes=['title', 'role', 'aria-label'],
        message_context="Additional context"
    )
)
```

### 2. Message Addition
```python
# Add state message
message_manager.add_state_message(
    state=browser_state,
    result=action_results,
    step_info=step_info,
    use_vision=True
)

# Add model output
message_manager.add_model_output(model_output)

# Add tool message
message_manager.add_tool_message(content="Action completed")
```

### 3. Message Retrieval
```python
# Get all messages
messages = message_manager.get_messages()

# Get total tokens
total_tokens = message_manager.state.history.get_total_tokens()
```

## Key Features

### 1. Token Management
- Tracks token usage for each message
- Enforces maximum token limits
- Handles different token counting for text and images
- Implements token trimming when needed

### 2. Sensitive Data Handling
```python
def _filter_sensitive_data(self, message: BaseMessage) -> BaseMessage:
    """Filter out sensitive data from the message"""
    def replace_sensitive(value: str) -> str:
        for key, val in self.settings.sensitive_data.items():
            if not val:
                continue
            value = value.replace(val, f'<secret>{key}</secret>')
        return value
```
- Replaces sensitive data with placeholders
- Supports different message content types
- Maintains data security
- Preserves message structure

### 3. History Management
```python
def add_message(self, message: BaseMessage, metadata: MessageMetadata, position: int | None = None) -> None:
    """Add message with metadata to history"""
    if position is None:
        self.messages.append(ManagedMessage(message=message, metadata=metadata))
    else:
        self.messages.insert(position, ManagedMessage(message=message, metadata=metadata))
    self.current_tokens += metadata.tokens
```
- Maintains ordered message history
- Tracks message metadata
- Supports message insertion at specific positions
- Manages token counts

### 4. State Messages
```python
def add_state_message(self, state: BrowserState, result: Optional[List[ActionResult]] = None, ...) -> None:
    """Add browser state as human message"""
    state_message = AgentMessagePrompt(
        state,
        result,
        include_attributes=self.settings.include_attributes,
        step_info=step_info,
    ).get_user_message(use_vision)
    self._add_message_with_tokens(state_message)
```
- Formats browser state for LLM consumption
- Includes action results
- Supports vision-based state representation
- Manages step information

## Message Types and Structure

### 1. System Messages
- Initial setup and instructions
- Task description
- Available actions
- Context information

### 2. Human Messages
- Browser state
- Action results
- Error messages
- User input

### 3. AI Messages
- Model responses
- Action decisions
- Reasoning
- Tool calls

### 4. Tool Messages
- Action execution results
- Error information
- State updates
- Completion status

## Error Handling

### 1. Token Limit Errors
```python
def cut_messages(self):
    """Remove oldest messages when token limit is reached"""
    while self.state.history.current_tokens > self.settings.max_input_tokens:
        self.state.history.remove_oldest_message()
```
- Removes oldest messages when token limit is reached
- Preserves system messages
- Maintains conversation coherence
- Logs token usage

### 2. Message Format Errors
- Validates message structure
- Handles missing fields
- Ensures proper serialization
- Maintains message integrity

## Best Practices

1. **Token Management**
   - Monitor token usage
   - Set appropriate max_input_tokens
   - Use efficient message formats
   - Implement proper trimming

2. **Sensitive Data**
   - Always use sensitive_data settings
   - Avoid hardcoding sensitive information
   - Use placeholders consistently
   - Monitor data exposure

3. **Message History**
   - Keep history concise
   - Remove unnecessary messages
   - Maintain important context
   - Track message types

4. **State Messages**
   - Include relevant attributes
   - Format state efficiently
   - Handle vision appropriately
   - Manage step information

## Example Usage

### Basic Setup
```python
from browser_use.agent.message_manager import MessageManager, MessageManagerSettings

# Initialize message manager
message_manager = MessageManager(
    task="Search for information",
    system_message=system_message,
    settings=MessageManagerSettings(
        max_input_tokens=128000,
        include_attributes=['title', 'role'],
        sensitive_data={'api_key': 'actual_key'}
    )
)

# Add state message
message_manager.add_state_message(
    state=browser_state,
    result=action_results,
    use_vision=True
)

# Get messages for LLM
messages = message_manager.get_messages()
```

### Advanced Usage
```python
# Add model output with tool calls
message_manager.add_model_output(model_output)

# Add plan
message_manager.add_plan(plan, position=-1)

# Add tool message
message_manager.add_tool_message(content="Action completed")

# Handle sensitive data
message_manager._filter_sensitive_data(message)
```

## Conclusion

The Message Manager provides a robust framework for handling communication between the agent and the LLM. Its features for token management, sensitive data handling, and history management ensure efficient and secure operation. By understanding its components and workflow, developers can effectively leverage it for various automation tasks. 