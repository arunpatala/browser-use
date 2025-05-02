# Browser-Use Extension: Modular Architecture V1

## Overview

This document outlines a highly modular architecture for the Browser-Use Chrome Extension, designed to maximize maintainability, testability, and extensibility. The architecture separates concerns into distinct modules with clear boundaries and responsibilities.

## Module Structure

### 1. Core Agent Module
Core orchestration layer responsible for task planning and execution.

```
agent/
  - planner.ts          # Task planning and decomposition
  - executor.ts         # Task execution orchestration
  - memory/
    - short_term.ts     # Working memory for current task
    - long_term.ts      # Persistent memory across sessions
  - types/
    - task.ts          # Task and subtask interfaces
    - plan.ts          # Planning interfaces
```

**Key Responsibilities:**
- Task decomposition and planning
- Orchestrating task execution
- Managing task state and memory
- Coordinating between other modules

**Example Interface:**
```typescript
interface Agent {
  plan(task: Task): Promise<Plan>;
  execute(plan: Plan): Promise<Result>;
  remember(memory: Memory): void;
  recall(query: Query): Memory[];
}
```

### 2. LLM Module
Handles all AI/Language Model interactions.

```
llm/
  - client/
    - base.ts          # Abstract LLM client interface
    - openai.ts        # OpenAI implementation
    - anthropic.ts     # Anthropic implementation
  - prompt/
    - templates.ts     # Reusable prompt templates
    - builder.ts       # Prompt construction utilities
  - types/
    - response.ts      # LLM response interfaces
```

**Key Responsibilities:**
- Managing API connections to LLM providers
- Prompt template management
- Response parsing and error handling
- Rate limiting and retry logic

**Example Interface:**
```typescript
interface LLMClient {
  complete(prompt: string): Promise<LLMResponse>;
  stream(prompt: string): AsyncIterator<LLMResponse>;
  embedText(text: string): Promise<number[]>;
}
```

### 3. Storage Module
Handles all data persistence needs.

```
storage/
  - adapters/
    - base.ts          # Abstract storage interface
    - chrome.ts        # Chrome storage implementation
    - indexdb.ts       # IndexedDB implementation
  - models/
    - task.ts          # Task storage models
    - history.ts       # History storage models
  - migrations/        # Storage schema migrations
```

**Key Responsibilities:**
- Data persistence across sessions
- Schema management and migrations
- Caching and optimization
- Data encryption (when needed)

**Example Interface:**
```typescript
interface StorageAdapter {
  get<T>(key: string): Promise<T>;
  set<T>(key: string, value: T): Promise<void>;
  delete(key: string): Promise<void>;
  clear(): Promise<void>;
}
```

### 4. Browser Interaction Module
Manages all browser-related operations.

```
browser/
  - dom/
    - interactor.ts    # DOM manipulation
    - observer.ts      # DOM change detection
    - selector.ts      # Element selection strategies
  - tools/
    - navigation.ts    # Page navigation
    - input.ts         # Form interactions
    - extraction.ts    # Data extraction
  - context/
    - capture.ts       # Page context capture
    - analyzer.ts      # Context analysis
  - types/
    - action.ts        # Browser action interfaces
```

**Key Responsibilities:**
- DOM manipulation and observation
- Page navigation and interaction
- Data extraction and injection
- Context analysis and capture

**Example Interface:**
```typescript
interface BrowserModule {
  navigate(url: string): Promise<void>;
  click(selector: string): Promise<void>;
  extract(selector: string): Promise<string>;
  observe(selector: string, callback: Function): void;
}
```

### 5. Extension Module
Chrome-specific implementation.

```
extension/
  - background/
    - service.ts       # Service worker
    - router.ts        # Message routing
  - content/
    - script.ts        # Content script
    - bridge.ts        # DOM-Extension bridge
  - popup/
    - ui/             # UI components
    - state/          # UI state management
  - messaging/
    - protocol.ts     # Messaging protocol
    - handlers.ts     # Message handlers
```

**Key Responsibilities:**
- Chrome extension lifecycle management
- Message routing between components
- UI management
- Extension-specific configuration

### 6. Task Module
Manages task workflows and execution.

