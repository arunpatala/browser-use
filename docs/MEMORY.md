# Agent Memory System

## Overview

The Agent Memory System is a sophisticated component that manages procedural memory for agents using the Mem0 framework. It transforms agent interaction history into concise, structured representations at specified intervals, optimizing context window utilization during extended task execution. This document explains how the memory system works and its key features.

## Core Components

### 1. Memory Class
The main `Memory` class (`browser_use/agent/memory/service.py`) is responsible for:
- Managing procedural memory
- Transforming interaction history
- Optimizing context window usage
- Preserving essential operational knowledge

### 2. MemoryConfig
The `MemoryConfig` class (`browser_use/agent/memory/views.py`) handles:
- Memory settings (interval, agent ID)
- Embedder configuration
- LLM settings
- Vector store settings

## Memory System Architecture

### 1. Initialization
```python
memory = Memory(
    message_manager=message_manager,
    llm=llm,
    config=MemoryConfig(
        agent_id="custom_agent",
        memory_interval=10,
        embedder_provider="openai",
        embedder_model="text-embedding-3-small"
    )
)
```

### 2. Configuration Options
```python
class MemoryConfig:
    # Memory settings
    agent_id: str = "browser_use_agent"
    memory_interval: int = 10  # Create memory every 10 steps
    
    # Embedder settings
    embedder_provider: Literal['openai', 'gemini', 'ollama', 'huggingface'] = 'huggingface'
    embedder_model: str = "all-MiniLM-L6-v2"
    embedder_dims: int = 384
    
    # Vector store settings
    vector_store_provider: Literal['faiss'] = 'faiss'
    vector_store_base_path: str = "/tmp/mem0"
```

## Memory Creation Process

### 1. Procedural Memory Creation
```python
def create_procedural_memory(self, current_step: int) -> None:
    """Create a procedural memory if needed based on the current step."""
    # Get all messages
    all_messages = self.message_manager.state.history.messages
    
    # Separate messages into those to keep and those to process
    new_messages = []
    messages_to_process = []
    
    for msg in all_messages:
        if isinstance(msg, ManagedMessage) and msg.metadata.message_type in {'init', 'memory'}:
            new_messages.append(msg)
        else:
            if len(msg.message.content) > 0:
                messages_to_process.append(msg)
    
    # Create memory if enough messages
    if len(messages_to_process) > 1:
        memory_content = self._create([m.message for m in messages_to_process], current_step)
        
        # Replace processed messages with consolidated memory
        if memory_content:
            memory_message = HumanMessage(content=memory_content)
            memory_metadata = MessageMetadata(tokens=memory_tokens, message_type='memory')
            new_messages.append(ManagedMessage(message=memory_message, metadata=memory_metadata))
            
            # Update history
            self.message_manager.state.history.messages = new_messages
```

### 2. Memory Storage
```python
def _create(self, messages: List[BaseMessage], current_step: int) -> Optional[str]:
    """Create a memory entry using Mem0."""
    parsed_messages = convert_to_openai_messages(messages)
    try:
        results = self.mem0.add(
            messages=parsed_messages,
            agent_id=self.config.agent_id,
            memory_type='procedural_memory',
            metadata={'step': current_step},
        )
        if len(results.get('results', [])):
            return results.get('results', [])[0].get('memory')
        return None
    except Exception as e:
        logger.error(f'Error creating procedural memory: {e}')
        return None
```

## Supported Embedders

### 1. OpenAI Embedder
- Model: text-embedding-3-small
- Dimensions: 1536
- Provider: openai

### 2. Google Gemini Embedder
- Model: models/text-embedding-004
- Dimensions: 768
- Provider: gemini

### 3. Ollama Embedder
- Model: nomic-embed-text
- Dimensions: 512
- Provider: ollama

### 4. HuggingFace Embedder
- Model: all-MiniLM-L6-v2 (default)
- Dimensions: 384
- Provider: huggingface

## Vector Store Configuration

