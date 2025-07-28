# LayeredDSL: Specification & Agent Guide

**A collaborative DSL-first architecture for building AI-aligned software.**

> _"AI writes better code when you stop thinking in code."_  

---

## 📘 What is LayeredDSL?

**LayeredDSL** is a structured YAML-based metamodel that describes software projects **at a conceptual level**, before any code is written — or alongside it.

It is designed to support **AI-assisted software development** by offering a clear, unambiguous blueprint of the system: data, logic, UI, workflows, security, and mappings. Instead of asking LLMs to work blindly across a massive codebase, developers and AI can collaborate **on the DSL map**, which becomes the **single source of truth**.

Once the map is validated, AI can safely generate or validate the code — with zero hallucinations and full traceability.

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

## 🧩 DSL Structure (at a Glance)

```yaml
project:
  name: InvoiceManager
  description: Track invoices and notify clients

domain:
  Invoice:
    id: UUID
    amount: float
    status: enum[draft, paid]

logic:
  CreateInvoice:
    inputs: [customerId, amount]
    output: Invoice
    modifies: Invoice

components:
  - id: billing_service
    type: module
    language: python
    responsibilities:
      - CreateInvoice

workflow:
  - name: InvoiceLifecycle
    steps:
      - call: CreateInvoice
      - call: NotifyCustomer

ui:
  pages:
    - name: Dashboard
      route: "/"
      components:
        - type: table
          source: Invoice
          columns: [customerId, amount, status]
```

---

## 🤖 AI Behavior Rules

1. ✅ Do **not generate code** unless the DSL is validated.
2. ❌ Never infer logic beyond what's written.
3. 🔄 Always map logic to components.
4. 🔍 Use the DSL to check if code already exists.
5. 🧠 Only generate missing parts if required — never the whole project.
6. 💬 Accept edit instructions to update either DSL or code to match.

---

## 🔁 Human-AI Workflow

1. **Define**: Human writes or co-writes DSL with AI help.
2. **Validate**: AI compares DSL to codebase, flags gaps or mismatches.
3. **Generate**: AI fills in only the missing or outdated parts — no freelancing.

---

## 📖 Related Reading

* 📰 [Medium article: From Chaos to Clarity — Why AI Writes Better Code When You Stop Thinking in Code](https://medium.com/@yourhandle)
* 📚 [layered-dsl-guide.md](./layered-dsl-guide.md)

---

## 🌍 Contributing

We welcome improvements to prompt templates, DSL schema ideas, and automation tooling. Open a PR or create an issue to join the discussion.

---

> **LayeredDSL is the interface between concept and implementation — for both humans and machines.**