```
tasks/
  - registry/
    - actions.ts       # Available actions
    - validators.ts    # Action validation
  - workflow/
    - builder.ts       # Workflow construction
    - executor.ts      # Workflow execution
  - types/
    - workflow.ts      # Workflow interfaces
```

**Key Responsibilities:**
- Action registry management
- Workflow construction and validation
- Task execution and monitoring
- Error handling and recovery

### 7. Core Module
Shared utilities and types.

```
core/
  - utils/
    - logger.ts
    - error.ts
    - validation.ts
  - types/
    - common.ts
  - config/
    - settings.ts
```

**Key Responsibilities:**
- Shared utility functions
- Common type definitions
- Configuration management
- Error handling utilities

## Implementation Guidelines

### 1. Dependency Management
```typescript
// Use dependency injection
class AgentImpl implements Agent {
  constructor(
    private llm: LLMClient,
    private storage: StorageAdapter,
    private browser: BrowserModule
  ) {}
}
```

### 2. Communication Patterns
```typescript
// Event-based communication
interface EventBus {
  emit(event: string, data: any): void;
  on(event: string, handler: Function): void;
}
```

### 3. Error Handling
```typescript
// Centralized error handling
class ErrorHandler {
  handle(error: Error): void {
    logger.error(error);
    metrics.recordError(error);
    ui.showError(error);
  }
}
```

### 4. Testing Strategy

```
tests/
  - unit/             # Unit tests per module
  - integration/      # Cross-module integration
  - e2e/             # End-to-end workflows
  - mocks/           # Mock implementations
```

Example test:
```typescript
describe('Agent', () => {
  it('should plan task execution', async () => {
    const agent = new AgentImpl(mockLLM, mockStorage, mockBrowser);
    const plan = await agent.plan(task);
    expect(plan.steps).toHaveLength(3);
  });
});
```

## Module Dependencies

```mermaid
graph TD
    Extension --> Agent
    Agent --> Tasks
    Agent --> LLM
    Agent --> Storage
    Tasks --> Browser
    Tasks --> Storage
    Browser --> Storage
    Core --> All[All Modules]
```

## Configuration Management

```typescript
// Centralized configuration
interface Config {
  llm: {
    provider: string;
    apiKey: string;
    maxRetries: number;
  };
  storage: {
    encryption: boolean;
    maxSize: number;
  };
  browser: {
    timeout: number;
    retryInterval: number;
  };
}
```

## Security Considerations

1. **Data Security**
   - Encrypt sensitive data in storage
   - Sanitize all inputs
   - Validate all actions before execution

2. **Permission Management**
   - Minimal required permissions
   - Clear user consent flows
   - Secure message passing

3. **Error Handling**
   - Graceful degradation
   - User-friendly error messages
   - Detailed logging for debugging

## Future Extensions

1. **Plugin System**
   ```typescript
   interface Plugin {
     name: string;
     initialize(): Promise<void>;
     getActions(): Action[];
   }
   ```

2. **Custom Action Support**
   ```typescript
   interface CustomAction {
     name: string;
     validate(params: any): boolean;
     execute(params: any): Promise<any>;
   }
   ```

3. **Multi-Provider Support**
   - Multiple LLM providers
   - Different storage backends
   - Cross-browser compatibility

## Version Control and Release Strategy

1. **Semantic Versioning**
   - Major: Breaking changes
   - Minor: New features
   - Patch: Bug fixes

2. **Release Process**
   - Feature branches
   - PR reviews
   - Automated testing
   - Staged rollouts

## Metrics and Monitoring

1. **Performance Metrics**
   - Action execution time
   - Memory usage
   - API latency

2. **Error Tracking**
   - Error rates
   - Stack traces
   - User impact

3. **Usage Analytics**
   - Feature usage
   - User patterns
   - Success rates

## Getting Started

1. **Development Setup**
   ```bash
   npm install
   npm run dev
   ```

2. **Building**
   ```bash
   npm run build
   ```

3. **Testing**
   ```bash
   npm run test
   npm run test:e2e
   ```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request
4. Ensure tests pass
5. Update documentation

---

