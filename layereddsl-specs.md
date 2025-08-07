# LayeredDSL Specifications v2.0

_The definitive specification for LayeredDSL - A collaborative DSL-first architecture for AI-aligned software development_

## Table of Contents

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Schema Specification](#schema-specification)
4. [Type System](#type-system)
5. [Validation Rules](#validation-rules)
6. [AI Behavior Contracts](#ai-behavior-contracts)
7. [Code Generation Specifications](#code-generation-specifications)
8. [Error Handling](#error-handling)
9. [Version Compatibility](#version-compatibility)
10. [Examples](#examples)
11. [Appendices](#appendices)

## Introduction

LayeredDSL is a structured YAML-based domain-specific language designed to describe software systems at a conceptual level, enabling deterministic AI-assisted code generation and validation.

**Specification Version**: This document describes LayeredDSL Specification v2.0, a major revision introducing formal contracts, comprehensive type systems, and additional layers. Individual projects using LayeredDSL have their own version numbers independent of the specification version.

### Design Principles

- **Declarative**: Describe what the system is, not how to build it
- **Layer-Separated**: Clear boundaries between data, logic, UI, and infrastructure
- **AI-Optimized**: Minimal ambiguity, maximum expressiveness
- **Technology-Agnostic**: Maps to any programming language or framework
- **Traceable**: Every generated line of code maps back to DSL declarations

### Document Conventions

- Keywords "MUST", "SHOULD", "MAY" follow RFC 2119
- `monospace` indicates literal values or code
- _italics_ indicate conceptual terms
- **bold** indicates important concepts
- This is version 2.0 of the specification, representing a complete redesign from v1.0

## Core Concepts

### 1. Project Root

Every LayeredDSL document MUST begin with a `project` root node containing:

- `name`: String, required - Project identifier
- `description`: String, required - Human-readable project description
- `version`: String, optional - Project version (default: "1.0.0")
- `metadata`: Object, optional - Additional project metadata
  - `dsl_version`: String, recommended - LayeredDSL specification version this project conforms to (e.g., "2.0")
  - Additional key-value pairs for project-specific metadata

### 2. Layers

LayeredDSL organizes systems into distinct layers:

#### 2.1 Domain Layer (`domain`)

Defines data models and entities.

#### 2.2 Logic Layer (`logic`)

Defines business operations and rules.

#### 2.3 Component Layer (`components`)

Maps logic to implementation modules.

#### 2.4 Workflow Layer (`workflow`)

Orchestrates logic operations.

#### 2.5 UI Layer (`ui`)

Defines user interface structure.

#### 2.6 Infrastructure Layer (`infrastructure`)

Defines deployment and system requirements.

#### 2.7 Security Layer (`security`)

Defines access control and security policies.

## Schema Specification

### 3.1 Document Structure

```yaml
project:
  name: <string>
  description: <string>
  version: <string>  # Project version, not DSL spec version
  metadata:
    dsl_version: <string>  # LayeredDSL specification version (e.g., "2.0")
    <key>: <value>  # Additional metadata

domain:
  <EntityName>:
    <field>: <type>
    ...

logic:
  <OperationName>:
    inputs: [<field>, ...]
    output: <type>
    modifies: <entity>
    preconditions: [<condition>, ...]
    postconditions: [<condition>, ...]
    errors: [<error>, ...]

components:
  - id: <string>
    type: <module|service|library>
    language: <string>
    responsibilities: [<operation>, ...]
    dependencies: [<component_id>, ...]

workflow:
  - name: <string>
    trigger: <event|schedule|manual>
    steps:
      - <step_definition>
    error_handling: <strategy>

ui:
  pages:
    - name: <string>
      route: <string>
      components: [<ui_component>, ...]
      data_bindings: [<binding>, ...]

infrastructure:
  deployment:
    type: <container|serverless|vm>
    regions: [<region>, ...]
  resources:
    - type: <database|storage|queue>
      config: <object>

security:
  authentication:
    type: <oauth|jwt|session>
    provider: <string>
  authorization:
    roles: [<role>, ...]
    permissions:
      - role: <string>
        resources: [<resource>, ...]
        actions: [<action>, ...]
```

## Type System

### 4.1 Primitive Types

- `string`: UTF-8 text
- `int`: 64-bit signed integer
- `float`: IEEE 754 double-precision
- `boolean`: true/false
- `UUID`: RFC 4122 UUID
- `datetime`: ISO 8601 timestamp
- `date`: ISO 8601 date
- `time`: ISO 8601 time
- `json`: Arbitrary JSON object
- `binary`: Base64-encoded binary data

### 4.2 Composite Types

- `enum[...]`: Enumeration of values
- `array[<type>]`: Ordered collection
- `map[<key_type>, <value_type>]`: Key-value pairs
- `optional[<type>]`: Nullable type
- `reference[<entity>]`: Foreign key reference

### 4.3 Custom Types

Custom types can be defined in the domain layer and referenced elsewhere:

```yaml
domain:
  Money:
    amount: float
    currency: enum[USD, EUR, GBP]
```

## Validation Rules

### 5.1 Structural Validation

1. Document MUST be valid YAML
2. Document MUST have a `project` root node
3. All references MUST resolve to existing definitions
4. Circular dependencies MUST be detected and reported
5. Each component MUST have unique `id`

### 5.2 Semantic Validation

1. Logic operations MUST reference valid domain entities
2. UI components MUST bind to available data sources
3. Workflow steps MUST reference defined logic operations
4. Security permissions MUST reference existing resources

### 5.3 Consistency Rules

1. Every logic operation MUST be assigned to at least one component
2. Component dependencies MUST be acyclic
3. Workflow error handlers MUST reference defined error types
4. UI routes MUST be unique

## AI Behavior Contracts

### 6.1 Generation Rules

AI systems interacting with LayeredDSL MUST:

1. **Never infer beyond specification**: Generate only what is explicitly defined
2. **Maintain traceability**: Every generated artifact must reference its DSL source
3. **Respect layer boundaries**: Code for each layer must remain isolated
4. **Validate before generation**: Ensure DSL validity before code generation
5. **Incremental generation**: Generate only missing or modified components

### 6.2 Validation Protocol

```
1. Parse DSL document
2. Validate structure against schema
3. Check semantic consistency
4. Compare with existing codebase
5. Report discrepancies
6. Generate validation report
```

### 6.3 Modification Protocol

When receiving edit instructions, AI MUST:

1. Identify target (DSL or code)
2. Validate proposed changes
3. Update DSL if structural change
4. Regenerate affected code
5. Maintain backward compatibility

## Code Generation Specifications

### 7.1 Generation Templates

Each layer maps to specific code patterns:

#### Domain → Data Models

```yaml
domain:
  User:
    id: UUID
    email: string
    created_at: datetime
```

Generates (Python example):

```python
@dataclass
class User:
    id: UUID
    email: str
    created_at: datetime
```

#### Logic → Business Logic

```yaml
logic:
  CreateUser:
    inputs: [email, password]
    output: User
    modifies: User
```

Generates:

```python
def create_user(email: str, password: str) -> User:
    # Implementation based on DSL specification
    pass
```

### 7.2 Component Mapping

Components determine code organization:

```yaml
components:
  - id: user_service
    type: module
    language: python
    responsibilities: [CreateUser, UpdateUser, DeleteUser]
```

### 7.3 Multi-Language Support

Generators MUST support language-specific idioms:

- Python: Classes, type hints, decorators
- TypeScript: Interfaces, generics, decorators
- Java: Classes, annotations, generics
- Go: Structs, interfaces, error handling

## Error Handling

### 8.1 Error Categories

1. **Syntax Errors**: Invalid YAML structure
2. **Schema Errors**: Violations of DSL schema
3. **Reference Errors**: Unresolved references
4. **Consistency Errors**: Logical inconsistencies
5. **Generation Errors**: Code generation failures

### 8.2 Error Format

```yaml
error:
  category: <category>
  severity: <fatal|error|warning|info>
  location:
    line: <number>
    column: <number>
    path: <yaml.path>
  message: <string>
  suggestion: <string>
```

### 8.3 Error Recovery

- Syntax errors: Stop processing
- Schema errors: Skip invalid sections
- Reference errors: Mark as undefined
- Consistency errors: Report and continue
- Generation errors: Rollback changes

## Version Compatibility

### 9.1 Version Format

LayeredDSL follows semantic versioning:

- Major: Breaking changes to schema
- Minor: New features, backward compatible
- Patch: Bug fixes, clarifications

### 9.2 Migration Rules

Version migrations MUST:

1. Preserve semantic meaning
2. Provide automated migration tools
3. Document breaking changes
4. Support gradual migration

### 9.3 Compatibility Matrix

| DSL Spec Version | Min Generator | Max Generator | Breaking Changes  |
| ---------------- | ------------- | ------------- | ----------------- |
| 1.0.x            | 1.0.0         | 1.x.x         | -                 |
| 2.0.x            | 2.0.0         | 2.x.x         | Complete redesign |
| 2.1.x            | 2.1.0         | 2.x.x         | -                 |

## Examples

### 10.1 Complete Example: Task Management System

```yaml
project:
  name: TaskManager
  description: Simple task management application
  version: 1.0.0 # This is the TaskManager project version
  metadata:
    dsl_version: '2.0' # Conforms to LayeredDSL Specification v2.0
    author: 'Development Team'
    license: 'MIT'

domain:
  Task:
    id: UUID
    title: string
    description: optional[string]
    status: enum[todo, in_progress, done]
    assignee: optional[reference[User]]
    created_at: datetime
    updated_at: datetime

  User:
    id: UUID
    email: string
    name: string
    created_at: datetime

logic:
  CreateTask:
    inputs: [title, description, assignee_id]
    output: Task
    modifies: Task
    errors: [InvalidAssignee, MissingTitle]

  UpdateTaskStatus:
    inputs: [task_id, status]
    output: Task
    modifies: Task
    preconditions:
      - task_exists: 'task_id must reference existing task'
    postconditions:
      - status_changed: 'task.status equals input status'

  AssignTask:
    inputs: [task_id, user_id]
    output: Task
    modifies: Task
    errors: [TaskNotFound, UserNotFound]

components:
  - id: task_service
    type: service
    language: python
    responsibilities: [CreateTask, UpdateTaskStatus, AssignTask]
    dependencies: [user_service]

  - id: user_service
    type: service
    language: python
    responsibilities: [CreateUser, GetUser]

workflow:
  - name: TaskLifecycle
    trigger: event
    steps:
      - call: CreateTask
      - condition:
          if: assignee_id != null
          then: AssignTask
      - parallel:
          - notify: EmailService
          - log: AuditService
    error_handling:
      strategy: retry
      max_attempts: 3

ui:
  pages:
    - name: TaskBoard
      route: /tasks
      components:
        - type: kanban_board
          source: Task
          columns: [status]
          card_fields: [title, assignee.name]
        - type: create_button
          action: CreateTask

    - name: TaskDetail
      route: /tasks/:id
      components:
        - type: form
          source: Task
          fields: [title, description, status, assignee]
          actions: [UpdateTaskStatus, AssignTask]

infrastructure:
  deployment:
    type: container
    regions: [us-east-1, eu-west-1]
  resources:
    - type: database
      config:
        engine: postgresql
        version: '14'
        schema: auto_migrate
    - type: queue
      config:
        provider: redis
        purpose: event_processing

security:
  authentication:
    type: jwt
    provider: auth0
  authorization:
    roles: [admin, member, viewer]
    permissions:
      - role: admin
        resources: [Task, User]
        actions: [create, read, update, delete]
      - role: member
        resources: [Task]
        actions: [create, read, update]
      - role: viewer
        resources: [Task]
        actions: [read]
```

### 10.2 Validation Example

Input DSL with error:

```yaml
logic:
  DeleteTask:
    inputs: [task_id]
    output: boolean
    modifies: NonExistentEntity # Error: undefined entity
```

Validation output:

```yaml
error:
  category: ReferenceError
  severity: error
  location:
    line: 5
    column: 14
    path: logic.DeleteTask.modifies
  message: "Entity 'NonExistentEntity' is not defined in domain layer"
  suggestion: "Did you mean 'Task'?"
```

## Appendices

### A. Reserved Keywords

The following keywords are reserved and MUST NOT be used as identifiers:

- Layer names: `project`, `domain`, `logic`, `components`, `workflow`, `ui`, `infrastructure`, `security`
- Type keywords: `string`, `int`, `float`, `boolean`, `UUID`, `datetime`, `date`, `time`, `json`, `binary`
- Structural keywords: `inputs`, `output`, `modifies`, `preconditions`, `postconditions`, `errors`

### B. YAML Best Practices

1. Use 2-space indentation
2. Quote strings containing special characters
3. Use block scalar notation for multi-line strings
4. Avoid tabs
5. Keep line length under 120 characters

### C. Tooling Integration

LayeredDSL supports integration with:

- **VSCode Extension**: Syntax highlighting, validation, auto-completion
- **CLI Tool**: Validation, generation, migration
- **CI/CD**: GitHub Actions, GitLab CI, Jenkins
- **IDE Plugins**: IntelliJ, Sublime Text, Vim

### D. Performance Considerations

- DSL parsing: < 100ms for 1000-line document
- Validation: < 500ms for complete validation
- Code generation: < 2s for typical project
- Incremental updates: < 200ms per change

### E. Security Considerations

1. DSL files SHOULD be version controlled
2. Generated code MUST be reviewed before deployment
3. Sensitive data MUST NOT be stored in DSL
4. Access to DSL files SHOULD follow principle of least privilege

---

## Change Log

### Version 2.0.0 (2025-08-07)

- Complete specification overhaul with formal structure
- Added comprehensive type system with primitive and composite types
- Introduced AI Behavior Contracts with generation and validation protocols
- Added Infrastructure and Security layers
- Formalized validation rules (structural and semantic)
- Added error handling specifications and recovery strategies
- Introduced multi-language code generation templates
- Added version compatibility and semantic versioning
- Included complete working examples with all layers
- Added performance and security considerations
- Defined reserved keywords and YAML best practices
- Introduced workflow orchestration with parallel execution
- Added component dependency management
- Formalized traceability requirements

### Version 1.0.0 (Original)

- Initial concept and basic DSL structure
- Simple domain and logic layers
- Basic component mapping
