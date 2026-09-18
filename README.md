# HF AI Agents Course

I'm teaching agents to think, act, and observe — one unit at a time. This repo tracks my progress through the [Hugging Face AI Agents Course](https://huggingface.co/learn/agents-course/unit0/introduction), from first principles to a working agent.

## Progress

- [x] [Unit 1](unit1/) — Intro to Agents
- [ ] Unit 2 — Frameworks for AI Agents
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

## Running it

### In a Hugging Face Space

Set `HF_TOKEN` under Settings > Variables and Secrets, then the Space runs it for you.

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