This modular architecture provides a solid foundation for building a robust, maintainable, and extensible browser automation system. Each module has clear responsibilities and boundaries, making it easier to develop, test, and maintain the codebase. 



# Browser-Use Packages

This directory contains independently developable modules for the Browser-Use extension.

## Package Structure

```
packages/
  ├── agent/           # Core agent functionality
  ├── llm/            # Language model integration
  ├── browser/        # Browser interaction
  ├── storage/        # Data persistence
  ├── tasks/          # Task management
  ├── core/           # Shared utilities
  └── extension/      # Chrome extension specific
```

## Development Workflow

Each package can be developed and tested independently:

1. Each package has its own:
   - `package.json`
   - `tsconfig.json`
   - `jest.config.js`
   - `README.md`

2. Shared dependencies are hoisted to the root

3. Inter-package dependencies are managed through workspace references

## Getting Started

To work on a specific package:

```bash
cd packages/<package-name>
npm install
npm run dev
npm test
```

## Package Dependencies

- `core`: No dependencies
- `llm`: Depends on `core`
- `storage`: Depends on `core`
- `browser`: Depends on `core`
- `tasks`: Depends on `core`, `browser`
- `agent`: Depends on `core`, `llm`, `tasks`
- `extension`: Depends on all packages

# Workflow Diagrams

## 1. Task Execution Workflow
```mermaid
sequenceDiagram
    participant User
    participant Extension
    participant Agent
    participant LLM
    participant Tasks
    participant Browser
    participant Storage

    User->>Extension: Initiate Task
    Extension->>Agent: Create Task Request
    Agent->>LLM: Generate Task Plan
    LLM-->>Agent: Return Plan Steps
    
    loop For Each Step
        Agent->>Tasks: Execute Step
        Tasks->>Browser: Perform Browser Action
        Browser-->>Tasks: Action Result
        Tasks->>Storage: Save Progress
        Tasks-->>Agent: Step Complete
    end
    
    Agent-->>Extension: Task Complete
    Extension-->>User: Show Results
```

## 2. Browser Interaction Flow
```mermaid
sequenceDiagram
    participant Tasks
    participant Browser
    participant DOM
    participant ContentScript
    participant Background
    
    Tasks->>Browser: Request Action
    Browser->>ContentScript: Send Action Command
    ContentScript->>DOM: Execute DOM Operation
    DOM-->>ContentScript: Operation Result
    ContentScript->>Background: Report Status
    Background-->>Browser: Update State
    Browser-->>Tasks: Action Complete
```

## 3. Memory Management Flow
```mermaid
sequenceDiagram
    participant Agent
    participant ShortTerm
    participant LongTerm
    participant Storage
    participant LLM
    
    Agent->>ShortTerm: Store Current Context
    Agent->>LLM: Process Task
    LLM->>ShortTerm: Query Context
    ShortTerm-->>LLM: Return Context
    
    opt Important Information
        Agent->>LongTerm: Store for Future
        LongTerm->>Storage: Persist Data
    end
```

## 4. Plugin System Architecture
```mermaid
graph TD
    A[Plugin Manager] --> B[Core Plugin API]
    B --> C[Action Registry]
    B --> D[Event System]
    
    C --> E[Custom Actions]
    C --> F[Built-in Actions]
    
    D --> G[Event Handlers]
    D --> H[Event Emitters]
    
    I[External Plugin] --> A
    J[Custom Plugin] --> A
```

## 5. Error Handling Flow
```mermaid
sequenceDiagram
    participant Component
    participant ErrorHandler
    participant Logger
    participant UI
    participant Recovery
    
    Component->>ErrorHandler: Error Occurs
    ErrorHandler->>Logger: Log Error
    ErrorHandler->>UI: Show User Message
    
    alt Recoverable Error
        ErrorHandler->>Recovery: Attempt Recovery
        Recovery-->>Component: Resume Operation
    else Fatal Error
        ErrorHandler->>UI: Show Error Details
        UI-->>Component: Reset State
    end
```

