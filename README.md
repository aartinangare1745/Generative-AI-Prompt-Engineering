# LLM Engineering — Learning Projects

A hands-on, project-based path into LLM/AI engineering, built while learning core concepts from the ground up: API calls, conversation state, tool calling, and the reliability problems that come with working with real LLMs.

Each project builds on the last. Code is intentionally simple and heavily commented — the goal was understanding every line, not maximizing sophistication.

## Stack

- **Python**
- **[Groq](https://groq.com/)** — free, fast cloud API for open-source LLMs (Llama 3.3 70B), used instead of a paid API to keep this project cost-free while learning
- **Google Colab** — zero local setup, runs entirely in the cloud

## Setup

1. Sign up at [console.groq.com](https://console.groq.com) (free, no credit card required) and create an API key
2. Install the client library:
   ```bash
   pip install groq
   ```
3. Set your API key as an environment variable:
   ```bash
   export GROQ_API_KEY="your-key-here"
   ```
   (In Colab, use the Secrets manager instead — see comments in each script.)
4. Run any project:
   ```bash
   python project0_hello_llm_groq.py
   ```

## Projects

### `project0_hello_llm_groq.py` — First API Call
The smallest possible LLM program: send one message, print the response. Covers the basic request/response shape shared by every major LLM provider's API (system prompt, user message, reading the response object).

### `project1_chatbot_loop.py` — Conversation Memory
Turns the single-shot call into a real back-and-forth chatbot. Demonstrates that LLMs are stateless — "memory" only works because the full conversation history is resent to the API on every single turn.

### `project2_tool_calling.py` — Tool Calling
Gives the model access to a real Python function (a calculator) instead of letting it guess at math. This is the foundational mechanism behind every agent framework: the model requests an action, real code executes it, the result is returned to the model.

## Problems Hit & Fixes (the actual learning)

**Malformed tool calls (`tool_use_failed` errors).** Tool calling requires the model to generate an exact structured format, and this generation is still probabilistic — it can occasionally break. Fixed by setting `temperature=0` (more deterministic token selection) and adding a simple retry wrapper around the API call, a standard pattern for handling transient API/model failures.

**Model contradicting its own correct tool output.** After the calculator tool returned a correct result, the model's follow-up explanation re-did the math *itself*, got it wrong, and told the user the correct answer was wrong. Root cause: the tool's result and the model's explanatory text are generated separately — the model doesn't treat a tool's return value as a locked, trusted fact by default. Real fix: minimize how much exact/numeric output is routed back through free-form generation at all; insert deterministic results directly rather than asking the model to restate them.

This turned into the core lesson of the whole project: **use the LLM for language and judgment, and hand off anything requiring guaranteed correctness to real, deterministic code.**

## What's Next

- RAG (Retrieval-Augmented Generation) — feeding external documents into context for grounded answers
- Multi-tool / multi-step agents
- Basic evals — a small test suite of known question → correct-answer pairs to catch regressions

## Notes

Full concept notes and glossary (tokens, context windows, temperature, RAG, agents, evals, etc.) are in [`llm_fundamentals_glossary.md`](./llm_fundamentals_glossary.md).
