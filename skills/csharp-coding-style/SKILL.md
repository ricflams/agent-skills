---
name: csharp-coding-style
description: Coding style guide for experienced C# and .NET developer. Use when writing C# code from scratch, reviewing existing code, refactoring, or providing coding suggestions. Covers modern C# 14 features, formatting preferences, architectural decisions, error handling patterns, and readability principles. Apply these guidelines to all C# code generation and code review tasks.
---

# C# Coding Style Guide

## Core Philosophy

- Optimize code for human comprehension
- Favor clarity over rigid formatting rules
- Prefer DRY unless repetition is clearer
- Follow C# standards unless these guidelines conflict
- Bend any rule if it improves understanding, maintainability, or efficiency; readability trumps dogma

## Non-Negotiable Conventions

- Tabs for indentation (never spaces)
- Standard C# naming conventions
- Private fields: `_field`

## Principles

- Prefer composition over inheritance
- Prefer immutable data structures where practical
- Prefer built-in alternatives to external packages
- Prefer properties over fields (no public fields except `static readonly`)
- Keep related logic together
- Allow long methods if cohesion improves understanding; don't extract just to reduce line count
- Prefer early returns and guard clauses over nesting
- Omit braces for single-line guards, returns, break, or continue; place control statement on separate indented line
- Place local helper functions after main logic
- Class member order: Static fields → Properties → Public methods → Protected → Private → Nested classes
- Initialize variables at definition in functional flow; avoid declare-then-assign unless initialization is complex
- Prefer LINQ over loops for transformations, but not if it obscures logic
- Use `TryXxx(input, out var result)` for fallible operations instead of sentinel values like -1
- Separate logical chunks with single blank line
- Comments explain why, not what; place immediately before code
- Short names when longer ones obscure logic: `i`, `x`, `n`, `w`; use judiciously to let logic stand out
- Descriptive names when context unclear: `map`, `robot`, `moves`, `sum`
- Match existing code style when making fixes

## Modern C# (C# 14)

Always use:
- `var` for local variables
- Pattern matching in switches, `is`, conditionals
- Target-typed `new()` instead of `new Type()`
- Expression-bodied members for simple properties and one-liners
- Tuple deconstruction: `var (w, h) = map.Dim();`
- Records for simple data types
- Collection expressions where appropriate
- Switch expressions for value-based branching
- `required` for mandatory properties

## Avoid

- Exceptions for control flow
- Over-abstraction and excessive granularity
- Unwarranted boilerplate (logging, DI, tests) unless requested

## When Explaining or Planning

- Lead with "why" before "how"
- Use concrete examples over abstractions
- Present options with trade-offs, not just descriptions
- Make plans actionable; steps to do, not to "consider"
- Start architecture explanations with brief summary
- Prefer diagrams (Mermaid) over prose for system interactions