### 1. FAISS Vector Store
```python
vector_store_config = {
    'provider': 'faiss',
    'config': {
        'embedding_model_dims': embedder_dims,
        'path': vector_store_path
    }
}
```

### 2. Storage Path
- Base path: /tmp/mem0
- Full path: /tmp/mem0_{embedder_dims}_faiss

## Memory Management Features

### 1. Token Optimization
- Consolidates multiple messages into single memory entries
- Reduces token usage while preserving essential information
- Maintains conversation coherence

### 2. Message Filtering
- Preserves system and memory messages
- Processes only relevant interaction messages
- Maintains message type tracking

### 3. Error Handling
- Graceful handling of memory creation failures
- Logging of memory-related errors
- Fallback to original messages if needed

## Embeddings and Context Management

### 1. Embedding Process
```python
# How messages are embedded and stored
def _create(self, messages: List[BaseMessage], current_step: int) -> Optional[str]:
    # Convert messages to OpenAI format
    parsed_messages = convert_to_openai_messages(messages)
    
    # Create embeddings and store in vector database
    results = self.mem0.add(
        messages=parsed_messages,
        agent_id=self.config.agent_id,
        memory_type='procedural_memory',
        metadata={'step': current_step},
    )
```

### 2. Context Retrieval
```python
# How relevant context is retrieved
def retrieve_context(self, query: str, top_k: int = 3) -> List[str]:
    """Retrieve relevant context from memory."""
    try:
        results = self.mem0.search(
            query=query,
            agent_id=self.config.agent_id,
            memory_type='procedural_memory',
            top_k=top_k
        )
        return [r.get('memory') for r in results.get('results', [])]
    except Exception as e:
        logger.error(f'Error retrieving context: {e}')
        return []
```

### 3. Context Integration
```python
# How context is integrated into messages
def add_context_to_messages(self, messages: List[BaseMessage]) -> List[BaseMessage]:
    """Add relevant context to messages."""
    # Get the last message as query
    last_message = messages[-1].content if messages else ""
    
    # Retrieve relevant context
    context = self.retrieve_context(last_message)
    
    if context:
        # Create system message with context
        context_message = SystemMessage(
            content=f"Relevant context from memory:\n" + "\n".join(context)
        )
        # Add context message before the last message
        messages.insert(-1, context_message)
    
    return messages
```

### 4. Embedding Models and Dimensions

#### OpenAI Embeddings
```python
# OpenAI embedding configuration
embedder_config = {
    'provider': 'openai',
    'model': 'text-embedding-3-small',
    'dimensions': 1536,
    'api_key': 'your-api-key'
}
```

#### HuggingFace Embeddings
```python
# HuggingFace embedding configuration
embedder_config = {
    'provider': 'huggingface',
    'model': 'all-MiniLM-L6-v2',
    'dimensions': 384,
    'device': 'cpu'  # or 'cuda' for GPU
}
```

### 5. Vector Store Operations

#### Adding to Vector Store
```python
# How vectors are stored
vector_store.add(
    vectors=embeddings,
    metadata={
        'step': current_step,
        'message_type': 'procedural_memory',
        'timestamp': datetime.now().isoformat()
    }
)
```

#### Searching Vector Store
```python
# How vectors are searched
results = vector_store.search(
    query_vector=query_embedding,
    top_k=3,
    filter={
        'message_type': 'procedural_memory',
        'step': {'$gte': current_step - 10}  # Only recent memories
    }
)
```

### 6. Memory Consolidation

#### Message Consolidation
```python
# How messages are consolidated into memory
def consolidate_messages(self, messages: List[BaseMessage]) -> str:
    """Consolidate multiple messages into a single memory entry."""
    consolidated = []
    current_context = []
    
    for msg in messages:
        if isinstance(msg, SystemMessage):
            current_context.append(msg.content)
        elif isinstance(msg, HumanMessage):
            if current_context:
                consolidated.append(f"Context: {' '.join(current_context)}")
                current_context = []
            consolidated.append(f"User: {msg.content}")
        elif isinstance(msg, AIMessage):
            consolidated.append(f"Assistant: {msg.content}")
    
    return "\n".join(consolidated)
```

