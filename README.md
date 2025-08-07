# LayeredDSL v2.0: Specification & Agent Guide

**A collaborative DSL-first architecture for building AI-aligned software with formal contracts and comprehensive type systems.**

> _"AI writes better code when you stop thinking in code."_  

---

## 📘 What is LayeredDSL?

**LayeredDSL** is a structured YAML-based domain-specific language that describes software systems **at a conceptual level**, enabling deterministic AI-assisted code generation and validation.

**Version 2.0** introduces major improvements:

- 🎯 **Formal AI Behavior Contracts** - Predictable AI interactions
- 🔧 **Comprehensive Type System** - Primitive, composite, and custom types
- 🏗️ **Extended Layer Architecture** - 7 distinct layers for complete system modeling
- ⚡ **Enhanced Validation Rules** - Structural, semantic, and consistency checks
- 🔄 **Version Compatibility Framework** - Seamless migration support

Instead of asking LLMs to work blindly across massive codebases, developers and AI collaborate **on the DSL specification**, which becomes the **single source of truth** with complete traceability.

---

## 🔍 Why This Repository

This repository contains the **official usage instructions, prompt templates, and behavior protocols** for any LLM, dev tool, or agent that works with LayeredDSL.

It provides:

- ✅ Guidelines for **how AI should interpret and act on LayeredDSL**
- 🧠 Prompt formats for developers working with AI assistants
- 📐 DSL schema structure and generation examples
- 🔁 AI validation rules, scaffolding instructions, and interaction contracts

---

## ✨ Key Advantages of DSL-First Development

- **Determinism**: AI behavior becomes predictable and auditable.
- **Clarity**: Everyone — devs, AI, and stakeholders — shares the same model.
- **Scalability**: Works on small apps and large, distributed systems.
- **Maintainability**: Changes in requirements affect only the map; code follows.
- **Cross-language**: The same DSL can power implementations in any tech stack.
- **AI Readiness**: Much more efficient than prompting with full source code.

> A well-written DSL document is a smaller, more focused, and more intelligent interface for AI reasoning than any codebase can be.

---

## 🧩 DSL Structure v2.0 (Complete Architecture)

LayeredDSL v2.0 organizes systems into **7 distinct layers** for comprehensive modeling:

```yaml
project:
  name: TaskManager
  description: Complete task management system
  version: 1.0.0  # Project version
  metadata:
    dsl_version: '2.0'  # LayeredDSL specification version
    author: 'Development Team'

# 1. DOMAIN LAYER - Data models and entities
domain:
  Task:
    id: UUID
    title: string
    status: enum[todo, in_progress, done]
    assignee: optional[reference[User]]
    created_at: datetime

# 2. LOGIC LAYER - Business operations with contracts
logic:
  CreateTask:
    inputs: [title, description]
    output: Task
    modifies: Task
    preconditions: ["title is not empty"]
    postconditions: ["task exists with given title"]
    errors: [ValidationError, DatabaseError]

# 3. COMPONENT LAYER - Implementation modules
components:
  - id: task_service
    type: module
    language: python
    responsibilities: [CreateTask, UpdateTask]
    dependencies: [database_service]

# 4. WORKFLOW LAYER - Operation orchestration
workflow:
  - name: TaskCreationFlow
    trigger: api_request
    steps:
      - call: ValidateInput
      - call: CreateTask
      - call: NotifyAssignee
    error_handling: rollback

# 5. UI LAYER - User interface structure
ui:
  pages:
    - name: TaskDashboard
      route: "/tasks"
      components:
        - type: task_list
          data_source: Task
          actions: [create, edit, delete]

# 6. INFRASTRUCTURE LAYER - Deployment and resources
infrastructure:
  deployment:
    type: container
    regions: [us-east-1]
  resources:
    - type: database
      engine: postgresql
      size: small

# 7. SECURITY LAYER - Access control and policies
security:
  authentication:
    type: jwt
    provider: auth0
  authorization:
    roles: [admin, user, guest]
    permissions:
      - role: user
        resources: [Task]
        actions: [read, create, update]
```

---

## 🤖 AI Behavior Contracts v2.0

**Formal contracts ensure predictable AI behavior:**

### Generation Rules

AI systems interacting with LayeredDSL MUST:

1. ✅ **Never infer beyond specification** - Generate only what is explicitly defined
2. 🔍 **Maintain traceability** - Every generated artifact must reference its DSL source
3. 🏗️ **Respect layer boundaries** - Code for each layer must remain isolated
4. ⚙️ **Validate before generation** - Ensure DSL validity before code generation
5. 🔄 **Incremental generation** - Generate only missing or modified components

### Validation Protocol

1. Parse DSL document
2. Validate structure against schema
3. Check semantic consistency
4. Compare with existing codebase
5. Report discrepancies
6. Generate validation report

### Modification Protocol

When receiving edit instructions, AI MUST:

1. Identify target (DSL or code)
2. Validate proposed changes
3. Update DSL if structural change
4. Regenerate affected code
5. Maintain backward compatibility

---

## 🔁 Human-AI Workflow v2.0

**Enhanced collaborative development process:**

1. **Define**: Human writes or co-writes DSL with AI assistance, leveraging the comprehensive type system
2. **Validate**: AI performs structural, semantic, and consistency validation against the formal schema
3. **Generate**: AI generates only missing or modified components with full traceability
4. **Verify**: AI ensures generated code respects layer boundaries and contracts
5. **Iterate**: Changes flow bidirectionally between DSL and code with version compatibility

### Key v2.0 Enhancements

- **Type Safety**: Comprehensive type system with primitives, composites, and custom types
- **Error Handling**: Formal error categories with recovery strategies
- **Version Migration**: Automated migration tools for specification updates
- **Multi-Language Support**: Language-specific code generation with proper idioms

---

## 🔧 Type System v2.0

**Comprehensive type safety for reliable code generation:**

### Primitive Types

- `string`, `int`, `float`, `boolean`
- `UUID`, `datetime`, `date`, `time`
- `json`, `binary`

### Composite Types

- `enum[value1, value2, ...]` - Enumeration of values
- `array[<type>]` - Ordered collections
- `map[<key_type>, <value_type>]` - Key-value pairs
- `optional[<type>]` - Nullable types
- `reference[<entity>]` - Foreign key references

### Custom Types

Define reusable types in the domain layer:

```yaml
domain:
  Money:
    amount: float
    currency: enum[USD, EUR, GBP]
  
  Address:
    street: string
    city: string
    postal_code: string
```

---

## 🔄 Version Compatibility

**Seamless migration between specification versions:**

- **Semantic Versioning**: Major.Minor.Patch format
- **Automated Migration**: Tools for upgrading between versions
- **Backward Compatibility**: Gradual migration support
- **Breaking Change Documentation**: Clear upgrade paths

| DSL Spec Version | Breaking Changes | Migration Required |
|------------------|------------------|--------------------|
| 1.0.x → 2.0.x    | Complete redesign | Yes, automated tools available |
| 2.0.x → 2.1.x    | None | No |

---

## 📚 Related Reading

- 📄 [LayeredDSL Specifications v2.0](./layereddsl-specs.md) - Complete technical specification
- 📰 [From Chaos to Clarity — Why AI Writes Better Code When You Stop Thinking in Code](https://medium.com/@yourhandle)
- 📚 [layered-dsl-guide.md](./layered-dsl-guide.md) - Implementation guide

---

## 🌍 Contributing

We welcome improvements to prompt templates, DSL schema ideas, and automation tooling. Open a PR or create an issue to join the discussion.

---

> **LayeredDSL is the interface between concept and implementation — for both humans and machines.**
