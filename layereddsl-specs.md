# LayeredDSL — Complete Guide for AI and Human Collaborators

## 📘 Purpose of this Document

This guide is designed for both **AI systems** and **human developers**. It explains how to understand, create, validate, and use **LayeredDSL** — a structured YAML-based language for describing **any software project** in a deterministic, platform-independent way.

LayeredDSL replaces traditional code-first development with a **map-first approach**. The DSL becomes the **single source of truth** from which code is generated only when needed, or validated against existing implementations.

**Key Principles:**

- **Declarative over Imperative**: Describe what the system should do, not how
- **Complete Specification**: Every behavior must be explicitly declared
- **No Hidden Assumptions**: AI must never infer undeclared functionality
- **Bidirectional Validation**: DSL can generate code or validate existing code

---

## 🚀 What is LayeredDSL?

**LayeredDSL** is a structured YAML metamodel that describes:

- **Data Domain**: Entities, their properties, and relationships
- **Business Logic**: Operations, behaviors, and contracts
- **Architecture**: Modular structure and component boundaries
- **User Interface**: Screens, forms, and interactions (optional)
- **Workflows**: Process flows and state transitions
- **Security**: Access control and permissions (optional)
- **Integration**: External APIs and services

### Design Goals

- **Platform-Independent**: Language and framework agnostic
- **Deterministic**: 1:1 mapping between DSL and implementation
- **Collaborative**: Human-editable and AI-parseable
- **Self-Contained**: Complete specification without external references
- **Versionable**: Track changes and evolution over time

---

## 📐 Type System

LayeredDSL supports a comprehensive type system for precise data modeling:

### Primitive Types

- `string`: Text data (UTF-8)
- `int`: Integer numbers
- `float`: Decimal numbers
- `boolean`: True/false values
- `datetime`: ISO 8601 timestamps
- `date`: Date without time
- `time`: Time without date
- `UUID`: Universally unique identifier

### Composite Types

- `enum[option1, option2, ...]`: Enumerated values
- `array[Type]`: Ordered list of elements
- `object{prop1: Type1, prop2: Type2}`: Structured objects
- `map[KeyType, ValueType]`: Key-value pairs
- `optional[Type]` or `Type?`: Nullable types

### Constraints (Optional)

```yaml
domain:
  Product:
    price: float{min: 0, max: 999999.99}
    name: string{minLength: 1, maxLength: 200}
    tags: array[string]{maxItems: 10}
    status: enum[draft, active, archived]{default: draft}
```

### Custom Types

```yaml
types:
  Email: string{pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"}
  Money: object{amount: float, currency: string{length: 3}}
```

---

## 🧱 DSL Structure — Complete Reference

### 1. `project` — Metadata and Context

```yaml
project:
  name: InvoiceManager
  version: 1.0.0
  description: Enterprise invoice management system with approval workflows
  type: web_application  # Options: web_application, api, library, mobile_app, cli
  repository: https://github.com/company/invoice-manager
  documentation: https://docs.company.com/invoice-manager
  authors:
    - name: Alice Johnson
      email: alice@company.com
      role: Lead Architect
    - name: Bob Smith
      role: Product Owner
  tags: [finance, saas, multi-tenant]
  created: 2025-01-15
  updated: 2025-07-28
```

**AI Interpretation**: Use project metadata to understand context, identify stakeholders, and maintain appropriate documentation standards.

---

### 2. `domain` — Data Entities and Relationships

Defines core entities with properties, relationships, and validation rules.

```yaml
domain:
  # Basic entity with constraints
  Customer:
    id: UUID
    name: string{minLength: 1, maxLength: 100}
    email: Email  # Custom type reference
    type: enum[individual, business]
    createdAt: datetime{auto: true}
    updatedAt: datetime{auto: true}

  # Entity with relationships
  Invoice:
    id: UUID
    customerId: UUID  # Foreign key
    customer: Customer{relation: many-to-one}  # Explicit relationship
    items: array[InvoiceItem]{relation: one-to-many}
    subtotal: Money
    tax: Money
    total: Money{computed: "subtotal + tax"}
    status: enum[draft, pending, paid, overdue, cancelled]{default: draft}
    dueDate: date
    paidAt: datetime?  # Optional field

  # Nested entity
  InvoiceItem:
    id: UUID
    invoiceId: UUID
    invoice: Invoice{relation: many-to-one}
    description: string
    quantity: int{min: 1}
    unitPrice: Money
    total: Money{computed: "quantity * unitPrice"}

  # Entity with validation rules
  User:
    id: UUID
    email: Email{unique: true}
    passwordHash: string{private: true}  # Won't be exposed in APIs
    role: enum[admin, accountant, viewer]
    active: boolean{default: true}
    lastLogin: datetime?

    # Custom validation rules
    validation:
      - rule: "email must be from company domain"
        expression: "email.endsWith('@company.com')"
```