## 6. Data Flow Between Packages
```mermaid
graph TD
    Core[Core Package] --> |Types & Utils| All[All Packages]
    
    LLM[LLM Package] --> |AI Processing| Agent[Agent Package]
    Storage[Storage Package] --> |Data Persistence| Agent
    
    Browser[Browser Package] --> |DOM Operations| Tasks[Tasks Package]
    Tasks --> |Execution| Agent
    
    Agent --> |Orchestration| Extension[Extension Package]
    Extension --> |User Interface| User[User Interface]
```

These workflow diagrams illustrate:
1. How tasks flow from user input through the system
2. How browser interactions are managed
3. How memory and context are handled
4. How plugins integrate with the core system
5. How errors are handled and recovered from
6. How data flows between different packages

Each workflow represents a key aspect of the system's operation and shows the relationships between different components. This should help in understanding how the various parts of the system interact and depend on each other.

# Real-World Use Case Examples

## 1. Use Case: Buy iPhone from Amazon
```mermaid
sequenceDiagram
    participant User
    participant Extension
    participant Agent
    participant LLM
    participant Tasks
    participant Browser
    participant Storage

    User->>Extension: "Buy iPhone 15 Pro from Amazon"
    Extension->>Agent: Create Shopping Task
    
    Agent->>LLM: Generate Shopping Plan
    LLM-->>Agent: Return Steps (Search, Compare, Purchase)
    
    %% Search Phase
    Agent->>Tasks: Execute Search
    Tasks->>Browser: Navigate to amazon.com
    Browser-->>Tasks: Page Loaded
    Tasks->>Browser: Search "iPhone 15 Pro"
    Browser-->>Tasks: Search Results
    Tasks->>Storage: Save Product Options
    
    %% Product Analysis
    Agent->>LLM: Analyze Product Options
    LLM-->>Agent: Best Match Found
    
    %% Price Check
    Agent->>Tasks: Check Price & Availability
    Tasks->>Browser: Extract Price Info
    Browser-->>Tasks: Price Data
    Tasks->>Storage: Save Price Info
    
    %% User Confirmation
    Agent-->>Extension: Show Best Option
    Extension-->>User: Request Purchase Approval
    User->>Extension: Confirm Purchase
    
    %% Purchase Process
    Agent->>Tasks: Execute Purchase
    Tasks->>Browser: Add to Cart
    Browser-->>Tasks: Cart Updated
    Tasks->>Browser: Navigate to Checkout
    Browser-->>Tasks: Checkout Page
    
    %% Payment Process
    Tasks->>Storage: Get Saved Payment Info
    Storage-->>Tasks: Payment Details
    Tasks->>Browser: Fill Payment Form
    Browser-->>Tasks: Form Filled
    
    %% Confirmation
    Tasks->>Browser: Complete Purchase
    Browser-->>Tasks: Order Confirmation
    Tasks->>Storage: Save Order Details
    Tasks-->>Agent: Purchase Complete
    Agent-->>Extension: Show Success
    Extension-->>User: Display Order Summary
```

## 2. Task Breakdown for iPhone Purchase

```mermaid
graph TD
    A[User Request] --> B[Task Planning]
    
    B --> C1[Search Phase]
    B --> C2[Analysis Phase]
    B --> C3[Purchase Phase]
    
    %% Search Phase Details
    C1 --> D1[Navigate to Amazon]
    D1 --> D2[Search iPhone 15 Pro]
    D2 --> D3[Extract Results]
    
    %% Analysis Phase Details
    C2 --> E1[Compare Models]
    E1 --> E2[Check Reviews]
    E2 --> E3[Verify Price]
    E3 --> E4[Check Availability]
    
    %% Purchase Phase Details
    C3 --> F1[Add to Cart]
    F1 --> F2[Begin Checkout]
    F2 --> F3[Fill Payment Info]
    F3 --> F4[Confirm Order]
    
    %% Status Updates
    D3 --> G[Progress Storage]
    E4 --> G
    F4 --> G
    
    G --> H[User Updates]
```

## 3. Component Interaction Details