#### Memory Optimization
```python
# How memories are optimized for storage
def optimize_memory(self, memory_content: str) -> str:
    """Optimize memory content for storage."""
    # Remove redundant information
    content = re.sub(r'\s+', ' ', memory_content)
    
    # Truncate if too long
    if len(content) > 1000:
        content = content[:997] + "..."
    
    return content
```

### 7. Memory Retrieval Strategies

#### Semantic Search
```python
# Semantic search for relevant memories
def semantic_search(self, query: str) -> List[Dict]:
    """Perform semantic search in memory."""
    query_embedding = self.embedder.embed_query(query)
    results = self.vector_store.similarity_search(
        query_embedding,
        k=3,
        filter={'type': 'procedural_memory'}
    )
    return results
```

#### Temporal Search
```python
# Search memories by time
def temporal_search(self, start_time: datetime, end_time: datetime) -> List[Dict]:
    """Search memories within a time range."""
    results = self.vector_store.search(
        filter={
            'timestamp': {
                '$gte': start_time.isoformat(),
                '$lte': end_time.isoformat()
            }
        }
    )
    return results
```

## Memory Compression and Summarization

### 1. Message Compression
```python
# How messages are compressed into memory
def create_procedural_memory(self, current_step: int) -> None:
    """Create a procedural memory by compressing multiple messages."""
    # Get all messages
    all_messages = self.message_manager.state.history.messages
    
    # Separate messages into those to keep and those to process
    new_messages = []
    messages_to_process = []
    
    for msg in all_messages:
        if isinstance(msg, ManagedMessage) and msg.metadata.message_type in {'init', 'memory'}:
            new_messages.append(msg)
        else:
            if len(msg.message.content) > 0:
                messages_to_process.append(msg)
    
    # Create memory if enough messages
    if len(messages_to_process) > 1:
        # Compress multiple messages into a single memory entry
        memory_content = self._create([m.message for m in messages_to_process], current_step)
        
        # Replace processed messages with consolidated memory
        if memory_content:
            memory_message = HumanMessage(content=memory_content)
            memory_metadata = MessageMetadata(tokens=memory_tokens, message_type='memory')
            new_messages.append(ManagedMessage(message=memory_message, metadata=memory_metadata))
            
            # Update history with compressed memory
            self.message_manager.state.history.messages = new_messages
```

### 2. Compression Example
```python
# Before compression:
messages = [
    "User: I want to search for a product",
    "Assistant: What type of product are you looking for?",
    "User: I'm looking for a laptop",
    "Assistant: What's your budget range?",
    "User: Around $1000"
]

# After compression:
compressed_memory = """
User initiated a product search for a laptop with a budget of around $1000.
The assistant asked about the product type and budget range.
"""
```

### 3. Compression Benefits
1. **Token Efficiency**: Reduces the number of tokens needed to maintain context
2. **Context Preservation**: Maintains essential information while removing redundancy
3. **History Management**: Prevents the conversation history from growing too large
4. **Focus on Important Information**: Emphasizes key details and decisions

### 4. Compression Process
```python
# How messages are compressed
def compress_messages(self, messages: List[BaseMessage]) -> str:
    """Compress multiple messages into a concise summary."""
    # Extract key information
    key_points = []
    current_context = []
    
    for msg in messages:
        if isinstance(msg, SystemMessage):
            current_context.append(msg.content)
        elif isinstance(msg, HumanMessage):
            if current_context:
                key_points.append(f"Context: {' '.join(current_context)}")
                current_context = []
            key_points.append(f"User action: {msg.content}")
        elif isinstance(msg, AIMessage):
            key_points.append(f"System response: {msg.content}")
    
    # Create concise summary
    summary = "\n".join(key_points)
    
    # Optimize for storage
    if len(summary) > 1000:
        summary = summary[:997] + "..."
    
    return summary
```