**Relationship Types**:

- `one-to-one`: Single related entity
- `one-to-many`: Array of related entities
- `many-to-one`: Reference to parent entity
- `many-to-many`: Requires junction table (auto-generated)

---

### 3. `logic` — Business Operations

Declares all operations with contracts, side effects, and error handling.

```yaml
logic:
  # Basic CRUD operation
  CreateInvoice:
    inputs:
      customerId: UUID
      items: array[object{description: string, quantity: int, unitPrice: Money}]
      dueDate: date
    output: Invoice
    modifies: [Invoice]
    errors: [CustomerNotFound, InvalidDueDate]
    description: Creates a new invoice in draft status

  # Query operation with filters
  ListInvoices:
    inputs:
      filters:
        customerId?: UUID
        status?: array[enum[draft, pending, paid, overdue, cancelled]]
        dateRange?: object{from: date, to: date}
      pagination:
        page: int{default: 1, min: 1}
        pageSize: int{default: 20, min: 1, max: 100}
      sort:
        field: enum[createdAt, dueDate, total]{default: createdAt}
        order: enum[asc, desc]{default: desc}
    output: object{items: array[Invoice], total: int, page: int, pageSize: int}
    description: Retrieves paginated list of invoices with optional filters

  # Complex business operation
  ProcessPayment:
    inputs:
      invoiceId: UUID
      paymentMethod: enum[credit_card, bank_transfer, check]
      paymentDetails: object  # Polymorphic based on method
    output: Payment
    modifies: [Invoice, Payment]
    emits: [PaymentProcessed, InvoiceUpdated]  # Events
    errors: [InvoiceNotFound, PaymentFailed, InvoiceAlreadyPaid]
    validation:
      - "Invoice must be in 'pending' status"
      - "Payment amount must match invoice total"
    description: Processes payment for an invoice

  # Async operation
  GenerateMonthlyReport:
    inputs:
      month: int{min: 1, max: 12}
      year: int
    output: Report
    async: true
    timeout: 300  # seconds
    retryable: true
    description: Generates comprehensive monthly financial report
```

**Operation Attributes**:

- `inputs`: Parameters (can be nested objects)
- `output`: Return type
- `modifies`: Entities that will be created/updated/deleted
- `errors`: Possible error conditions
- `emits`: Events published (for event-driven systems)
- `validation`: Business rule assertions
- `async`: Long-running operation
- `idempotent`: Safe to retry

---

### 4. `components` — Architecture and Modules

Maps logical operations to implementation boundaries.

```yaml
components:
  # Backend service
  - id: invoice_service
    type: service
    technology: 
      language: python
      framework: fastapi
      database: postgresql
    responsibilities:
      - CreateInvoice
      - UpdateInvoice
      - ListInvoices
      - ProcessPayment
    dependencies:
      - notification_service  # Internal dependency
      - payment_gateway      # External service
    deployment:
      type: container
      replicas: 3
      resources:
        cpu: "500m"
        memory: "512Mi"

  # External integration
  - id: payment_gateway
    type: external_api
    provider: stripe
    version: "2023-10-16"
    operations:
      - ChargeCard
      - RefundPayment
    authentication:
      type: api_key
      location: header
      name: Authorization

  # Frontend application
  - id: web_app
    type: frontend
    technology:
      framework: react
      language: typescript
      bundler: vite
    consumes:  # Which backend APIs it uses
      - invoice_service
    deployment:
      type: static_hosting
      cdn: cloudflare

  # Shared library
  - id: validation_lib
    type: library
    language: python
    exports:
      - ValidateEmail
      - ValidateTaxNumber
    version: 2.1.0
```

