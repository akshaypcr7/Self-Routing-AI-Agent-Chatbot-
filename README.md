# LangGraph Chatbot With Tools

A tool-augmented conversational agent built with LangGraph. Instead of relying only on trained knowledge, the agent decides at runtime whether to call an external tool, fetches the data, and uses it to form its response.

---

## How It Works

The agent is modeled as a state graph with two nodes and a conditional routing edge:

```
START -> chatbot -> (tools -> chatbot)* -> END
```

1. **State** — A `TypedDict` holds a `messages` list annotated with `add_messages`, LangGraph's reducer that appends new messages instead of overwriting. Conversation history accumulates across turns.

2. **Tool binding** — Wikipedia and Arxiv are wrapped as LangChain tools and bound to the LLM with `llm.bind_tools(tools)`. The model receives their schemas on every request so it knows when and how to call them.

3. **Nodes**
   - `chatbot` — invokes the tool-aware LLM
   - `tools` — a prebuilt `ToolNode` that executes whichever tool the LLM requested

4. **Edges**
   - `START -> chatbot` (entry point)
   - `chatbot -> conditional` — `tools_condition` inspects the last message; if there is a tool call, route to `tools`, otherwise `END`
   - `tools -> chatbot` — feeds tool output back so the model can form a final answer

5. **Execution** — `graph.stream()` walks the state machine and yields events at each step: user message -> tool call -> tool result -> final answer.

---

## Stack

| Component | Library |
|---|---|
| Agent orchestration | `langgraph` |
| LLM inference | `langchain_groq` |
| Tools | `langchain_community` (Wikipedia, Arxiv) |
| Tracing | `langsmith` |
| Language | Python 3.10+ |

---

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/your-username/langgraph-chatbot-with-tools.git
cd langgraph-chatbot-with-tools
```

**2. Install dependencies**
```bash
pip install langgraph langsmith langchain langchain_groq langchain_community arxiv wikipedia python-dotenv
```

**3. Add your API key**

Create a `.env` file in the root directory:
```
GROQ_API_KEY=your_groq_api_key_here
```
Get a free key at [console.groq.com](https://console.groq.com).

**4. Run the notebook**

Open `Langgraph_Chatbot_With_Tools.ipynb` in VS Code or Jupyter and run all cells.

---

## Model

The notebook uses `openai/gpt-oss-20b` via the Groq API. This model supports structured tool calling reliably. You can swap it for `llama-3.3-70b-versatile` or any other Groq-supported model that supports tool use.

```python
llm = ChatGroq(groq_api_key=groq_api_key, model_name="openai/gpt-oss-20b")
```

> Note: `Gemma2-9b-It` (used in the original Colab version) is deprecated on Groq and does not support tool calling reliably.

---

## Project Structure

```
langgraph-chatbot-with-tools/
├── Langgraph_Chatbot_With_Tools.ipynb
├── .env                  # not committed
├── .gitignore
└── README.md
```

---

## What's Next

- Persistent memory using LangGraph checkpointers
- Human-in-the-loop interrupts
- Swapping Wikipedia/Arxiv for a custom tool

---

## License

MIT
