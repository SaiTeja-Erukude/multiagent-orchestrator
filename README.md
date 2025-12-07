# 🎛️ Multi-Agent Orchestrator

A small **planning & coordination** model that acts as a _conductor_ for your AI **agents** and **tools**.

This model is **not** a general chatbot.  
It's job is to look at the current task + available agents/tools and decide the **next action** as strict JSON.

---

## 🎯 What it does

`multiagent-orchestrator`:

- 🗂️ Reads a **task state** (goal, history, constraints)
- 👥 Reads an **agent registry** (who can do what, cost/latency)
- 🛠️ Optionally sees available **tools** (APIs, DB, FS, etc.)
- 🎬 Outputs exactly one **next action**:

```json
{
  "action": "call_agent" | "call_tool" | "ask_user" | "finish",
  "target": "agent_or_tool_name_or_null",
  "arguments": { "any": "json" },
  "final_answer": "string or null",
  "reason": "short natural language rationale"
}
```

You run your **own runtime/loop** that:

1. 🔁 Calls this model to get the next action
2. 🧰 Executes the requested agent/tool
3. 📝 Updates the task state
4. 🔄 Calls the model again, until `action == "finish"` ✅

---

## 📌 When to use it
Use `multiagent-orchestrator` when you have:
- 🤖 Multiple LLM agents (researcher, coder, critic, etc.)
- 🛠️ Multiple tools (search, DB, filesystem, APIs)
- 🧭 And you want a central brain that:
    - 🧩 Plans multi-step workflows
    - 🎯 Delegates to the right worker
    - ⚖️ Stays cost/latency aware

It’s designed to be framework-agnostic (works with LangGraph, AutoGen-style setups, custom orchestrators, etc.) as long as you follow the JSON contracts.

---

## 🧾 Basic prompt format
You can embed your agent registry and task state into a single prompt.

**System prompt (recommended):**

```text
You are ORCHESTRATOR, a supervisor coordinating a team of AI agents and tools.
You NEVER solve the task directly. You ONLY decide the next action.

You must ALWAYS respond with a single JSON object using this schema:

{
  "action": "call_agent" | "call_tool" | "ask_user" | "finish",
  "target": string or null,
  "arguments": object,
  "final_answer": string or null,
  "reason": string
}

- Use "call_agent" to delegate work to a specialized agent.
- Use "call_tool" for non-LLM tools (APIs, DB, filesystem, etc.).
- Use "ask_user" if required information is missing.
- Use "finish" only when the overall task is completed.

Prefer cheaper/faster agents when possible.
Never output anything that is not valid JSON.
```

**User content example:**
```text
Available agents:
{
  "agents": [
    {
      "name": "researcher",
      "role": "Collects and summarizes information from the web and docs.",
      "cost": "medium",
      "latency": "medium"
    },
    {
      "name": "coder",
      "role": "Writes and fixes code and runs tests.",
      "cost": "high",
      "latency": "high"
    }
  ]
}

Current task state:
{
  "task": "Build a short technical blog post about multi-agent LLM systems.",
  "status": "in_progress",
  "step": 1,
  "max_steps": 8,
  "history": []
}

Decide the next action as JSON only.
```

---

**🐍 Example: using with the Ollama Python client**
```python
import json
import ollama

def call_orchestrator(agents, state):
    system = """You are ORCHESTRATOR, a supervisor coordinating a team of AI agents and tools.
You NEVER solve the task directly. You ONLY decide the next action.
Always respond with a single JSON object:
{"action": "...", "target": "...", "arguments": {...}, "final_answer": null or string, "reason": "..."}"""

    user = f"""Available agents:
{json.dumps(agents, ensure_ascii=False)}

Current task state:
{json.dumps(state, ensure_ascii=False)}

Decide the next action as JSON only.
"""

    res = ollama.chat(
        model='multiagent-orchestrator:latest',
        messages=[
            {"role": "system", "content": system},
            {"role": "user", "content": user},
        ],
    )

    action_text = res["message"]["content"].strip()
    return json.loads(action_text)
```

You then plug `call_orchestrator()` into your own loop that actually calls agents/tools and updates `state`.

---


## 👤 Author

**Author:** Sai Teja Erukude  
**Role:** Developer & Maintainer of `multiagent-orchestrator`  

---

**📝 Notes**

- 🚦 This model is optimized for planning & routing, not long-form generation.
- 🛡️ Always parse and validate the JSON output before executing any real tools.
- 🔧 You can extend the schema(e.g., add cost_estimate, confidence) as long as you keep it consistent in your prompts and runtime.


Happy orchestrating! 🤖