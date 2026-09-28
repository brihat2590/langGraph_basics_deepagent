---
name: deep-agent-dev
description: Guidelines for building and extending Deep Agents using models, tools, skills, subagents, backends, and middleware.
---

# Deep Agent Development Skill

Use this skill when building, modifying, or explaining Deep Agents.

## Core Principles

- Keep the agent architecture simple unless complexity is required.
- Prefer existing Deep Agents functionality before implementing custom behavior.
- Separate agent instructions, tools, skills, state, and persistence.
- Do not put large amounts of specialized knowledge directly into the system prompt.
- Use skills for specialized instructions that should be loaded when relevant.

## Architecture

Think about a Deep Agent as:

Model
→ Agent
→ Middleware
→ Tools
→ Backend
→ State
→ Checkpointer

Each component should have a clear responsibility.

## Skills

Use `SKILL.md` files for specialized capabilities.

Prefer progressive disclosure:

1. Expose the skill name and description.
2. Load the full skill instructions only when the skill is relevant.
3. Follow the loaded instructions while solving the task.

## Tools

Before creating a new tool:

1. Check whether an existing tool can solve the task.
2. Define a clear tool name and description.
3. Keep the tool's input schema explicit.
4. Validate important inputs.
5. Handle tool errors clearly.

## Subagents

Use subagents when a task can be isolated into a specialized workflow.

A subagent should have:

- A clear purpose.
- A focused system prompt.
- Only the tools it actually needs.
- Relevant skills when necessary.

Avoid creating subagents for simple tasks that the main agent can handle directly.

## Code Quality

When modifying agent code:

- Prefer small functions.
- Use type hints.
- Keep configuration separate from business logic.
- Avoid unnecessary abstractions.
- Explain important architectural decisions.