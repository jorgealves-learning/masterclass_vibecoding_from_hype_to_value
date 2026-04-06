# Context Window: Understanding LLM Working Memory

> **Backend II Masterclass: Vibecode with Safety**  
> Reference documentation extracted from Section 2

---

## What is a Context Window?

The **context window** is the total amount of text (measured in **tokens**) that an LLM can "see" at once. Think of it as the model's **working memory** for a conversation.

Every message you send, every response the model generates, and every instruction in the system prompt consumes tokens from this window. When the window fills up, something has to give.

### The Computer Memory Analogy

If you've worked with computer memory (RAM), the context window is similar:

| Concept             | Computer RAM                       | LLM Context Window                |
|---------------------|------------------------------------|------------------------------------|
| **What it stores**  | Running programs and data          | Conversation history and context   |
| **Limit**           | Physical limit (e.g., 16 GB)       | Token limit (e.g., 128K tokens)    |
| **What happens when full** | System slows down or crashes | Oldest messages get dropped        |
| **Cost factor**     | More RAM = more expensive          | More tokens = higher API costs     |

Just like you can't load a 20 GB game into 16 GB of RAM, you can't fit a 200,000-token conversation into a 128,000-token context window.

---

## Visual Representation

```
+----------------------------------------------------------+
|                   CONTEXT WINDOW                          |
|                                                          |
|  [System message]                                        |
|  [User message 1]                                        |
|  [Assistant response 1]                                  |
|  [User message 2]                                        |
|  [Assistant response 2]  <-- we are here                 |
|  [... next token prediction happens here ...]            |
|                                                          |
|  Total: all of the above must fit within the limit       |
+----------------------------------------------------------+
```

---

## Current Context Window Sizes (April 2026)

| Model          | Context Window  | Roughly equivalent to...          |
|----------------|-----------------|-----------------------------------|
| GPT-4o         | 128K tokens     | ~300 pages of text                |
| Claude Sonnet  | 200K tokens     | ~500 pages of text                |
| Claude Opus    | 200K tokens     | ~500 pages of text                |
| Gemini 2.5     | 1M tokens       | ~2,500 pages of text              |

**Note:** One token ≈ 0.75 English words on average. The exact conversion depends on the tokenizer and language.

### What is a Token?

A **token** is the basic unit of text that the model processes. It's **not** the same as a word.

**Examples:**

| Text                  | Tokens | Why?                                          |
|-----------------------|--------|-----------------------------------------------|
| `hello`               | 1      | Common word, single token                     |
| `FastAPI`             | 2      | `Fast` + `API` (mixed case splits)            |
| `user_id`             | 2      | `user` + `_id` (underscore splits)            |
| `@app.get("/users")`  | 7      | Special characters each count as tokens       |
| `{'name': 'João'}`    | 6      | JSON structure + special chars + accented char|

**Key insight:** Code uses **more tokens** than plain English because of symbols, indentation, brackets, and mixed-case naming.

Try it yourself: https://platform.openai.com/tokenizer

---

## What Happens When You Exceed the Limit?

When the context window is full:

1. **Oldest messages get dropped** (silently, without warning)
2. **Or the API request fails entirely** (depending on the provider and configuration)
3. **The model "forgets" earlier parts of the conversation** — leading to inconsistent or degraded output

**The model does NOT warn you.** It just loses context silently.

### Real-World Example: The Debugging Disaster

Imagine this conversation:

```
Turn 1 (You):      Here's my FastAPI app. I need help debugging auth.
                   [pastes 5,000 tokens of code]
                   
Turn 2 (Model):    I see the issue. Your JWT secret is hardcoded. Fix that.

Turn 3 (You):      Thanks! Now can you add rate limiting?

Turn 4 (Model):    Sure, here's a rate limiter using Redis...

Turn 5 (You):      Great! Now add logging to the auth endpoint.

Turn 6 (Model):    Here's the logging code...
                   [but the model has FORGOTTEN the earlier JWT fix]
                   [it re-introduces the hardcoded secret because Turn 1 was dropped]
```

**What happened?** The conversation exceeded the context window between Turn 1 and Turn 6. The model dropped the earliest messages and lost critical context about the JWT fix.

**How to avoid this:**
- Start a new conversation for each major task
- Or ask the model to summarize the current state before continuing

---

## Why Context Windows Matter

### 1. Quality Degradation
When context overflows, the model loses track of:
- Earlier instructions you gave
- Code you shared previously
- Constraints you specified (like "don't log PII")

Result: The output becomes inconsistent or ignores your requirements.

