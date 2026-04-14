---
layout: post
title: "Dependency Inversion"
author: "Othmane Ataallah"
tags:
  - dependency-inversion
  - architecture
  - design-principles
---

*Notes on the structural mechanics of dependency inversion and the nature of control flow opposition.*

---

## Execution Flow

Execution flow is the sequential state progression of a program. It is the path execution takes—one instruction, statement, or subroutine after the next. This is a concrete property of the runtime environment. At any given moment in a single thread, execution is at exactly one node in this path, progressing forward linearly unless modified by a control structure.

---

## Control

Control is the computational authority to dictate the next transition in execution. A module holds control when it evaluates a condition, executes a loop iteration, or invokes another module. Control strictly concerns runtime decision-making.

---

## Control Flow

Control flow is the directed graph formed as control transfers between modules during execution. If routine `A` invokes routine `B`, control flows from `A` to `B`.

In direct invocations, the branch target is statically resolved at compile time. In invocations via abstractions—callbacks, interfaces, or higher-order functions—the target is resolved at runtime. Regardless of the dispatch mechanism, control flow always describes the actual sequence of operations executed.

---

## Source-Code Dependency

A source-code dependency is a lexical relationship between files. Module `A` depends on module `B` if the source text of `A` requires symbols exported by `B` to compile or parse. This is a static, structural property. It exists entirely independently of whether the program is ever executed.

> **Compilation versus Interpretation**  
> In compiled languages, lexical dependencies typically dictate the compilation graph. If module `B` modifies its exported signatures, module `A` requires recompilation. In interpreted environments, there is no static compilation phase, but the lexical dependency remains identical. Module `A` contains a textual reference to module `B`. Dependency inversion is an operation strictly on this static, lexical layer.

---

## Policy and Detail

### Policy
Policy is the high-level computational intent. It defines the state transitions and invariants that matter to the domain.
- Business rules
- Data validation
- Core domain logic

### Detail
Details (or mechanisms) are the IO-bound or infrastructure implementations required to execute the policy.
- Database drivers
- Network clients
- Third-party SDKs

The premise of dependency inversion is isolating the policy layer from the instability of the detail layer.

---

## Un-inverted Architecture

In a standard procedural codebase, high-level modules (policies) invoke low-level modules (details). Consequently, control flows from higher layers to lower layers. Because the higher layer must explicitly reference the lower layer's types to invoke its functions, the lexical dependency graph matches the control flow graph exactly. 

The immediate consequence of this alignment is that modifying a low-level detail forces a recompilation or invalidation of the high-level policy, even if the policy invariants remain untouched.

![Before Dependency Inversion Diagram]({{ "/assets/images/dip-before.png" | relative_url }})

---

## The Inversion Mechanism

Dependency inversion is a single structural manipulation: it forces the lexical dependency graph to point in the opposite direction of the control flow graph.

The runtime execution path cannot change—policy still inevitably invokes details to produce side effects. What changes is the location of the type definitions.

> **The Formal Postulate**  
> 1. High-level modules should not depend on low-level modules. Both should depend on abstractions.  
> 2. Abstractions should not depend on details. Details should depend on abstractions.  

The high-level module defines an abstract interface and retains lexical ownership of it. It dictates the contract it requires to proceed. The low-level module then implements this contract.

> **Module Boundary Ownership**  
> The abstraction must reside within the high-level module's boundary. If it resides in a shared third component, you achieve decoupling, but not inversion. Because the abstraction belongs to the high-level layer, the low-level module must statically import the high-level layer to fulfill the interface. The lexical dependency graph now points upward, directly opposing the downward flow of execution.

This directional opposition of lexical imports versus runtime execution is the precise definition of "inversion".

This mechanism is often conflated with sibling concepts:

| Concept | Definition |
|---------|------------|
| DIP | A static architectural rule concerning the directionality of lexical dependencies. Achieved by ensuring the interface is owned by the calling layer. |
| IoC | A behavioral pattern where framework code encapsulates the main execution loop, invoking user code via callbacks. It dictates who owns the execution context. |
| DI | A parameter-passing technique where instantiated objects are provided to an environment rather than constructed within it. It is the runtime mechanism that makes DIP possible without global state. |

![After Dependency Inversion Diagram]({{ "/assets/images/dip-after.png" | relative_url }})

---

## Purpose

The primary utility of this structural constraint is bounding the propagation of module invalidation.

High-level policy logic describes fundamental domain rules and is inherently stable. Detail logic, conversely, deals with external boundaries (filesystems, network protocols) and requires constant churn as infrastructure shifts.

By inverting the lexical dependency, the policy layer becomes a closed system. The detail layer can be entirely rewritten or swapped without altering a single instruction in the policy layer.

---