---

### 5. `ui` — User Interface (Optional)

Describes screens, forms, and user interactions.

```yaml
ui:
  # Shared UI components
  components:
    InvoiceStatusBadge:
      type: badge
      props:
        status: enum[draft, pending, paid, overdue, cancelled]
      styling:
        draft: gray
        pending: yellow
        paid: green
        overdue: red
        cancelled: gray

  # Application pages
  pages:
    # List page with filters
    - name: InvoiceList
      route: "/invoices"
      title: "Invoices"
      components:
        # Filter bar
        - type: filter_bar
          id: invoice_filters
          filters:
            - field: status
              type: multi_select
              options: [draft, pending, paid, overdue, cancelled]
            - field: dateRange
              type: date_range
            - field: customer
              type: search
              source: Customer

        # Data table
        - type: table
          source: ListInvoices
          columns:
            - field: id
              label: "Invoice #"
              sortable: true
            - field: customer.name
              label: "Customer"
              sortable: true
            - field: total
              label: "Amount"
              format: currency
              sortable: true
            - field: status
              label: "Status"
              component: InvoiceStatusBadge
            - field: dueDate
              label: "Due Date"
              format: date
              sortable: true
          actions:
            - label: "View"
              action: navigate
              target: "/invoices/{id}"
            - label: "Edit"
              action: navigate
              target: "/invoices/{id}/edit"
              condition: "status === 'draft'"

        # Action buttons
        - type: button
          label: "New Invoice"
          action: navigate
          target: "/invoices/new"
          style: primary

    # Form page
    - name: CreateInvoice
      route: "/invoices/new"
      title: "Create Invoice"
      components:
        - type: form
          onSubmit: CreateInvoice
          layout: two_column
          sections:
            - title: "Customer Information"
              fields:
                - name: customerId
                  type: select
                  label: "Customer"
                  source: Customer
                  required: true
                  searchable: true

            - title: "Invoice Items"
              fields:
                - name: items
                  type: array
                  label: "Line Items"
                  minItems: 1
                  itemFields:
                    - name: description
                      type: text
                      label: "Description"
                      required: true
                    - name: quantity
                      type: number
                      label: "Quantity"
                      min: 1
                      default: 1
                    - name: unitPrice
                      type: money
                      label: "Unit Price"
                      required: true

            - title: "Payment Terms"
              fields:
                - name: dueDate
                  type: date
                  label: "Due Date"
                  min: today
                  required: true

          actions:
            - type: submit
              label: "Create Invoice"
              style: primary
            - type: cancel
              label: "Cancel"
              action: navigate
              target: "/invoices"
```

---

### 6. `workflow` — Process Flows

Describes complex multi-step processes with conditions, loops, and error handling.

#### 📋 Workflow Patterns Reference

##### ✅ **Complete Pattern Template**

```yaml
workflow:
  - name: CompletePatternExample
    description: Demonstrates all available workflow patterns
    steps:
      # Basic function call
      - call: FetchData
        condition: user_authenticated == true

      # Loop over collection
      - call: ProcessItem
        loop: each item in Items[]
        condition: item.active == true

      # Wait for condition or time
      - call: WaitForApproval
        wait: until approval.status == "approved"
        timeout: 48h

      # Parallel execution
      - parallel:
          - call: SendEmail
          - call: LogActivity
          - call: UpdateMetrics

      # Retry on failure
      - call: ExternalAPICall
        retry:
          attempts: 3
          delay_seconds: 5
          backoff: exponential

      # Error handling
      - call: RiskyOperation
        on_error: HandleFailure

      # Conditional branching
      - branch:
          when:
            - condition: amount > 10000
              call: RequireApproval
            - condition: amount > 1000
              call: NotifyManager
            - condition: amount <= 1000
              call: AutoProcess
```

##### 📖 **Pattern Details**