### 2. Cost Implications
Most API providers charge **per token** for both input and output. A conversation that repeatedly exceeds the context window wastes money on:
- Tokens that get dropped
- Re-sending context that was lost
- Inefficient back-and-forth to re-establish context

### 3. Workflow Friction
Developers who don't understand context windows:
- Paste entire codebases into chat and wonder why output degrades
- Lose track of multi-turn conversations
- Waste time repeating themselves

---

## Strategies for Working Within Context Limits

### 1. Use Focused Prompts
**Bad:**
```
Here's my entire 10,000-line codebase. Find the bug.
```

**Good:**
```
I have a bug in the user authentication flow. Here's the relevant 
function (50 lines). The issue is that tokens expire immediately 
after being issued. Here's the token generation code...
```

### 2. Use Project Configuration Files
Instead of repeating instructions in every conversation, use:
- **AGENTS.md** / **CLAUDE.md** for project-wide system prompts
- **SPEC.md** for requirements and architecture summaries
- **Context files** that tools like Cursor or OpenCode can read automatically

This way, you provide context **once** in structured files, not repeatedly in chat.

### 3. Summarize Long Conversations
If a conversation gets long:
- Ask the model to **summarize the key decisions and context**
- Start a **new conversation** with that summary as the opening message
- This "resets" the context window while preserving important information

### 4. Use Context-Aware Tools
Modern AI coding assistants (Cursor, GitHub Copilot Workspace, OpenCode) intelligently manage context by:
- Only loading **relevant files** based on your current task
- Using **semantic search** to find pertinent code, not brute-force pasting
- Automatically injecting project configuration files into the system prompt

### 5. Break Large Tasks into Smaller Steps
Instead of:
```
Refactor my entire backend to use async/await, add caching, 
fix all type hints, and migrate from SQLAlchemy 1.4 to 2.0
```

Do:
```
Step 1: Convert the user service to async/await
Step 2: Add Redis caching to the user service
Step 3: [in a new conversation] Migrate user service to SQLAlchemy 2.0
```

---

## Tokens and Pricing: A Quick Guide

### How Tokens are Counted

| Text Type                  | Approximate Token Count       |
|----------------------------|-------------------------------|
| 1 word                     | ~1.3 tokens (English)         |
| 1 line of Python code      | ~10-20 tokens                 |
| 1 page of text (500 words) | ~650 tokens                   |
| 1 typical API response     | 200-1,000 tokens              |

**Code uses more tokens than prose** because of symbols, indentation, and mixed case.

### Cost Example (Hypothetical Pricing)

If a model charges $0.01 per 1,000 input tokens:

- A 50,000-token conversation (like pasting 100 pages of docs) costs **$0.50** per request
- A focused 5,000-token conversation costs **$0.05** per request

Over 100 API calls, the difference is **$50 vs $5**.

**Token awareness prevents wasted money.**

---

## Context Window and GDPR/PII Safety

Larger context windows can be a **privacy risk** if not managed carefully:

- **Risk:** Developers paste entire database dumps, logs with PII, or customer data into chat to "give the model context"
- **Problem:** That data is now sent to a third-party API and may be retained for training or logging (depending on the provider's terms)

### Safe Practices

1. **Never paste full logs** — redact PII first (use `/dev/usernames`, `/tmp/emails`, etc.)
2. **Never paste production database records** — use synthetic data
3. **Use context-aware tools** that let you control what gets sent to the API
4. **Check your API provider's data retention policy** — some providers do NOT use your data for training (e.g., OpenAI's API with proper settings, Anthropic's Claude API)

See `MODELS.md` for privacy and GDPR considerations by provider.

---

## Decision Matrix: When to Worry About Context Windows

| Scenario                                  | Context Window Concern? | Strategy                          |
|-------------------------------------------|-------------------------|-----------------------------------|
| One-off question about a 20-line function | No                      | Just paste the function           |
| Debugging a bug across 5 files            | Moderate                | Share only relevant functions     |
| Refactoring an entire microservice        | High                    | Break into multi-step tasks       |
| Long conversation (20+ turns)             | High                    | Summarize and start fresh         |
| Pasting docs + code + logs all at once    | Critical                | Use project files (AGENTS.md)     |

---

## Further Reading

- OpenAI Tokenizer (to see how text gets split): https://platform.openai.com/tokenizer
- Anthropic Context Windows: https://docs.anthropic.com/claude/docs/models-overview
- Google Gemini Context: https://ai.google.dev/gemini-api/docs/models/gemini
