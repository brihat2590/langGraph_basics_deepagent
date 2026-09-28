# Deep Agent

Context reference for the "deep agent" implemented in [deepagent.ipynb](deepagent.ipynb). Built with the `deepagents` package (`create_deep_agent`) on top of `langgraph`/`langchain`.

## Model

- LLM: `openai/gpt-oss-20b`
- Provider: Groq, loaded via `langchain.chat_models.init_chat_model(model, model_provider="groq")`
- API key: `GROQ_API_KEY` (from `.env`, loaded with `python-dotenv`)

## System Prompt

`"Act as a research agent"`

## Tools

- `web_search(query: str, retries: int = 5, topic: Literal["general", "sports", "news"] = "general")`
  - Wraps `tavily.TavilyClient.search`
  - Requires `TAVILY_API_KEY` (from `.env`)
  - Returns raw Tavily search results for the agent to reason over

## Backends

The deep agent's virtual file system is provided by a `deepagents.backends` backend, swappable per run:

- `StateBackend()` — in-memory, scoped to the LangGraph run state; file writes do **not** persist to disk.
- `FilesystemBackend(root_dir=".")` — real file system access rooted at `src/project`; file writes persist to disk (e.g. it created [notes/todo.txt](notes/todo.txt)).

## Skills

Reusable capability packs live under [skills/](skills/), one directory per skill:

```
skills/
├── deep-agent-dev/
│   └── SKILL.md      # name: deep-agent-dev
└── langgraph-memory/
    └── SKILL.md      # name: langgraph-memory
```

Requirements for `SkillsMiddleware` (added automatically by `create_deep_agent` when
`skills=` is provided) to recognize a skill:

- Each skill is a **directory** containing a `SKILL.md` file with YAML frontmatter
  (`name`, `description`).
- The frontmatter `name` **must exactly match the parent directory name** — a
  mismatch (e.g. `name: deep-agent-development` in `skills/deep-agent-dev/`) is a
  spec violation and causes the skill to be skipped with a warning.
- `skills=` must point to the **parent** directory that contains the skill
  subdirectories (e.g. `"skills"`), not to each individual skill folder — the
  middleware scans one level down for `SKILL.md` files. Passing the individual
  skill paths (e.g. `"skills/deep-agent-dev"`) silently finds nothing.
- Paths are POSIX-style (forward slashes) and relative to the backend's root.
  With `FilesystemBackend(root_dir=".")` rooted at `src/project`, the correct
  value is `skills=["skills"]`.
- `StateBackend()` cannot read skills from disk; skill file contents must be
  supplied via `invoke(files={...})` instead.

## Invocation Pattern

```python
agent = create_deep_agent(
    model=model,
    backend=FilesystemBackend(root_dir="."),  # or StateBackend(), tools=[web_search]
    skills=["skills"],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "<instruction>"}]
})

result["messages"][-1].content  # final agent reply
```

## Capabilities

- Can research the web via the `web_search` tool (when supplied) and cite/summarize findings.
- Can create, read, and edit files through its backend's virtual file system tools (e.g. `write_file`, `read_file`, `ls`), scoped to whichever backend is configured.
- Runs as a compiled LangGraph state graph (`CompiledStateGraph`), so it supports the standard LangGraph invoke/stream interface.

## Environment Requirements

- `.env` file with `GROQ_API_KEY` and `TAVILY_API_KEY`
- Dependencies (see [pyproject.toml](../../pyproject.toml)): `deepagents>=0.7.19`, `langchain-groq>=1.1.3`, `langgraph>=1.2.11`, `dotenv>=0.9.9`

## Tracing

All LangChain/LangGraph runs (including the deep agent, subgraphs, and tool calls) are traced automatically to [LangSmith](https://smith.langchain.com/) once these `.env` vars are set — no code changes needed beyond calling `load_dotenv()` before building a graph or agent:

- `LANGSMITH_TRACING=true`
- `LANGSMITH_API_KEY`
- `LANGSMITH_PROJECT` (optional, defaults to `"default"`)
