---
name: langgraph-memory
description: Guidelines for understanding and implementing LangGraph state, threads, checkpoints, persistence, and long-term memory.
---

# LangGraph Memory Skill

Use this skill when working with state, memory, checkpoints, threads, persistence, or conversations in LangGraph and Deep Agents.

## Mental Model

Distinguish these concepts:

State
→ Data available during graph execution.

Checkpoint
→ A saved snapshot of graph state.

Checkpointer
→ The component responsible for saving and restoring checkpoints.

Thread ID
→ Identifies a particular execution/conversation.

Long-term memory
→ Information that should survive across different threads.

## State

Use state for information required by nodes during execution.

Example:

```python
class State(TypedDict):
    messages: list
    user_name: str