### Search Phase
```typescript
// Agent planning the search
interface SearchPlan {
    searchTerms: string[];
    priceRange: PriceRange;
    filters: ProductFilters;
}

// Browser interaction
interface BrowserActions {
    async navigateToAmazon(): Promise<void>;
    async searchProduct(terms: string): Promise<SearchResults>;
    async extractProductData(selector: string): Promise<ProductData>;
}

// Storage operations
interface StorageOperations {
    saveSearchResults(results: SearchResults): Promise<void>;
    saveProductOptions(options: ProductOption[]): Promise<void>;
}
```

### Analysis Phase
```typescript
// LLM product analysis
interface ProductAnalysis {
    compareOptions(products: ProductOption[]): Promise<BestMatch>;
    validatePrice(price: number, budget: number): boolean;
    analyzeReviews(reviews: Review[]): SentimentScore;
}

// Decision making
interface PurchaseDecision {
    productMatch: number;
    priceMatch: number;
    availabilityStatus: boolean;
    recommendedAction: Action;
}
```

### Purchase Phase
```typescript
// Purchase execution
interface PurchaseExecution {
    async addToCart(productId: string): Promise<CartStatus>;
    async navigateToCheckout(): Promise<CheckoutPage>;
    async fillPaymentDetails(payment: PaymentInfo): Promise<void>;
    async confirmPurchase(): Promise<OrderConfirmation>;
}

// Order tracking
interface OrderTracking {
    orderId: string;
    status: OrderStatus;
    confirmationDetails: ConfirmationDetails;
    saveOrderHistory(): Promise<void>;
}
```

This real-world example demonstrates:
1. How user intent is broken down into actionable steps
2. How different components coordinate for a complex task
3. How the system handles user interaction points
4. How data is persisted throughout the process
5. How error cases and validations are managed
6. The actual interfaces used by each component

The workflow shows both the high-level sequence and the detailed component interactions needed to complete a real e-commerce transaction.

## 4. Use Case: Book a Flight
```mermaid
sequenceDiagram
    participant User
    participant Extension
    participant Agent
    participant LLM
    participant Tasks
    participant Browser
    participant Storage

    User->>Extension: "Book flight from NYC to SF next weekend"
    Extension->>Agent: Create Travel Task
    
    Agent->>LLM: Generate Travel Plan
    LLM-->>Agent: Return Steps (Search, Compare, Book)
    
    %% Initial Search
    Agent->>Tasks: Execute Flight Search
    Tasks->>Browser: Navigate to Multiple Airlines
    Browser-->>Tasks: Pages Loaded
    
    par Search Multiple Sites
        Tasks->>Browser: Search on Kayak
        Tasks->>Browser: Search on Expedia
        Tasks->>Browser: Search on Airline Sites
    end
    
    %% Collect Results
    Browser-->>Tasks: All Search Results
    Tasks->>Storage: Save Flight Options
    
    %% Analysis
    Agent->>LLM: Analyze Flight Options
    Note over LLM: Consider price, duration, layovers
    LLM-->>Agent: Best Options Selected
    
    %% User Preference Check
    Agent-->>Extension: Show Top 3 Options
    Extension-->>User: Request Flight Selection
    User->>Extension: Select Preferred Flight
    
    %% Booking Process
    Agent->>Tasks: Begin Booking
    Tasks->>Browser: Navigate to Booking Page
    Browser-->>Tasks: Booking Form Loaded
    
    %% Fill Details
    Tasks->>Storage: Get Traveler Info
    Storage-->>Tasks: Traveler Details
    Tasks->>Browser: Fill Passenger Info
    Tasks->>Browser: Select Seats
    Browser-->>Tasks: Seat Map
    
    %% Payment
    Tasks->>Storage: Get Payment Info
    Storage-->>Tasks: Payment Details
    Tasks->>Browser: Complete Payment
    Browser-->>Tasks: Booking Confirmation
    
    %% Confirmation
    Tasks->>Storage: Save Booking Details
    Tasks-->>Agent: Booking Complete
    Agent-->>Extension: Show Itinerary
    Extension-->>User: Display Booking Summary
```

