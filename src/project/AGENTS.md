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

## Invocation Pattern

```python
agent = create_deep_agent(
    model=model,
    backend=FilesystemBackend(root_dir="."),  # or StateBackend(), tools=[web_search]
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
