# HF AI Agents Course

I'm teaching agents to think, act, and observe — one unit at a time. This repo tracks my progress through the [Hugging Face AI Agents Course](https://huggingface.co/learn/agents-course/unit0/introduction), from first principles to a working agent.

## Progress

- [x] [Unit 1](unit1/) — Intro to Agents
- [ ] Unit 2 — Frameworks for AI Agents
  - [x] [smolagents](unit2/smolagents/)
  - [ ] LlamaIndex
  - [ ] LangGraph
- [ ] Unit 3 — Use Case for Agentic RAG
- [ ] Unit 4 — Final Project (create, test and certify the agent)

Each unit has its own folder with the code for that stage. This file is the running log of what I learned and built along the way.

---

## Unit 1 — Intro to Agents

Core idea: an agent is a system that uses an AI model to interact with its environment and carry out tasks — not just generate text. It runs on a loop:

**Think → Act → Observe**

- **Think**: model decides what to do
- **Act**: model outputs a structured tool call, code runs it
- **Observe**: the result gets added back into the context, loop continues or the agent answers

Also covered the basics of transformers/LLMs — next-token prediction, attention, why generation has to *stop* cleanly after a tool call so the model doesn't just make up a fake result while waiting.

**Project:** added a `get_weather` tool ([code](unit1/app.py)) so the agent can answer weather questions — calls the `wttr.in` API for real data.

**Takeaways:**

- The tool's docstring is what the model actually reads to decide when to use it
- Defining a tool and registering it in the agent's `tools=[...]` list are two separate steps — missing the second one means the model has no idea the tool exists, and it'll start guessing or making up tool names instead
- "Memory" is just the conversation getting longer, not some separate storage
- Stopping generation right after a tool call matters — otherwise the model will guess at results instead of waiting for real ones

---

## Unit 2 — Frameworks for AI Agents

Unit 2 covers three frameworks: smolagents, LlamaIndex and LangGraph. Progress so far: **smolagents done**, the other two are up next.

### smolagents

Core idea: a `CodeAgent` acts by **writing and running Python code** rather than only emitting structured tool calls. That's why it can answer `1 + 1` with no math tool, and why sandboxing matters: imports outside a safe list are blocked unless you allow them with `additional_authorized_imports`.

**Concepts covered:**

- **Tools**: the `@tool` decorator for simple functions, and the `Tool` class for tools that need setup (like loading a knowledge base)
- **RAG → Agentic RAG**: retrieve, augment, generate. RAG lets a model answer from fresh or private data instead of only its training. Agentic RAG lets the agent decide when and how often to retrieve, and can reformulate queries
- **Multi-agent systems**: a manager delegates to specialist agents (web search, retrieval, image generation). Each has its own memory, which keeps context small and costs down
- **Vision agents (VLMs)**: models that take images in and give text out, the opposite of image generation
- **Observability and evaluation**: OpenTelemetry traces sent to Langfuse, so I can see each step, tool call and token count. Agents are non-deterministic, so one good run proves little

**Project:** a Gradio agent deployed as a Hugging Face Space ([live Space](https://huggingface.co/spaces/mafuko254/First_agent_template), [code](unit2/smolagents/app.py)):

- Tools: `get_weather`, `get_current_time_in_timezone`, `suggest_menu`, DuckDuckGo web search, an image generation tool
- A custom retriever tool (`Tool` class) that searches a small knowledge base, so the agent answers from my own entries instead of guessing
- Langfuse tracing to inspect every run

**Takeaways:**

- Course code drifts from the current library. Unpinning `smolagents` fixed one crash but triggered a chain of others: a renamed model class, a removed `grammar` parameter, a new `ddgs` dependency, and a `prompts.yaml` missing a required `final_answer` template. **Pin a working version** in `requirements.txt`
- Template files like `Gradio_UI.py` can lag behind the library too. Image outputs came back wrapped in a `FinalAnswerStep` and didn't render in the UI
- The tool's description decides whether the agent picks it. Update it when the tool's contents change
- Set up the instrumentor before creating the agent, so tracing hooks in
- Test code (`agent.run(...)` demos) belongs in a notebook, not the deployed `app.py`, because it re-runs on every Space restart
- Secrets go in the Space's secrets manager, never in the code. A public Space also means my account absorbs the usage



## Running it

### In a Hugging Face Space

Set `HF_TOKEN` under Settings > Variables and Secrets, then the Space runs it for you.

For Unit 2, also add these under Settings > Variables and Secrets:

- `LANGFUSE_PUBLIC_KEY`
- `LANGFUSE_SECRET_KEY`
- `LANGFUSE_HOST`

`smolagents` is pinned to a specific version in `requirements.txt` (currently `[version]`). Traces show up in the Langfuse dashboard.

### Locally (e.g. in VS Code)

```bash
git clone https://github.com/YOUR-USERNAME/hf-ai-agents-course.git
cd hf-ai-agents-course
pip install -r requirements.txt
pip install python-dotenv
```

Create a `.env` file in the project folder (and make sure it's in `.gitignore` so it never gets committed):

```
HF_TOKEN=your_token_here
```

At the top of the unit's `app.py`:

```python
from dotenv import load_dotenv
load_dotenv()
```

Then run it from the VS Code integrated terminal, or just hit Run with `app.py` open:

```bash
python unit1/app.py
```

Token needs inference permission — grab one at [hf.co/settings/tokens](https://hf.co/settings/tokens).