## 5. Use Case: Schedule Doctor Appointment
```mermaid
sequenceDiagram
    participant User
    participant Extension
    participant Agent
    participant LLM
    participant Tasks
    participant Browser
    participant Storage

    User->>Extension: "Schedule dentist appointment next week"
    Extension->>Agent: Create Medical Task
    
    %% Initial Setup
    Agent->>Storage: Get Healthcare Info
    Storage-->>Agent: Insurance & Provider Details
    
    Agent->>LLM: Generate Scheduling Plan
    LLM-->>Agent: Return Steps
    
    %% Provider Search
    Agent->>Tasks: Find In-Network Dentists
    Tasks->>Browser: Navigate to Insurance Portal
    Browser-->>Tasks: Provider Directory
    Tasks->>Browser: Search Local Dentists
    Browser-->>Tasks: Provider List
    
    %% Availability Check
    loop For Each Provider
        Tasks->>Browser: Check Availability
        Browser-->>Tasks: Available Slots
        Tasks->>Storage: Save Options
    end
    
    %% Analysis
    Agent->>LLM: Analyze Options
    LLM-->>Agent: Ranked Appointments
    
    %% User Selection
    Agent-->>Extension: Show Available Slots
    Extension-->>User: Request Time Selection
    User->>Extension: Confirm Preferred Time
    
    %% Booking Process
    Agent->>Tasks: Schedule Appointment
    Tasks->>Browser: Navigate to Booking
    Tasks->>Storage: Get Patient Info
    Storage-->>Tasks: Medical History
    
    %% Form Filling
    Tasks->>Browser: Fill Patient Forms
    Tasks->>Browser: Submit Insurance Info
    Browser-->>Tasks: Confirmation Page
    
    %% Confirmation
    Tasks->>Storage: Save Appointment
    Tasks-->>Agent: Scheduling Complete
    Agent-->>Extension: Show Confirmation
    Extension-->>User: Display Appointment Details
```

## 6. Component Interfaces for New Use Cases

### Flight Booking Interfaces
```typescript
// Flight search handling
interface FlightSearch {
    searchCriteria: {
        origin: string;
        destination: string;
        dates: DateRange;
        passengers: PassengerInfo[];
    };
    async searchMultipleProviders(): Promise<FlightOptions[]>;
    async compareResults(options: FlightOptions[]): Promise<RankedFlights>;
}

// Booking process
interface FlightBooking {
    async fillPassengerDetails(passengers: PassengerInfo[]): Promise<void>;
    async selectSeats(preferences: SeatPreference): Promise<SeatAssignment>;
    async processPayment(paymentInfo: PaymentDetails): Promise<BookingConfirmation>;
}

// Travel storage
interface TravelStorage {
    saveTravelPreferences(prefs: TravelPreferences): Promise<void>;
    saveBookingDetails(booking: BookingDetails): Promise<void>;
    getTravelerProfiles(): Promise<TravelerProfile[]>;
}
```

### Medical Appointment Interfaces
```typescript
// Provider search
interface ProviderSearch {
    searchCriteria: {
        specialty: string;
        location: Location;
        insurance: InsuranceInfo;
    };
    async findInNetworkProviders(): Promise<Provider[]>;
    async checkAvailability(provider: Provider): Promise<TimeSlot[]>;
}

// Appointment scheduling
interface AppointmentScheduling {
    async submitPatientInfo(info: PatientInfo): Promise<void>;
    async scheduleAppointment(slot: TimeSlot): Promise<Appointment>;
    async submitInsurance(insurance: InsuranceInfo): Promise<void>;
}

// Medical records
interface MedicalStorage {
    async getPatientHistory(): Promise<MedicalHistory>;
    async saveAppointment(appointment: Appointment): Promise<void>;
    async updateInsuranceInfo(insurance: InsuranceInfo): Promise<void>;
}
```

These additional examples demonstrate:
1. Parallel processing (searching multiple flight sites simultaneously)
2. Complex form handling (medical forms, flight booking forms)
3. Integration with external systems (insurance portals, airline booking systems)
4. Handling sensitive information (medical records, payment details)
5. Multi-step decision processes with user interaction
6. Different types of data persistence and retrieval

Each use case shows how the system adapts to different domains while maintaining consistent patterns in:
- Task decomposition
- User interaction points
- Data management
- External system integration
- Error handling and validation
- Progress tracking and storage