| Pattern       | Description             | DSL Syntax                             | Code Equivalent               |
| ------------- | ----------------------- | -------------------------------------- | ----------------------------- |
| **call**      | Execute a function      | `call: FunctionName`                   | `functionName()`              |
| **condition** | Conditional execution   | `condition: expression`                | `if (expression)`             |
| **loop**      | Iterate over collection | `loop: each item in Items[]`           | `for (item of items)`         |
| **wait**      | Pause execution         | `wait: until condition` or `wait: 30s` | `await` / `sleep()`           |
| **parallel**  | Concurrent execution    | `parallel: [calls]`                    | `Promise.all()` / threads     |
| **retry**     | Retry on failure        | `retry: {attempts: 3}`                 | `try/catch` with loop         |
| **on_error**  | Error fallback          | `on_error: FallbackAction`             | `catch (e) { fallback() }`    |
| **branch**    | Multi-way conditional   | `branch: when: [conditions]`           | `if/else if/else` or `switch` |
| **foreach**   | Parallel iteration      | `foreach: item in items`               | Parallel `map()`              |
| **timeout**   | Time limit              | `timeout: 30s`                         | `setTimeout()` / deadline     |

##### 🔄 **Pattern Combinations**

```yaml
# Retry with exponential backoff and error handling
- call: UnreliableService
  retry:
    attempts: 5
    delay_seconds: 1
    backoff: exponential
    max_delay: 60
  on_error: 
    call: UseBackupService

# Conditional parallel execution
- condition: bulk_processing_enabled == true
  parallel:
    - foreach: batch in Batches[]
      call: ProcessBatch
      max_parallel: 5

# Loop with timeout and break condition
- loop:
    while: status != "complete"
    max_iterations: 100
    steps:
      - call: CheckStatus
      - wait: 10s
    timeout: 30m
    on_timeout: CancelOperation
```

##### 🎯 **Real-World Pattern Examples**

```yaml
# API polling with backoff
- name: PollExternalAPI
  steps:
    - loop:
        while: result.status == "processing"
        steps:
          - call: CheckAPIStatus
            output: result
          - wait: "${iteration * 5}s"  # Increasing delay
        max_iterations: 20

# Batch processing with error recovery
- name: BatchImport
  steps:
    - call: SplitIntoBatches
      output: batches

    - foreach: batch in batches
      max_parallel: 3
      steps:
        - call: ValidateBatch
          on_error: 
            call: QuarantineBatch
            continue: true  # Continue with next batch

        - call: ProcessBatch
          retry:
            attempts: 2

        - call: VerifyBatch
          condition: batch.critical == true
```

#### 📊 **Workflow Examples**

```yaml
workflow:
  # Invoice lifecycle workflow
  - name: InvoiceLifecycle
    description: Manages invoice from creation to payment
    triggers:
      - event: InvoiceCreated
      - schedule: "0 9 * * *"  # Daily at 9 AM for overdue checks

    steps:
      # Initial validation
      - call: ValidateCustomerCredit
        inputs: 
          customerId: "${trigger.invoice.customerId}"
        on_error: 
          call: NotifyAccountingTeam
          then: stop

      # Conditional approval
      - branch:
          when:
            - condition: "${invoice.total} > 10000"
              steps:
                - call: RequestManagerApproval
                  wait: until "${approval.status} != 'pending'"
                  timeout: 48h
                  on_timeout: 
                    call: EscalateToDirector

            - condition: "${customer.type} == 'business'"
              call: ApplyBusinessDiscount

      # Send invoice
      - call: SendInvoiceEmail
        retry:
          attempts: 3
          backoff: exponential
          initial_delay: 60s

      # Payment monitoring loop
      - loop:
          until: "${invoice.status} == 'paid' OR ${invoice.status} == 'cancelled'"
          max_iterations: 90  # days
          steps:
            - wait: 1d

            - branch:
                when:
                  - condition: "${invoice.dueDate} < ${now()} AND ${invoice.status} == 'pending'"
                    steps:
                      - call: UpdateInvoiceStatus
                        inputs:
                          status: overdue
                      - call: SendOverdueNotice
                      - wait: 7d
                      - call: InitiateCollectionProcess

      # Completion
      - call: ArchiveInvoice
        condition: "${invoice.status} == 'paid'"

  # Parallel processing example
  - name: MonthEndProcessing
    triggers:
      - schedule: "0 0 1 * *"  # First day of month

    steps:
      - call: GetUnprocessedInvoices
        output: unprocessedInvoices

      - parallel:
          - call: GenerateAgingReport
          - call: CalculateMonthlyRevenue
          - call: UpdateCustomerCreditScores

          - foreach: invoice in "${unprocessedInvoices}"
            call: ProcessInvoice
            max_parallel: 10

      - call: SendMonthlyReport
        inputs:
          recipients: ["cfo@company.com", "accounting@company.com"]
```

