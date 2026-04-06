# Prompt Engineering: The Anatomy of Effective AI Instructions

> **Backend II Masterclass: Vibecode with Safety**  
> Reference documentation extracted from Section 2

---

## What is a Prompt?

A **prompt** is the instruction you give to an LLM. It's the "input" that determines the "output."

But here's what most developers miss: **a prompt is not just the text you type into the chat box.** Behind the scenes, every interaction with an LLM follows a structured, role-based message format.

Understanding that structure is the difference between getting useful code and getting garbage.

### The Common Mistake: Treating Prompts Like Google Searches

Many developers treat prompts as if they're searching Google — short, vague, hoping the model reads their mind.

**The problem:**
- **Vague prompts produce vague code.** "Write me an API" produces useless output.
- **Specific prompts produce usable code.** "Write a FastAPI endpoint that accepts a JSON body with fields `name` (str, required) and `email` (str, validated), stores it in PostgreSQL via SQLAlchemy, and returns 201 with the created resource" produces something you can actually use.

---

## The Three-Role Message Structure

Every LLM conversation, behind the scenes, is a sequence of messages with roles:

```
System:    "You are a Python backend developer. You use FastAPI and SQLAlchemy."
User:      "Write a health check endpoint."
Assistant: "Here's a health check endpoint: ..."
User:      "Add a database connectivity check."
Assistant: "Updated version with DB check: ..."
```

### Role Breakdown

| Role       | Purpose                                        | Who writes it?              |
|------------|------------------------------------------------|-----------------------------|
| `system`   | Sets behavior, persona, constraints            | The developer / tool config |
| `user`     | The human's request or input                   | You                         |
| `assistant`| The model's response                           | The LLM                     |

### Understanding Each Role

#### 1. System Message
- **What it does:** Sets the model's behavior, persona, and constraints for the entire conversation
- **When it's set:** At the start of a conversation (usually invisible to you in chat UIs)
- **Who controls it:** You (via AGENTS.md/CLAUDE.md) or the tool's default configuration
- **Power level:** MAXIMUM — this is your most powerful lever

**Example system messages:**
```
"You are a senior Python backend engineer specializing in FastAPI and PostgreSQL."

"You are a code reviewer. Focus on security vulnerabilities and GDPR compliance."

"You are a database migration assistant. Never generate DDL that drops tables without explicit confirmation."
```

#### 2. User Message
- **What it does:** Represents your request or question
- **When it's sent:** Every time you type something in the chat
- **Who controls it:** You
- **Power level:** HIGH — but limited by the system message's constraints

#### 3. Assistant Message
- **What it does:** The model's response to your user message
- **When it appears:** After the model generates a response
- **Who controls it:** The LLM (but shaped by system + user messages)
- **Power level:** REACTIVE — follows the instructions from system and user roles

---

## The System Message: Your Most Powerful Lever

**The system message is the most powerful lever you have.**

It is where:
- **CLAUDE.md** or **AGENTS.md** content gets injected (project-wide instructions)
- **Coding standards** and **constraints** are defined
- **GDPR and PII safety rules** are enforced (e.g., "Never log user emails")

Think of the system message as the **permanent context** that shapes every response.

### Example: System Message in Action

**Without a system message:**
```
User: Write a user registration endpoint