### 5. Memory Types
```python
# Different types of compressed memories
memory_types = {
    'procedural_memory': "Summarizes steps and actions taken",
    'context_memory': "Maintains important context and state",
    'decision_memory': "Records key decisions and their rationale"
}
```

### 6. Compression Strategy
```python
# How compression is applied
def apply_compression_strategy(self, messages: List[BaseMessage]) -> str:
    """Apply appropriate compression strategy based on message content."""
    # Count messages
    message_count = len(messages)
    
    if message_count <= 2:
        # No compression needed for few messages
        return self.consolidate_messages(messages)
    elif message_count <= 5:
        # Light compression for moderate message count
        return self.compress_messages(messages)
    else:
        # Heavy compression for many messages
        return self.summarize_messages(messages)
```

## Best Practices

1. **Memory Interval**
   - Set appropriate memory_interval based on task complexity
   - Balance between memory creation frequency and performance
   - Consider token usage patterns

2. **Embedder Selection**
   - Choose embedder based on available resources
   - Consider embedding dimensions and performance
   - Match embedder with LLM capabilities

3. **Vector Store Management**
   - Monitor vector store size
   - Implement cleanup strategies
   - Handle storage path permissions

4. **Error Handling**
   - Implement proper error logging
   - Handle memory creation failures
   - Maintain message history integrity

## Example Usage

### Basic Setup
```python
from browser_use.agent.memory import Memory, MemoryConfig
from browser_use.agent.message_manager import MessageManager

# Initialize memory system
memory_config = MemoryConfig(
    agent_id="search_agent",
    memory_interval=5,
    embedder_provider="openai"
)

memory = Memory(
    message_manager=message_manager,
    llm=llm,
    config=memory_config
)

# Create memory at specific step
memory.create_procedural_memory(current_step=10)
```

### Advanced Configuration
```python
# Custom embedder configuration
memory_config = MemoryConfig(
    agent_id="custom_agent",
    memory_interval=15,
    embedder_provider="huggingface",
    embedder_model="all-mpnet-base-v2",
    embedder_dims=768,
    vector_store_base_path="/custom/path/mem0"
)

# Initialize with custom config
memory = Memory(
    message_manager=message_manager,
    llm=llm,
    config=memory_config
)
```

## Use Cases

### 1. Long-Running Web Automation Tasks
```python
# Example: Automated form filling across multiple pages
memory_config = MemoryConfig(
    agent_id="form_filler",
    memory_interval=5,  # Create memory every 5 steps
    embedder_provider="openai"  # Use OpenAI for high-quality embeddings
)

# The memory system will:
# - Remember form field values across page navigations
# - Track progress through multi-step forms
# - Maintain context about previously filled information
```

### 2. Web Scraping with State Management
```python
# Example: Scraping product information across multiple pages
memory_config = MemoryConfig(
    agent_id="product_scraper",
    memory_interval=10,
    embedder_provider="huggingface"  # Use HuggingFace for cost-effective embeddings
)

# The memory system will:
# - Remember product categories and navigation paths
# - Track already scraped products to avoid duplicates
# - Maintain context about filtering and sorting preferences
```

### 3. Interactive Web Testing
```python
# Example: Automated UI testing with state tracking
memory_config = MemoryConfig(
    agent_id="ui_tester",
    memory_interval=3,  # More frequent memory creation for precise state tracking
    embedder_provider="gemini"  # Use Gemini for balanced performance
)

# The memory system will:
# - Track UI state changes and interactions
# - Remember test steps and their outcomes
# - Maintain context about test scenarios and expected results
```

### 4. Multi-Session Browser Automation
```python
# Example: Maintaining state across browser sessions
memory_config = MemoryConfig(
    agent_id="session_manager",
    memory_interval=15,
    vector_store_base_path="/custom/path/session_memories"  # Custom storage location
)

# The memory system will:
# - Preserve session-specific information
# - Remember user preferences and settings
# - Track progress across multiple automation sessions
```