**Workflow Patterns**:

- `call`: Execute a logic operation
- `condition`: Boolean expression for conditional execution
- `loop`: Iterate over collections or until condition
- `wait`: Pause for time or event
- `parallel`: Concurrent execution
- `foreach`: Parallel iteration with concurrency control
- `branch`: Multi-way conditional (if/else if/else)
- `retry`: Automatic retry with backoff
- `on_error`: Error handling and recovery
- `timeout`: Maximum execution time

---

### 7. `security` — Access Control (Optional)

Defines authentication, authorization, and data privacy rules.

```yaml
security:
  # Authentication methods
  authentication:
    - type: oauth2
      providers: [google, microsoft]

    - type: api_key
      header: X-API-Key

    - type: jwt
      issuer: "https://auth.company.com"
      audience: "invoice-api"

  # Role definitions
  roles:
    - id: admin
      description: Full system access
      inherits: []

    - id: accountant
      description: Manage invoices and payments
      inherits: [viewer]

    - id: viewer
      description: Read-only access
      inherits: []

    - id: customer
      description: External customer access
      inherits: []

  # Permission rules
  permissions:
    # Operation-level permissions
    - action: CreateInvoice
      allowed_roles: [admin, accountant]

    - action: ProcessPayment
      allowed_roles: [admin, accountant]
      rate_limit: "100/hour"

    - action: ListInvoices
      allowed_roles: [admin, accountant, viewer]
      data_filter: |
        if (user.role == 'customer') {
          return invoices.filter(i => i.customerId == user.customerId)
        }
        return invoices

    - action: GenerateMonthlyReport
      allowed_roles: [admin]
      time_restriction: "business_hours"  # 9 AM - 5 PM

  # Field-level security
  field_access:
    Invoice:
      - field: internalNotes
        read: [admin, accountant]
        write: [admin, accountant]

      - field: creditScore
        read: [admin]
        write: []  # Read-only

  # Data privacy
  privacy:
    - entity: Customer
      pii_fields: [email, phone, taxId]
      retention_days: 2555  # 7 years
      anonymization_rules:
        - field: email
          method: hash
        - field: phone
          method: partial_mask
          keep_last: 4
```

---

### 8. `integrations` — External Services

Defines connections to external APIs, databases, and services.

```yaml
integrations:
  # Third-party API
  - id: stripe_api
    type: rest_api
    base_url: "https://api.stripe.com/v1"
    authentication:
      type: bearer
      token_env: STRIPE_SECRET_KEY
    operations:
      CreateCharge:
        method: POST
        path: "/charges"
        request:
          amount: int  # In cents
          currency: string
          source: string
        response: StripeCharge
        errors:
          - code: card_declined
            retry: false
          - code: rate_limit
            retry: true
            backoff: exponential

  # Database connection
  - id: main_db
    type: database
    engine: postgresql
    connection_env: DATABASE_URL
    pool_size: 20
    migrations:
      auto: true
      directory: "./migrations"

  # Message queue
  - id: event_bus
    type: message_queue
    provider: rabbitmq
    connection_env: AMQP_URL
    exchanges:
      - name: invoice_events
        type: topic
        durable: true
    queues:
      - name: payment_processing
        routing_key: "invoice.payment.*"

  # Email service
  - id: email_service
    type: email
    provider: sendgrid
    api_key_env: SENDGRID_API_KEY
    templates:
      - id: invoice_created
        subject: "Invoice #{invoice.id} from {company.name}"
      - id: payment_received
        subject: "Payment received for Invoice #{invoice.id}"
```

---

### 9. `mapping` — Explicit Bindings

Connects logic operations to their implementations.

```yaml
mapping:
  # Service mappings
  CreateInvoice: components.invoice_service.CreateInvoice
  UpdateInvoice: components.invoice_service.UpdateInvoice
  ProcessPayment: components.invoice_service.ProcessPayment

  # External API mappings
  ChargeCard: integrations.stripe_api.CreateCharge

  # Library mappings
  ValidateEmail: components.validation_lib.ValidateEmail

  # Event mappings
  PaymentProcessed: integrations.event_bus.invoice_events.payment.processed

  # Database mappings
  Invoice: integrations.main_db.tables.invoices
  Customer: integrations.main_db.tables.customers
```

---

### 10. `tests` — Test Specifications (Optional)

Defines test cases and scenarios.

```yaml
tests:
  # Unit test
  - name: CreateInvoice_ValidInput_Success
    type: unit
    target: CreateInvoice
    input:
      customerId: "test-customer-123"
      items:
        - description: "Consulting services"
          quantity: 10
          unitPrice: {amount: 100.00, currency: "USD"}
      dueDate: "2025-08-15"
    expected:
      status: draft
      total: {amount: 1000.00, currency: "USD"}

  # Integration test
  - name: PaymentFlow_EndToEnd
    type: integration
    steps:
      - call: CreateInvoice
        capture: invoice
      - call: ProcessPayment
        inputs:
          invoiceId: "${invoice.id}"
          paymentMethod: credit_card
      - assert: "${invoice.status} == 'paid'"

  # Load test
  - name: InvoiceCreation_LoadTest
    type: performance
    target: CreateInvoice
    load:
      users: 100
      duration: 300s
      ramp_up: 60s
    assertions:
      - p95_latency: "< 500ms"
      - error_rate: "< 1%"
```

---

### 11. `metadata` — Version Control and History

Tracks changes, migration paths, and evolution.

```yaml
metadata:
  dsl_version: 2.0  # Version of LayeredDSL spec
  schema_version: 1.3.0  # Version of this project's schema

  # Compatibility
  compatible_versions:
    min: 1.0.0
    max: 2.0.0

  # Change history
  history:
    - version: 1.3.0
      date: 2025-07-28
      author: Alice Johnson
      changes:
        - "Added payment processing workflow"
        - "Enhanced security model with field-level access"
        - "Integrated Stripe payment gateway"
      breaking: false

    - version: 1.2.0
      date: 2025-06-15
      author: Bob Smith
      changes:
        - "Added customer credit scoring"
        - "Implemented monthly reporting"
      migration:
        - "Run script: migrations/v1.2.0_add_credit_scores.sql"

  # Migration paths
  migrations:
    from_1_0_to_1_1:
      - type: schema
        description: "Add 'createdAt' to all entities"
        auto: true

    from_1_1_to_1_2:
      - type: data
        description: "Populate customer credit scores"
        script: "migrations/populate_credit_scores.py"

  # Tags for tooling
  tags:
    - stable
    - production-ready
    - requires-review  # For human validation
```

---

## 🎯 Step-by-Step Guide to Writing LayeredDSL

### 1. Start with Project Context

Define the what and why before the how.

```yaml
project:
  name: YourProject
  description: Clear, one-line description
  type: web_application
```

### 2. Model Your Domain

Think about the core entities and their relationships.

```yaml
domain:
  Entity1:
    id: UUID
    relatedEntity: Entity2{relation: many-to-one}
```

### 3. Define Core Operations

Start with basic CRUD, then add business logic.

```yaml
logic:
  CreateEntity:
    inputs: [field1, field2]
    output: Entity1
    modifies: [Entity1]
```

### 4. Map to Components

Decide architectural boundaries.

```yaml
components:
  - id: main_service
    responsibilities: [CreateEntity, UpdateEntity]
```

### 5. Design UI (if needed)

Create pages and forms that use your logic.

```yaml
ui:
  pages:
    - name: EntityList
      components:
        - type: table
          source: ListEntities
```

### 6. Add Workflows

Define multi-step processes.

```yaml
workflow:
  - name: EntityLifecycle
    steps:
      - call: CreateEntity
      - call: NotifyUser
```

### 7. Secure Your System

Add roles and permissions.

```yaml
security:
  roles: [admin, user]
  permissions:
    - action: CreateEntity
      allowed_roles: [admin]
```