### 5. Complex Web Workflows
```python
# Example: Managing complex multi-step workflows
memory_config = MemoryConfig(
    agent_id="workflow_manager",
    memory_interval=8,
    embedder_provider="ollama",  # Use Ollama for local processing
    embedder_model="nomic-embed-text"  # Custom embedding model
)

# The memory system will:
# - Track workflow progress and state
# - Remember decision points and their outcomes
# - Maintain context about workflow dependencies
```

### 6. Data Collection and Analysis
```python
# Example: Automated data gathering and processing
memory_config = MemoryConfig(
    agent_id="data_collector",
    memory_interval=20,
    embedder_provider="huggingface",
    embedder_model="all-mpnet-base-v2"  # High-performance model for data processing
)

# The memory system will:
# - Remember data sources and collection patterns
# - Track data processing steps and transformations
# - Maintain context about data relationships and dependencies
```

## Memory Scope and Persistence

### 1. Session vs. Persistent Memory
```python
# Memory configuration for different scopes
class MemoryConfig:
    # Session-specific memory (default)
    session_memory = MemoryConfig(
        agent_id="session_agent",
        memory_interval=10,
        vector_store_base_path="/tmp/mem0/session"  # Session-specific storage
    )
    
    # Persistent memory across sessions
    persistent_memory = MemoryConfig(
        agent_id="persistent_agent",
        memory_interval=10,
        vector_store_base_path="/custom/path/persistent_mem0"  # Persistent storage
    )
```

### 2. How It Differs from RAG
```python
# Traditional RAG
def traditional_rag(query: str):
    # 1. Query external knowledge base
    # 2. Retrieve relevant documents
    # 3. Augment generation with retrieved context
    return augmented_response

# Agent Memory System
def agent_memory(query: str):
    # 1. Compress current session history
    # 2. Store in session-specific memory
    # 3. Retrieve relevant session context
    # 4. Augment generation with session context
    return augmented_response
```

### 3. Memory Lifecycle
```python
# Memory lifecycle in a session
def memory_lifecycle(self):
    # 1. Session Start
    self.memory = Memory(
        message_manager=message_manager,
        llm=llm,
        config=MemoryConfig(
            agent_id=f"session_{uuid.uuid4()}",  # Unique session ID
            memory_interval=10
        )
    )
    
    # 2. During Session
    # - Messages are compressed and stored
    # - Context is retrieved when needed
    # - Memory is maintained in session scope
    
    # 3. Session End
    # - Session memory is cleared by default
    # - Can be configured to persist if needed
```

### 4. Persistence Options
```python
# How to enable persistence across sessions
def enable_persistence(self):
    # 1. Configure persistent storage
    memory_config = MemoryConfig(
        agent_id="persistent_agent",
        memory_interval=10,
        vector_store_base_path="/custom/path/persistent_mem0"
    )
    
    # 2. Initialize memory with persistence
    self.memory = Memory(
        message_manager=message_manager,
        llm=llm,
        config=memory_config
    )
    
    # 3. Memory will now persist across sessions
```

### 5. Key Differences from RAG
1. **Scope**:
   - RAG: Typically works with external knowledge bases
   - Agent Memory: Primarily works with session-specific history

2. **Persistence**:
   - RAG: Usually persistent across all uses
   - Agent Memory: Session-based by default, can be configured for persistence

3. **Content**:
   - RAG: Works with static knowledge documents
   - Agent Memory: Works with dynamic conversation history

4. **Usage**:
   - RAG: Augments generation with external knowledge
   - Agent Memory: Maintains context within a session

### 6. When to Use Each
```python
# Traditional RAG Use Case
def use_rag():
    # When you need:
    # - Access to external knowledge
    # - Persistent information across all sessions
    # - Static document retrieval
    pass

# Agent Memory Use Case
def use_agent_memory():
    # When you need:
    # - Session-specific context
    # - Dynamic conversation history
    # - Temporary memory for current task
    pass
```

## Conclusion

The Agent Memory System provides a robust framework for managing procedural memory in agent interactions. Its features for memory creation, token optimization, and vector storage ensure efficient operation during extended tasks. By understanding its components and configuration options, developers can effectively leverage it for various automation scenarios. 