### 8. Document External Dependencies

List integrations and APIs.

```yaml
integrations:
  - id: external_api
    type: rest_api
```

### 9. Create Explicit Mappings

Connect everything together.

```yaml
mapping:
  CreateEntity: components.main_service.CreateEntity
```

### 10. Track Changes

Maintain history and versioning.

```yaml
metadata:
  version: 1.0.0
  history:
    - date: 2025-07-28
      changes: ["Initial version"]
```

---

## 🧠 AI Behavior Rules

When processing LayeredDSL, AI systems MUST follow these rules:

### 1. **Strict Adherence**

- Only implement what is explicitly declared in the DSL
- Never infer functionality not specified
- Treat the DSL as the single source of truth

### 2. **Validation First**

- Validate YAML syntax before processing
- Check all references (components, types, operations)
- Ensure mappings are complete

### 3. **Code Generation Rules**

- Generate code only for unmapped operations
- Include DSL references as comments in generated code
- Preserve existing code that matches DSL specifications

### 4. **Error Handling**

- Report all validation errors before proceeding
- Never generate partial implementations
- Provide clear error messages referencing DSL sections

### 5. **Incremental Updates**

- Compare existing code with DSL before generating
- Generate only missing or changed components
- Maintain backward compatibility unless explicitly marked as breaking

### 6. **Documentation**

- Generate inline documentation from DSL descriptions
- Create API documentation from logic contracts
- Maintain traceability between DSL and implementation

---

## 🔄 Migration Patterns

### From Existing Codebase

1. **Analyze**: Extract domain model from existing code
2. **Document**: Create DSL matching current implementation
3. **Validate**: Ensure DSL correctly describes the system
4. **Refactor**: Gradually align code with DSL structure

### Versioning Strategy

1. **Semantic Versioning**: Use MAJOR.MINOR.PATCH
2. **Breaking Changes**: Increment MAJOR, provide migration guide
3. **New Features**: Increment MINOR, maintain compatibility
4. **Bug Fixes**: Increment PATCH, no DSL structure changes

### Example Migration

```yaml
metadata:
  migrations:
    from_1_0_to_2_0:
      - type: rename
        old: Customer.customerName
        new: Customer.name
      - type: add_field
        entity: Invoice
        field: metadata
        default: {}
```

---

## 🚦 Validation Checklist

Before considering a LayeredDSL complete:

- [ ] All entities have proper types and constraints
- [ ] All relationships are bidirectional where appropriate
- [ ] Every logic operation has inputs, outputs, and descriptions
- [ ] All logic operations are mapped to components
- [ ] Security rules cover all sensitive operations
- [ ] Workflows handle error cases
- [ ] External integrations have error handling
- [ ] Metadata includes version and history
- [ ] No circular dependencies exist
- [ ] All custom types are defined

---

## 💡 Best Practices

### For Humans

1. **Start Simple**: Begin with core entities and basic CRUD
2. **Iterate**: Add complexity gradually
3. **Document Intent**: Use descriptions liberally
4. **Think in Contracts**: Define what, not how
5. **Version Everything**: Track all changes

### For AI Systems

1. **Parse Completely**: Load entire DSL before processing
2. **Validate Thoroughly**: Check all constraints and references
3. **Generate Conservatively**: Only create what's needed
4. **Preserve Context**: Maintain DSL references in code
5. **Report Clearly**: Provide actionable feedback

---

## 🎯 Example Prompts for AI Interaction

- "Validate this LayeredDSL for completeness and consistency"
- "Generate only the missing components for this DSL"
- "Compare this DSL with the existing codebase and report differences"
- "Create a DSL from this project description: [description]"
- "Add user authentication workflow to this DSL"
- "Generate API documentation from this DSL"
- "Create database migrations for DSL version 1.2 to 1.3"

---

## ✅ Conclusion

LayeredDSL enables a new paradigm of software development where:

- **Specification precedes implementation**
- **AI and humans collaborate through a common language**
- **Code becomes a derived artifact, not the source of truth**
- **Systems remain maintainable and evolvable**

By following this guide, both humans and AI can effectively create, validate, and implement software systems with clarity, precision, and collaboration.
