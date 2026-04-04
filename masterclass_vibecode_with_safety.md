---
format: Co-hosted Session
hosts: Jorge Alves (Tech Instructor, Backend II) / Saulo Chanoca (Legal Expert, Sonae Sierra)
audience: Backend II students -- intermediate Python developers familiar with web frameworks
duration: 3 hours (with 10-minute break)
topic_type: Technical + Legal + Security
---

# Vibecode with Safety: AI-Assisted Development with GDPR & PII Awareness

## Session Agenda

| Section                                      |
|----------------------------------------------|
| 1. What is an LLM? Which one to choose?      |
| 2. Prompts and Context Windows               |
| 3. What is AGENTS.md?                        |
| 4. What is SKILL.md?                         |
| 5. What is ARCHITECTURE.md?                  |
| **10 min. BREAK**                            |
| 6. What is SPEC.md?                          |
| 7. Subagents: How to use them                |
| 8. GDPR and PII                              |
| 9. Vibecoding: How to start safely           |

---

# PART 1 -- Technical Foundations

---

## 1. What is an LLM? Which one to choose?

### Learning Objectives
- Understand what a Large Language Model is and how it generates text
- Know the major LLM providers and their tradeoffs
- Be able to choose an LLM based on task requirements, cost, and privacy

---

### 1.1 Context of the Problem

Welcome, everyone. Over the next three hours, we are going to change the way you think about writing code. Not because the tools are new -- you have probably all used ChatGPT or Copilot by now -- but because most developers use these tools without understanding what they are, what they can see, or what happens to the data they send.

You build APIs. You handle user data. You deploy services. That means AI-assisted development is not just a productivity question for you -- it is a safety question.

Let us start with the foundation: what is actually running when you type a prompt and get code back?

### 1.2 The Problem

Most developers treat LLMs as magic autocomplete. They do not understand:
- Why the model sometimes "forgets" things mid-conversation
- Why the same prompt gives different results on different models
- Why some models are free and others cost $20/month (and what you pay with when it is "free")
- Whether their code and data are being used for training

This lack of understanding leads to poor model choices, wasted money, leaked data, and false confidence in generated code.

### 1.3 How to Possibly Solve It

You could read every research paper on transformer architecture. You could memorize benchmarks. But for a working developer, what you actually need is a practical mental model.

### 1.4 The Solution

**An LLM is a statistical text prediction engine trained on massive datasets.**

Here is what that means concretely:

**How it works (the 60-second version):**
1. Text goes in as **tokens** (roughly word-pieces: "unbelievable" = "un" + "believ" + "able")
2. The model predicts the **next most probable token** given all previous tokens
3. It repeats this process token-by-token until it generates a complete response
4. The model has **no memory** between conversations -- it sees only what you send it right now

**The key implication:** An LLM does not "know" things. It has learned statistical patterns from text. It can produce confident, well-structured nonsense. This is called a **hallucination**, and it is not a bug -- it is a fundamental property of how these systems work.

**The current landscape (April 2026):**

| Model              | Provider    | Strengths                                    | Privacy Model                          | Cost Tier  |
|--------------------|-------------|----------------------------------------------|----------------------------------------|------------|
| GPT-4o/4.1         | OpenAI      | General-purpose, widely available            | Data may train models (opt-out exists) | $$         |
| Claude Opus/Sonnet | Anthropic   | Strong reasoning, long context, safety focus | No training on user data (API/Pro)     | $$-$$$     |
| Gemini 2.5         | Google      | Multimodal, large context                    | Integrated with Google ecosystem       | $$         |
| Mistral Large      | Mistral AI  | European-hosted, open weights available      | EU-based data processing               | $$         |
| Llama 3/4          | Meta        | Open-source, self-hostable                   | Full control if self-hosted            | Free-$$$   |
| Gemma 3/4          | Google      | Self-hostable                                | Self-hosted, private                   | Free       |
 
Notice that column -- "Privacy Model." We will come back to this in a big way after the break when our guest joins us for the GDPR section. For now, remember: **where your prompts go and what happens to them is a technical decision with legal consequences.**

### 1.5 Why This Matters

- **Wrong model choice = leaked data.** If you paste production database schemas into a free-tier ChatGPT, that data may be used for training.
- **Wrong model choice = wasted budget.** Using GPT-4 for simple code formatting is like renting a crane to hang a picture frame.
- **Wrong model choice = poor results.** Each model has different strengths. Claude excels at long-document reasoning. GPT-4o is fast for short tasks. Llama gives you full control but requires infrastructure.

### 1.6 When to Choose What

| Situation                                | Recommended Approach                                                                         |
|------------------------------------------|----------------------------------------------------------------------------------------------|
| Prototyping, learning, personal projects | Any model -- GPT-4o, Claude Sonnet are solid defaults                                        |
| Working with client/user data            | Self-hosted (Llama) or API with no-training guarantees (Claude API, OpenAI API with opt-out) |
| European regulatory requirements         | Mistral (EU-hosted) or self-hosted Llama                                                     |
| Long codebases, complex reasoning        | Claude (200K+ context), Gemini 2.5 (1M context)                                              |
| Budget-constrained team                  | Llama (self-hosted) or smaller models via Ollama                                             |

### 1.7 When NOT to Use an LLM

- **For security-critical logic** (authentication, encryption, access control) without expert human review
- **For generating legal or medical text** you plan to use verbatim
- **As a replacement for understanding** -- if you cannot review the output, you should not ship it
- **When deterministic output is required** -- LLMs are NON deterministic; same input can produce different output

### Exercise 1 -- Model Selection

> **Scenario:** Your team is building a health insurance API for a Portuguese client. The API processes user names, tax IDs (NIF), and medical claim descriptions. You need an LLM to help generate boilerplate CRUD endpoints and write tests.
>
> **Question:** Which model(s) would you consider? Which would you rule out immediately? Why?
>
> *Discuss with a neighbor for 2 minutes.*

---

## 2. What is a Prompt? What is a Context Window?

### Learning Objectives
- Understand the anatomy of a prompt and the role-based message structure
- Know what a context window is and why token limits shape your workflow
- Apply practical strategies for working within context limits

---

### 2.1 Context of the Problem

You just learned that an LLM predicts text based on what you send it. The natural next question: what exactly are you sending it, and how much can it see?

Every interaction with an LLM has a structure, even when the chat interface hides it. Understanding that structure is the difference between getting useful code and getting garbage.

### 2.2 The Problem

Developers treat prompts as Google searches -- short, vague, hoping the model reads their mind. They hit two walls:

1. **Quality wall:** Vague prompts produce vague code. "Write me an API" produces useless output. "Write a FastAPI endpoint that accepts a JSON body with fields `name` (str, required) and `email` (str, validated), stores it in PostgreSQL via SQLAlchemy, and returns 201 with the created resource" produces something you can actually use.

2. **Context wall:** They paste an entire codebase into the chat, the model starts "forgetting" earlier parts, and the output degrades. They do not understand why.

### 2.3 How to Possibly Solve It

The solution is structural. Learn the three-role message format, understand token limits, and develop strategies for working within them.

### 2.4 The Solution

**The Three-Role Message Structure:**

Every LLM conversation, behind the scenes, is a sequence of messages with roles:

```
System:    "You are a Python backend developer. You use FastAPI and SQLAlchemy."
User:      "Write a health check endpoint."
Assistant: "Here's a health check endpoint: ..."
User:      "Add a database connectivity check."
Assistant: "Updated version with DB check: ..."
```

| Role       | Purpose                                        | Who writes it?              |
|------------|------------------------------------------------|-----------------------------|
| `system`   | Sets behavior, persona, constraints            | The developer / tool config |
| `user`     | The human's request or input                   | You                         |
| `assistant`| The model's response                           | The LLM                     |

**The system message is the most powerful lever you have.** It is where CLAUDE.md, AGENTS.md, and SPEC.md content gets injected. We will cover those shortly.

**The Context Window:**

The context window is the total amount of text (measured in tokens) that the model can "see" at once. Think of it as the model's working memory.

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

**Current context window sizes (April 2026):**

| Model          | Context Window  | Roughly equivalent to...          |
|----------------|-----------------|-----------------------------------|
| GPT-4o         | 128K tokens     | ~300 pages of text                |
| Claude Sonnet  | 200K tokens     | ~500 pages of text                |
| Claude Opus    | 200K tokens     | ~500 pages of text                |
| Gemini 2.5     | 1M tokens       | ~2,500 pages of text              |

**What happens when you exceed it?** The oldest messages get dropped (or the request fails). The model does not warn you -- it just loses context silently.

### 2.5 Why This Matters

- A well-structured prompt with a clear system message can turn a mediocre model into a highly effective coding partner
- Understanding the context window explains why the model "forgets" things in long conversations
- Token awareness prevents wasted API calls and unexpected costs (you pay per token on most APIs)

### 2.6 When to Invest in Prompt Engineering

- When you are going to reuse the same prompt pattern repeatedly (automate it)
- When the task is complex and the default output is poor
- When you are setting up project-level configurations (CLAUDE.md, system prompts)
- When working with sensitive data and you need the model to follow strict rules

### 2.7 When NOT to Invest in Prompt Engineering

- For one-off throwaway questions -- just ask naturally
- When the model already gives good output on the first try -- do not over-engineer
- As a substitute for good code review -- no prompt makes an LLM infallible

### Practical Example: The Anatomy of a Good Prompt

**Bad prompt:**
```
Make a user API
```

**Good prompt:**
```
Create a FastAPI router for user management with the following:

- POST /users: accepts JSON with name (str, required), email (str, email format),
  and role (enum: "admin", "viewer", default "viewer"). Returns 201 with created user.
- GET /users/{id}: returns user by UUID. Returns 404 if not found.
- Use SQLAlchemy async with a PostgreSQL backend.
- Use Pydantic v2 models for request/response validation.
- Include docstrings on each endpoint.
- Do NOT log or print any user PII (name, email) -- use user ID only in logs.
```

Notice that last line. Even in a prompt, you are making safety decisions. We are planting seeds now that will fully bloom in the GDPR section.

### Exercise 2 -- Prompt Rewriting

> **Task:** Rewrite this bad prompt into a good one:
>
> *"Write me a database migration script"*
>
> Your improved prompt should specify: the ORM, the database, the table(s), the columns with types, any constraints, and any safety requirements (e.g., "do not include example data with real names").
>
> *3 minutes. Write it down. We will share a few.*

---

## 3. What is AGENTS.md? What is its Purpose?


### Learning Objectives
- Understand the role of AGENTS.md in AI-assisted development workflows
- Know the standard structure and conventions of the file
- Be able to write an effective AGENTS.md for a backend project

---

### 3.1 Context of the Problem

You now know that system messages shape LLM behavior. But typing a system message every time you open a chat is tedious and error-prone. What if your whole project could have a persistent system message that every AI tool reads automatically?

That is exactly what AGENTS.md (and its predecessor/sibling CLAUDE.md) does.

### 3.2 The Problem

Without project-level AI configuration:
- Every developer on the team prompts the LLM differently
- The model does not know your tech stack, conventions, or constraints
- Safety rules (like "never log PII") must be remembered and typed every time
- Context is wasted on repetitive instructions instead of the actual task

### 3.3 The Solution

**AGENTS.md** is a Markdown file placed in the root of your repository. AI-assisted development tools (Claude Code, GitHub Copilot, Cursor, etc.) read it automatically and inject its contents into the system prompt.

**Standard structure:**

```markdown
# AGENTS.md

## Project Overview
Brief description of what this project does.

## Tech Stack
- Language: Python 3.12
- Framework: FastAPI
- ORM: SQLAlchemy 2.0 (async)
- Database: PostgreSQL 15
- Testing: pytest + httpx

## Conventions
- Use type hints on all function signatures
- Async by default for I/O operations
- Pydantic v2 for all data validation

## Safety Rules
- NEVER include real PII in test fixtures or examples
- NEVER log user-identifiable information (names, emails, IPs)
- All database queries must use parameterized statements
- Secrets must come from environment variables, never hardcoded

## File Structure
- src/api/     -- route handlers
- src/models/  -- SQLAlchemy models
- src/schemas/ -- Pydantic schemas
- tests/       -- test files mirror src/ structure
```

**How tools use it:**

| Tool        | How it reads AGENTS.md / CLAUDE.md                       |
|-------------|----------------------------------------------------------|
| Claude Code | Reads CLAUDE.md automatically from project root          |
| Cursor      | Reads .cursorrules or AGENTS.md                          |
| GitHub Copilot | Reads .github/copilot-instructions.md or AGENTS.md    |
| Aider       | Reads CONVENTIONS.md or configured files                 |

The naming varies by tool, but the concept is identical. For Claude Code specifically, the file is called `CLAUDE.md`. The community-standard cross-tool file is `AGENTS.md`. Many teams maintain both.

### 3.4 Why Use It

- **Consistency:** Every AI interaction starts with the same project context
- **Safety:** Rules like "never log PII" are enforced structurally, not by memory
- **Efficiency:** You stop wasting tokens re-explaining your stack every conversation
- **Onboarding:** New developers (and new AI sessions) immediately understand the project

### 3.5 When to Use It

- Every project that uses AI-assisted development tools (which is every project, starting today)
- Especially when multiple developers use AI tools on the same codebase
- When your project handles sensitive data

### 3.6 When NOT to Use It

- Do not put secrets or credentials in AGENTS.md -- it is committed to Git
- Do not use it as a substitute for actual documentation -- it is for AI context, not human onboarding
- Do not over-specify -- 50 lines of clear rules beats 500 lines of noise. The model's context window is finite, and every token in AGENTS.md is a token not available for your actual task

### Practical Example: A Minimal AGENTS.md

```markdown
# AGENTS.md

## Project
E-commerce order API. FastAPI + SQLAlchemy + PostgreSQL.

## Commands
- Run tests: `pytest`
- Lint: `ruff check .`
- Format: `ruff format .`

## Rules
- Use async/await for all database operations
- Never log customer PII (name, email, address, payment info)
- All endpoints must have OpenAPI descriptions
- Test fixtures must use fake/generated data only
```

Thirteen lines. That is all it takes to meaningfully improve every AI interaction in your project.

### Exercise 3 -- Write Your Own AGENTS.md

> **Task:** Write a AGENTS.md for a fictional project: a student grade management API for a Portuguese university.
>
> Include: project description, tech stack, 3 conventions, and 3 safety rules.
>
> *5 minutes. You will use this in later exercises.*

---

## 4. What is SKILL.md? What is its Use?

### Learning Objectives
- Understand the difference between AGENTS.md (global context) and skills (reusable task instructions)
- Know how to define and invoke a skill in Claude Code
- See practical backend examples of skill usage

---

### 4.1 Context of the Problem

AGENTS.md gives the model general project context. But what about specific, repeatable tasks? If you find yourself writing the same kind of prompt over and over -- "create a new endpoint with these conventions" or "write a migration for this model" -- you are doing manually what should be automated.

### 4.2 The Problem

Developers repeat the same complex prompts for common tasks:
- "Create a new CRUD endpoint following our project conventions..."
- "Write tests for this endpoint with our standard fixtures..."
- "Generate a migration with our naming conventions..."

Each time, they risk forgetting a convention, a safety rule, or a step. The quality is inconsistent.

### 4.3 The Solution

**Skills** (in Claude Code, defined as Markdown files in `.claude/skills/`) are reusable, parameterized instruction sets that you invoke by name.

**AGENTS.md vs. Skills:**

| Aspect         | AGENTS.md / CLAUDE.md         | Skills                           |
|----------------|-------------------------------|----------------------------------|
| Scope          | Always active, every interaction | Invoked on demand              |
| Purpose        | Global context and rules       | Specific task instructions       |
| Analogy        | Your team's coding standards   | A step-by-step recipe            |
| Token cost     | Always consumed                | Only consumed when invoked       |

**Defining a skill (Claude Code):**

File: `.claude/skills/create_endpoint.md`

```markdown
# Skill: Create CRUD Endpoint

## Description
Creates a complete CRUD endpoint set for a given resource.

## Instructions
When asked to create a CRUD endpoint for a resource:

1. Create the SQLAlchemy model in `src/models/{resource}.py`
2. Create Pydantic schemas in `src/schemas/{resource}.py`:
   - `{Resource}Create` (input)
   - `{Resource}Response` (output)
   - `{Resource}Update` (partial update)
3. Create the router in `src/api/{resource}.py` with:
   - POST /{resources} -- create
   - GET /{resources}/{id} -- read
   - GET /{resources} -- list (with pagination)
   - PATCH /{resources}/{id} -- partial update
   - DELETE /{resources}/{id} -- soft delete
4. Add the router to `src/api/__init__.py`
5. Create tests in `tests/api/test_{resource}.py`
6. NEVER use real PII in test fixtures -- use faker or static fake data

## Example Invocation
"Use the create_endpoint skill to create a CRUD endpoint for 'orders'"
```

**Invoking a skill:**

In Claude Code, you reference skills with the `/` command or by asking the agent to use a specific skill:

```
/create_endpoint for resource "orders"
```

Or in natural language:
```
Use the create_endpoint skill to scaffold CRUD for a "products" resource.
```

### 4.4 Why Use Skills

- **Consistency:** The same task is done the same way every time
- **Safety by default:** Safety rules (like "no PII in fixtures") are baked into the skill, not left to memory
- **Speed:** Complex multi-file scaffolding happens in one invocation
- **Knowledge sharing:** A senior developer writes the skill; the whole team benefits

### 4.5 When to Use Skills

- When you have a recurring multi-step task with consistent conventions
- When onboarding new developers who need to follow patterns they do not yet know
- When safety steps must not be skipped (e.g., "always add input validation")

### 4.6 When NOT to Use Skills

- For one-off tasks -- just write a good prompt
- When the task is too variable to template (e.g., "design the architecture")
- As a replacement for understanding -- if a developer cannot do the task manually, they cannot review the skill's output
- When the skill becomes so long it consumes too much of the context window

### Exercise 4 -- Design a Skill

> **Task:** Design a skill (just the outline, not full Markdown) for one of these tasks:
>
> a) Writing a database migration for a new table
> b) Adding authentication middleware to an endpoint
> c) Creating a new test suite for an existing endpoint
>
> What steps would you include? What safety rules?
>
> *3 minutes. Share with the class.*

---

## 5. What is ARCHITECTURE.md?


### Learning Objectives
- Understand why explicit architecture documentation improves AI-assisted development
- Know what to include in an ARCHITECTURE.md file
- Recognize how LLMs use architectural context for safer, more accurate suggestions

---

### 5.1 Context of the Problem

We have CLAUDE.md for project rules and skills for repeatable tasks. But there is a third piece of context that dramatically improves AI output: understanding how your system is actually built.

When you ask an LLM to add a feature, it needs to know where that feature fits. Without architectural context, the model guesses -- and it guesses based on generic patterns, not your specific system.

### 5.2 The Problem

An LLM that does not understand your architecture will:
- Put code in the wrong layer (business logic in the route handler, database calls in the schema)
- Create new patterns that conflict with existing ones
- Miss integration points (e.g., not knowing that all writes go through an event bus)
- Suggest solutions that violate your data flow constraints

### 5.3 The Solution

**ARCHITECTURE.md** is a file that describes your system's high-level design: components, boundaries, data flows, and key decisions.

**What to include:**

```markdown
# ARCHITECTURE.md

## System Overview
Order management service for an e-commerce platform.
Receives orders via REST API, processes payments via Stripe,
and publishes events to RabbitMQ for downstream services.

## Components
- **API Layer** (src/api/): FastAPI routers. Thin -- validation and routing only.
- **Service Layer** (src/services/): Business logic. All domain rules live here.
- **Repository Layer** (src/repositories/): Database access. SQLAlchemy queries only.
- **Events** (src/events/): RabbitMQ publishers. All state changes emit events.

## Data Flow
Request -> API (validate) -> Service (business logic) -> Repository (DB) -> Event (publish)

## Key Decisions
- Soft deletes only -- no hard deletes in any table
- All monetary values stored as integers (cents), never floats
- Authentication is handled by an external API gateway, not this service
- PII is stored encrypted at rest; decryption happens in the service layer only

## External Dependencies
- PostgreSQL 15 (primary datastore)
- Redis (caching, rate limiting)
- RabbitMQ (event publishing)
- Stripe API (payment processing -- NEVER log Stripe tokens or card data)
```

### 5.4 Why Use It

- The LLM places new code in the correct layer
- Suggestions respect your data flow (no direct DB calls from route handlers)
- Key decisions (like "soft deletes only") are followed without reminders
- Safety constraints on sensitive data (PII encryption, payment tokens) are part of the model's context

### 5.5 When to Use It

- Any project with more than one layer or component
- Any project where multiple developers (or AI tools) add code
- Any project that handles sensitive data with specific storage/access rules

### 5.6 When NOT to Use It

- Tiny scripts or single-file projects -- overkill
- Do not duplicate what is in CLAUDE.md -- ARCHITECTURE.md is about structure, CLAUDE.md is about rules
- Do not make it a full design document -- keep it concise enough that it fits in a context window with room to spare. A page or two, not twenty.

That wraps Part 1. You now have the technical vocabulary: LLMs, prompts, context windows, CLAUDE.md, AGENTS.md, skills, and architecture documentation. After the break, we shift gears. We are going to talk about specifications, subagents, and then our guest from Sonae Sierra is going to make sure you never forget what GDPR means for your code.

Take 10 minutes. Grab a coffee. Come back ready.

---

# BREAK -- 10 MINUTES (1:30-1:40)

---

# PART 2 -- Safety, Law, and Practice

---

## 6. What is SPEC.md and Why Does It Matter for AI?


### Learning Objectives
- Understand what a specification file is and what it should contain
- Know why specs have legal significance when using AI tools
- See how SPEC.md guides LLM behavior and reduces hallucinations

---

### 6.1 Context of the Problem

Welcome back. For Part 2, I want to introduce our guest. We have a legal expert from Sonae Sierra joining us today to bring the perspective that most developer workshops ignore: the legal one. They will be with us for this section and the GDPR section.

Let me start with the technical side, and then I will hand over.

A specification -- a spec -- is the written agreement about what software should do. Not how, but what. As backend developers, you might think of specs as something product managers write and you read. But in the age of AI-assisted development, specs take on a new role: they become instructions for your AI tools.

### 6.2 The Problem

When you prompt an LLM to build a feature without a spec, the model fills in the blanks with assumptions. Sometimes those assumptions are reasonable. Sometimes they are not. And here is the critical part: when the software does the wrong thing because the spec was missing, **who is responsible?**

I will hand over to our guest to explain why that question matters more than you might think.

**[SAULO]:** Thank you. Let me give you a simple scenario.

Your team uses an AI tool to build a user registration endpoint. Nobody wrote down that the email field must be verified before the account is activated. The AI generates an endpoint that creates active accounts immediately. A spam bot discovers this, creates 50,000 accounts, and sends phishing emails from your domain. Your client gets fined. Your company gets sued.

The question in court is: **was there a specification that required email verification?**

If yes -- and the developer (or the AI tool) did not follow it -- there is a clear line of accountability. If no -- there was no spec -- then the question becomes: **who was responsible for defining the requirements?** And that is a much uglier conversation, especially when AI tools are involved.

Here is what I want you to understand: **AI does not reduce your legal responsibility. It increases your need for documentation.**

When a human developer makes a decision, you can ask them why. When an AI generates code and nobody questions it, there is no "why" -- there is only the output. The spec is your "why." It is your defense.

### 6.3 The Solution

From a technical perspective, SPEC.md is a Markdown file that lives in your repository and defines:

```markdown
# SPEC.md

## Feature: User Registration

### Requirements
- Users register with email and password
- Email must be validated (format + verification link)
- Account is INACTIVE until email is verified
- Passwords must be hashed with bcrypt (min cost factor 12)
- Rate limit: max 5 registration attempts per IP per hour

### Business Rules
- Duplicate emails are rejected with 409 Conflict
- Disposable email domains (mailinator, guerrillamail, etc.) are blocked
- Registration events are published to the audit log

### Data Handling
- Email is PII -- stored encrypted at rest
- IP address is logged for rate limiting only, rotated after 24 hours
- Password is NEVER stored in plain text, NEVER logged, NEVER returned in any API response

### Out of Scope
- Social login (OAuth) -- separate feature
- Admin-created accounts -- separate endpoint
```

When this file is in your repo and referenced in CLAUDE.md, the LLM uses it as a constraint. It will not generate an endpoint that stores plain-text passwords or skips email verification -- because the spec says not to.

**[SAULO]:** And from my perspective, that file is also evidence. It shows that your team considered security, defined data handling rules, and set explicit boundaries. If something goes wrong, the conversation changes from "did anyone think about this?" to "the spec was followed / not followed." That is a much better position to be in.

### 6.4 Why Use It

- **Technical:** Reduces LLM hallucinations by providing explicit constraints. The model stops guessing what the feature should do.
- **Legal:** Creates a documented trail of requirements and data handling decisions. Demonstrates due diligence.
- **Team:** Aligns developers (human and AI) on what "done" means. Prevents scope creep.

### 6.5 When to Use It

- Every feature that handles user data
- Every feature with business rules that are not obvious from the code
- Before you start prompting an LLM to build the feature (not after)
- When multiple developers (or AI tools) will work on the same feature

### 6.6 When NOT to Use It

- Do not write specs for trivial changes (fixing a typo, updating a dependency)
- Do not let the spec become stale -- an outdated spec is worse than no spec, because it creates false confidence
- Do not use SPEC.md as the only safety measure -- it guides the AI, but you must still review the output

### Exercise 6.1 -- Spec Review

> **Scenario:** Your SPEC.md says "user passwords must be hashed with bcrypt." The AI generates code that uses MD5. Your test suite does not check the hashing algorithm. The code passes review and ships.
>
> **Discussion questions (2 minutes, open floor):**
> 1. Who is responsible: the developer, the AI, the reviewer, or the spec author?
> 2. What process failure allowed this to happen?
> 3. How would you prevent it?

---

## 7. Subagents: How to Use Them


### Learning Objectives
- Understand the orchestrator-subagent pattern and why it exists
- Know the difference between parallel and sequential subagent execution
- Apply subagent patterns to real backend development tasks

---

### 7.1 Context of the Problem

So far, we have talked about a single AI interaction: you prompt, it responds. But modern AI-assisted development tools -- Claude Code in particular -- can do something more powerful. They can spawn **subagents**: smaller, focused AI instances that handle specific subtasks.

Think of it like this: instead of one developer doing everything, you have a lead developer who delegates to specialists.

### 7.2 The Problem

Complex tasks break down when handled in a single prompt:
- "Build the entire user management module" is too broad for one context window
- Long conversations degrade in quality as the context fills up
- A single agent context mixing database migrations, API routes, tests, and documentation leads to mistakes in all of them

### 7.3 How to Possibly Solve It

You could break the work into smaller prompts yourself and manage the workflow manually. That works, but it is slow and you become the bottleneck.

### 7.4 The Solution

**The Orchestrator-Subagent Pattern:**

```
+------------------+
|   ORCHESTRATOR   |  <-- Main agent, sees the full plan
|   (Lead Dev)     |
+--------+---------+
         |
    +----+----+----+
    |         |    |
+---v---+ +---v---+ +---v---+
| SUB 1 | | SUB 2 | | SUB 3 |
| Model | | Routes| | Tests |
+-------+ +-------+ +-------+
```

The **orchestrator** understands the full task and delegates to **subagents**, each working in an isolated context focused on one concern.

**In Claude Code, this happens automatically** when the task is complex enough. The main agent may spawn subagents using the `Agent` tool to:
- Research a part of the codebase
- Implement a specific file
- Run and analyze tests

**Parallel vs. Sequential Execution:**

| Pattern     | When to use                                   | Example                                          |
|-------------|-----------------------------------------------|--------------------------------------------------|
| **Parallel**    | Subtasks are independent                      | Generate model, schema, and test files simultaneously |
| **Sequential**  | Each subtask depends on the previous one      | Create model -> create migration -> run migration -> verify |

**A real backend example -- building a new feature:**

```
Orchestrator receives: "Add an 'orders' resource to the API"

Step 1 (Sequential): Read SPEC.md and ARCHITECTURE.md for context
Step 2 (Parallel):
  - Subagent A: Create SQLAlchemy model (src/models/order.py)
  - Subagent B: Create Pydantic schemas (src/schemas/order.py)
Step 3 (Sequential, depends on Step 2):
  - Subagent C: Create API routes (src/api/orders.py) using model + schemas
Step 4 (Sequential, depends on Step 3):
  - Subagent D: Write tests (tests/api/test_orders.py)
Step 5 (Sequential):
  - Subagent E: Run tests and fix failures
```

### 7.5 Why Use Subagents

- **Focused context:** Each subagent gets only the context it needs, reducing noise and errors
- **Parallelism:** Independent tasks run simultaneously, saving time
- **Isolation:** A failure in one subagent does not corrupt the others
- **Scalability:** Complex features that would overwhelm a single context become manageable

### 7.6 When to Use Them

- Multi-file features (model + schema + routes + tests)
- Tasks with clearly separable concerns
- When you want to parallelize independent work
- Large codebase exploration tasks (searching across many files)

### 7.7 When NOT to Use Them

- Simple, single-file changes -- the overhead is not worth it
- When subtasks are tightly coupled and cannot be meaningfully separated
- When you need the full conversation history in every subtask (subagents have isolated contexts)
- For security-sensitive orchestration -- do not let a subagent make decisions about access control or data deletion without human review

The key insight is this: subagents are not magic. They are a software engineering pattern -- decomposition and delegation -- applied to AI interactions. The same principles that make good microservice design also make good subagent design: clear boundaries, well-defined interfaces, isolated concerns.

### Exercise 7.1 -- Decompose a Task

> **Task:** You need to add a "notifications" feature: a database table, an async background worker that sends emails, an API endpoint to list a user's notifications, and tests for all of it.
>
> **Question:** How would you decompose this into subagent tasks? Which can run in parallel? Which must be sequential?
>
> *Draw it out (boxes and arrows) on paper. 3 minutes.*

---

## 8. GDPR and PII: What Is This and Why Is It So Important?

### Learning Objectives
- Understand GDPR fundamentals: what it is, who it applies to, and the key principles
- Know what qualifies as PII and why common developer assumptions are wrong
- Recognize the specific risks of using AI/LLM tools with sensitive data
- Have a practical checklist for safe AI-assisted development

---

### 8.1 Context of the Problem

This is the section I have been building toward all day. Every technical choice we discussed -- which LLM to use, how to write prompts, what goes in CLAUDE.md, what goes in SPEC.md -- is shaped by what you are about to learn.

I am going to hand the floor to our guest from Sonae Sierra. They deal with data protection in a company that operates shopping centres across Europe, handling millions of people's data. They know what happens when things go wrong -- not in theory, but in practice.

**[Saulo]:** Thank you. I want to start by saying something directly: **GDPR is not optional, and it is not just for lawyers.** If you write code that processes personal data -- and almost all backend code does -- GDPR applies to you. Not to your company in the abstract. To you, the person writing the code.

### 8.2 The Problem

**[Saulo]:** Let me frame this in developer terms. You have a bug. The bug is that most software systems collect, store, and process personal data in ways that violate European law. Unlike most bugs, this one comes with fines of up to 20 million euros or 4% of global annual turnover -- whichever is higher.

And here is the new dimension: when you use AI tools in your development workflow, you potentially create a **second data processing pipeline** that nobody audited, nobody consented to, and nobody controls.

### 8.3 GDPR Fundamentals

**[Saulo]:**

**What is GDPR?**

The General Data Protection Regulation (EU 2016/679) is European law that governs how personal data is collected, processed, stored, and shared. It took effect on May 25, 2018. It applies to:

- Any organization processing personal data of people in the EU/EEA
- Regardless of where the organization is based
- Regardless of whether the data subject is an EU citizen (it is about location, not citizenship)

**The Six Principles (Article 5):**

| Principle                              | What it means for developers                                                      |
|----------------------------------------|-----------------------------------------------------------------------------------|
| **Lawfulness, fairness, transparency** | You must have a legal basis to process data. Users must know what you do with it. |
| **Purpose limitation**                 | Collect data for a specific purpose. Do not repurpose it without consent.         |
| **Data minimization**                  | Collect only what you need. If you do not need the birth date, do not ask for it. |
| **Accuracy**                           | Keep data correct and up to date. Provide mechanisms for users to correct it.     |
| **Storage limitation**                 | Do not keep data forever. Define and enforce retention periods.                   |
| **Integrity and confidentiality**      | Protect data with appropriate security. Encryption, access controls, audit logs.  |

**Key rights of data subjects:**

- **Right to access:** Users can request all data you hold about them
- **Right to erasure ("right to be forgotten"):** Users can request deletion
- **Right to portability:** Users can request their data in a machine-readable format
- **Right to object:** Users can object to certain types of processing

**[Saulo]:** Every one of these rights translates to code you must write. The right to erasure means your soft-delete is not enough -- you need actual data removal capabilities. The right to access means you need an export function. These are not optional features. They are legal requirements.

### 8.4 What Counts as PII?

**[Saulo]:** PII -- Personally Identifiable Information -- is any data that can identify a person, either directly or in combination with other data. Developers consistently underestimate what qualifies.

**Obviously PII:**
- Full name
- Email address
- Phone number
- National ID number (NIF in Portugal, BSN in Netherlands, etc.)
- Passport / ID card number
- Bank account / credit card number
- Biometric data (fingerprints, face recognition)
- Photograph

**Less obvious, but still PII:**
- IP address (yes, it is PII under GDPR)
- Device identifiers (IMEI, advertising IDs)
- Location data (GPS coordinates, even approximate)
- Cookie identifiers that can be linked to a person
- Behavioral data (purchase history, browsing patterns, click streams)
- Employee ID numbers
- License plate numbers
- Social media handles (when linkable to a real person)

**The combination trap:**
- "Male, 34, software developer in Faro" -- individually, none of these are PII. Combined, in a small city, they might uniquely identify someone. This is called **quasi-identifiers** or **indirect identification**.

**Special category data (extra protected):**
- Health data
- Racial or ethnic origin
- Political opinions
- Religious beliefs
- Trade union membership
- Sexual orientation
- Genetic or biometric data

**[Saulo]:** Processing special category data requires **explicit consent** or a specific legal exemption. There is no "legitimate interest" shortcut for health data. If your API touches medical records, insurance claims, or even fitness tracker data -- you are in special category territory.

### 8.5 Real Consequences

**[Saulo]:** Let me make this concrete with real cases.

**Amazon (Luxembourg, 2021):** 746 million euros. The largest GDPR fine ever. For processing personal data for targeted advertising without proper consent.

**Meta/Instagram (Ireland, 2023):** 405 million euros. For exposing children's email addresses and phone numbers through business accounts.

**Clearview AI (Multiple countries, 2022-2023):** Fined in France (20M), Italy (20M), Greece (20M), UK (7.5M). For scraping publicly available photos to build a facial recognition database without consent.

**Small company example:** A Portuguese SME was fined 400,000 euros for having a CCTV system that monitored employee common areas without proper legal basis or signage.

The point: **fines scale with the violation, not the company size.** And regulators are increasingly focused on **technical implementation**, not just policy documents. They want to see that your code actually does what your privacy policy promises.

### 8.6 AI-Specific Risks

Thank you. Now let me bring this back to our specific context: using AI/LLM tools as developers. Here is where the technical and legal worlds collide.

**Risk 1: Prompt leakage**

When you paste code into an LLM, that code is sent to an external server. If that code contains:
- Database connection strings with real credentials
- API keys or tokens
- User data from logs or database dumps
- Hardcoded test data with real names/emails

...you have just shared PII (or credentials) with a third party. Depending on the LLM provider's terms, that data may be:
- Stored in logs
- Used for model training
- Accessible to the provider's employees
- Subject to a different jurisdiction's laws

```python
# DANGEROUS: Real user data in a prompt
# "Hey Claude, why is this query slow?"
#
# SELECT * FROM users WHERE email = 'maria.joaquina@gmail.com'
# AND nif = '123456789';
#
# You just sent a real person's email and tax ID to an external service.

# SAFE: Anonymized data in a prompt
# "Hey Claude, why is this query slow?"
#
# SELECT * FROM users WHERE email = 'user@example.com'
# AND nif = '000000000';
```

**Risk 2: Training data contamination**

Some LLM providers use your conversations to train future models. This means PII you share could theoretically appear in another user's output months later. Even if the probability is low, the **liability** is yours.

**Risk 3: Log exposure**

AI tool interactions are often logged -- by the tool, by your IDE, by browser extensions. Those logs may contain the PII you prompted with, now sitting in a log file with different access controls than your database.

**Risk 4: Context window residue**

In a long conversation, PII from an early message remains in the context window for all subsequent messages. If you paste a user record early in the session and later ask the model to "generate sample data," the model might use that real data as a template.

```python
# Example: What your CLAUDE.md safety rules should include

# In CLAUDE.md:
# ## PII Safety Rules
# - NEVER include real user data in prompts, examples, or test fixtures
# - NEVER log function arguments that may contain PII
# - Use placeholder data: user@example.com, "Jane Doe", NIF: 000000000
# - If you see what appears to be real PII in code, flag it immediately
# - All database queries in examples must use parameterized placeholders
```

**Risk 5: Automated code generation without review**

An LLM might generate code that:
- Logs user email addresses in plain text
- Returns more user fields in an API response than necessary (violating data minimization)
- Stores data without encryption
- Misses retention/deletion logic

If you ship that code without review, **you** are the data processor who failed to implement GDPR safeguards, not the AI.

### 8.7 The Developer Checklist for Safe AI-Assisted Development

**Before you prompt:**
- [ ] Scrub all real PII from code snippets, logs, and database output before pasting
- [ ] Replace real credentials with placeholders
- [ ] Verify your LLM provider's data processing terms (does it train on your data?)
- [ ] Use API/Pro tiers, not free consumer tiers, for any work-related prompts

**In your project configuration:**
- [ ] CLAUDE.md/AGENTS.md includes explicit PII safety rules
- [ ] SPEC.md defines data handling requirements for each feature
- [ ] ARCHITECTURE.md documents where PII is stored and how it flows
- [ ] Test fixtures use only fake/generated data (use `faker` library)

**In your code:**
- [ ] Log user actions by ID, never by name/email/IP
- [ ] API responses return only the fields the client needs (data minimization)
- [ ] Implement soft delete AND hard delete (for right-to-erasure requests)
- [ ] Encrypt PII at rest
- [ ] Parameterize all database queries (no string concatenation)
- [ ] Implement data retention policies (auto-delete after defined period)

**In your workflow:**
- [ ] Review all AI-generated code for PII handling before committing
- [ ] Never commit real user data to Git, even in test files
- [ ] Run a PII scanner on your codebase periodically
- [ ] Document your data processing in a Record of Processing Activities (ROPA)

### Practical Example: Safe vs. Unsafe Logging

```python
# UNSAFE -- logs PII
import logging

logger = logging.getLogger(__name__)

@app.post("/users")
async def create_user(user: UserCreate):
    logger.info(f"Creating user: {user.name}, email: {user.email}")  # PII LEAK
    db_user = await user_repo.create(user)
    logger.info(f"User created: {db_user.name} (ID: {db_user.id})")  # PII LEAK
    return db_user  # Returns ALL fields, including internal ones


# SAFE -- logs only identifiers
import logging

logger = logging.getLogger(__name__)

@app.post("/users", response_model=UserResponse)  # Controls output fields
async def create_user(user: UserCreate):
    logger.info("Creating user", extra={"request_id": request_id})
    db_user = await user_repo.create(user)
    logger.info("User created", extra={"user_id": db_user.id})  # ID only
    return db_user  # UserResponse schema excludes internal fields
```

### Practical Example: Safe Test Fixtures

```python
# UNSAFE -- real data or realistic-looking data that could be real
TEST_USERS = [
    {"name": "Maria Silva", "email": "maria.joaquina@gmail.com", "nif": "234567890"},
    {"name": "Joao Santos", "email": "jalface@hotmail.com", "nif": "345678901"},
]

# SAFE -- obviously fake data using faker
from faker import Faker

fake = Faker("pt_PT")  # Portuguese locale for realistic but fake data

def generate_test_user():
    return {
        "name": fake.name(),
        "email": fake.email(),
        "nif": "000000000",  # Explicitly invalid NIF
    }

# Or use clearly fictional static data
TEST_USERS = [
    {"name": "Test User Alpha", "email": "alpha@test.example.com", "nif": "000000000"},
    {"name": "Test User Beta", "email": "beta@test.example.com", "nif": "000000000"},
]
```

### Exercise 8.1 -- PII Audit

> **Task:** The following API response is returned when a client calls `GET /users/123`. Identify all PII fields and determine which should be removed to comply with data minimization.
>
> ```json
> {
>   "id": "550e8400-e29b-41d4-a716-446655440000",
>   "name": "Ana Ferreira",
>   "email": "ana.ferreira@example.com",
>   "nif": "123456789",
>   "phone": "+351912345678",
>   "created_at": "2026-01-15T10:30:00Z",
>   "last_login_ip": "192.168.1.42",
>   "browser_fingerprint": "a1b2c3d4e5f6",
>   "role": "viewer",
>   "department": "Engineering",
>   "date_of_birth": "1994-07-22",
>   "salary": 2800.00,
>   "is_active": true
> }
> ```
>
> **Questions:**
> 1. Which fields are PII?
> 2. Which fields should a typical "get user profile" endpoint return?
> 3. Which fields should NEVER be in this response?
> 4. Write a Pydantic `UserResponse` schema that enforces data minimization.
>
> *5 minutes. Discuss in pairs, then we review together.*

### Exercise 8.2 -- Spot the Risk

> **Scenario:** A developer runs this command in their terminal while using Claude Code:
>
> ```bash
> cat production_users.csv | head -20
> ```
>
> Then asks Claude Code: "Help me write a migration to add a 'verified' column to the users table. Here are some sample rows so you can see the schema."
>
> **Questions:**
> 1. What went wrong?
> 2. Where does that data now exist (list all locations)?
> 3. How should the developer have accomplished the same goal safely?
>
> *3 minutes. Open discussion.*

---

## 9. Vibecoding: How to Start Safely


### Learning Objectives
- Understand what vibecoding is and why it requires discipline, not just tools
- Set up a safe vibecoding environment with the files covered in this masterclass
- Follow a workflow that balances speed with safety

---

### 9.1 Context of the Problem

We have covered the pieces. Now let us put them together.

"Vibecoding" is a term coined by Andrej Karpathy (co-founder of OpenAI, former head of AI at Tesla) in early 2025. He described it as a way of coding where you "fully give in to the vibes" -- you describe what you want in natural language, the AI writes the code, and you guide the process through conversation rather than typing every line.

It caught on because it describes something real: a fundamentally different relationship with code, where you are the architect and reviewer, and the AI is the builder.

### 9.2 The Problem

Vibecoding, done naively, is dangerous:

- **Speed without review = bugs at scale.** The AI writes code faster than you can review it. If you ship without reviewing, you ship bugs faster.
- **Delegation without understanding = undebuggable systems.** If you cannot explain what the code does, you cannot debug it, extend it, or secure it.
- **Convenience without boundaries = data leaks.** The easiest thing to do is paste everything into the chat. The safest thing to do is the opposite.

The internet is full of "I built a SaaS in 2 hours with vibecoding" stories. What you do not see are the "I leaked my users' data because I vibecoded without thinking" stories -- because those people are too busy talking to lawyers.

### 9.3 The Solution: Safe Vibecoding Workflow

**Step 1: Set up your project configuration files**

Before you write a single line of AI-assisted code, create these files:

```
your-project/
  CLAUDE.md          # Project rules, tech stack, safety constraints
  AGENTS.md          # Cross-tool AI configuration (if needed)
  ARCHITECTURE.md    # System design, components, data flows
  SPEC.md            # Feature requirements and data handling rules
  .claude/
    skills/          # Reusable task instructions
```

This is your **safety net**. Every AI interaction inherits these constraints.

**Step 2: The prompt-review-test-commit cycle**

```
+---------+     +---------+     +--------+     +---------+
| PROMPT  | --> | REVIEW  | --> |  TEST  | --> | COMMIT  |
| (ask AI)|     | (read   |     | (run   |     | (save   |
|         |     |  every  |     |  tests)|     |  if     |
|         |     |  line)  |     |        |     |  green) |
+---------+     +---------+     +--------+     +---------+
      ^                                              |
      |               ITERATE                        |
      +----------------------------------------------+
```

**PROMPT:** Ask the AI to do one thing at a time. Not "build the entire feature" but "create the SQLAlchemy model for orders with these fields."

**REVIEW:** Read every line of generated code. Specifically check:
- Does it log PII? (names, emails, IPs)
- Does it expose more data than needed in API responses?
- Does it handle errors securely (no stack traces to the client)?
- Does it use parameterized queries?
- Does it follow your ARCHITECTURE.md layers?

**TEST:** Run the tests. If there are no tests, write them first (or have the AI write them based on your SPEC.md).

**COMMIT:** Only commit code that passes review and tests. Write clear commit messages. Small commits are better than large ones -- they are easier to revert.

**Step 3: Know what NEVER to send to an LLM**

| Category                            | Examples                                                  | What to do instead                      |
|-------------------------------------|-----------------------------------------------------------|-----------------------------------------|
| **User PII**                        | Names, emails, NIFs, phone numbers from your DB           | Use fake/anonymized data                |
| **Credentials**                     | API keys, database passwords, tokens                      | Use placeholder values like `<API_KEY>` |
| **Proprietary business logic**      | Trade secrets, pricing algorithms, confidential processes | Describe the pattern, not the specifics |
| **Production logs with user data**  | Server logs containing IPs, user agents, request bodies   | Sanitize or redact before sharing       |
| **Internal infrastructure details** | Server IPs, network topology, security configurations     | Abstract to generic descriptions        |

**Step 4: Configure your tools**

```python
# .env -- NEVER committed to Git, NEVER shared with LLM
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxx
JWT_SECRET=your-secret-key-here

# .env.example -- committed to Git, safe to share with LLM
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
STRIPE_SECRET_KEY=sk_test_placeholder
JWT_SECRET=change-me-in-production
```

```gitignore
# .gitignore -- essential entries
.env
*.pem
*.key
production_*.csv
**/secrets/
```

### 9.4 Why This Workflow

- **Configuration files** mean you set safety rules once, not every conversation
- **The review step** catches the AI's mistakes before they reach production
- **The test step** catches the mistakes the review step missed
- **The commit step** creates a recoverable history -- if something slips through, you can find and revert it
- **The "never send" list** prevents data leaks at the source

### 9.5 When to Vibecode

- Scaffolding new projects, modules, or endpoints
- Writing boilerplate (CRUD operations, standard patterns)
- Generating test suites from specifications
- Exploring unfamiliar libraries or APIs
- Refactoring with clear before/after patterns
- Writing documentation and docstrings

### 9.6 When NOT to Vibecode

- **Security-critical code** (authentication flows, encryption, access control) -- write this yourself or review with extreme scrutiny
- **Performance-critical code** where you need to understand every operation
- **When you cannot explain the output** -- if you cannot read the generated code and explain what it does, you are not ready to ship it
- **With production data** -- never paste real user data into an AI tool, no matter how convenient
- **Under time pressure without review** -- "the deadline is tonight, just ship what the AI wrote" is exactly how incidents happen
- **Complex business logic** that requires domain expertise the model does not have

### 9.7 Common Pitfalls

| Pitfall                           | Consequence                                    | Prevention                                    |
|-----------------------------------|------------------------------------------------|-----------------------------------------------|
| "It works, ship it"               | Untested, unreviewed code in production        | Enforce the prompt-review-test-commit cycle   |
| Pasting production data           | PII sent to external service                   | Always anonymize before prompting             |
| Trusting without verifying        | Hallucinated dependencies, wrong API calls     | Run the code. Read the docs. Check versions.  |
| Vibecoding security features      | Subtle vulnerabilities in auth/crypto          | Human-write or expert-review security code    |
| No CLAUDE.md/AGENTS.md            | Inconsistent, unsafe AI output                 | Set up project files BEFORE coding            |
| Context window overload           | Model "forgets" earlier instructions           | Keep conversations focused and short          |
| Using free-tier for work data     | Data potentially used for training             | Use API/Pro tiers with no-training policies   |

### Practical Example: A Complete Safe Vibecoding Session

```
SESSION START
=============

1. Project is set up with CLAUDE.md, ARCHITECTURE.md, SPEC.md

2. Developer prompt:
   "Based on SPEC.md, create the SQLAlchemy model for the 'orders' resource.
    Follow ARCHITECTURE.md -- the model goes in src/models/order.py."

3. AI generates code. Developer reviews:
   - [x] Fields match SPEC.md
   - [x] No PII in default __repr__
   - [x] Soft delete column included
   - [x] Timestamps use UTC
   - [ ] Missing: encryption for customer_email field

4. Developer prompt:
   "The customer_email field needs to be encrypted at rest.
    Use the EncryptedString pattern from our existing User model."

5. AI updates code. Developer reviews:
   - [x] Encryption applied correctly

6. Developer prompt:
   "Write tests for this model. Use faker for test data.
    Do NOT use any real names, emails, or NIFs."

7. AI generates tests. Developer reviews and runs:
   $ pytest tests/models/test_order.py -v
   All pass.

8. Developer commits:
   $ git add src/models/order.py tests/models/test_order.py
   $ git commit -m "feat: add Order model with encrypted PII fields"

SESSION END
===========
```

### Exercise 9.1 -- Safe Vibecoding Setup

> **Task:** Using the fictional university grade management API from Exercise 3.1, create a complete project configuration:
>
> 1. Finalize your CLAUDE.md (from Exercise 3.1)
> 2. Write a 5-line ARCHITECTURE.md (layers and data flow)
> 3. Write a SPEC.md for one feature: "Student Grade Submission"
>    - Include: what data is collected, who can submit, data handling rules
>    - Remember: student names, IDs, and grades are all PII
>
> *7 minutes. This is your take-home foundation.*

### Exercise 9.2 -- Red Flags Review

> **Task:** Your colleague vibecoded the following endpoint. Find all the problems.
>
> ```python
> @app.post("/students")
> async def create_student(student: dict):
>     print(f"Creating student: {student}")
>     query = f"INSERT INTO students (name, email, grade) VALUES ('{student['name']}', '{student['email']}', '{student['grade']}')"
>     await db.execute(query)
>     logger.info(f"Student {student['name']} ({student['email']}) created with grade {student['grade']}")
>     return {"status": "ok", "data": student}
> ```
>
> **Hint:** There are at least 7 problems. How many can you find?
>
> *5 minutes. List them, then we review as a class.*
>
> **Answer key (do not read until you have tried):**
> 1. `dict` instead of a Pydantic model -- no input validation
> 2. `print()` with full student data -- PII in stdout
> 3. SQL string concatenation -- SQL injection vulnerability
> 4. Logger includes name and email -- PII in logs
> 5. Logger includes grade -- this is special educational data
> 6. Returns the raw `student` dict -- no output schema, potential data leak
> 7. No error handling -- unhandled exceptions may expose internals
> 8. No authentication/authorization check
> 9. No response model controlling which fields are returned

---

## Key Takeaways

- **An LLM is a tool, not a colleague.** It does not understand your code, your users, or the law. It predicts tokens. Your job is to guide it and verify its output.
- **Configuration files are your safety net.** CLAUDE.md, SPEC.md, and ARCHITECTURE.md are not bureaucracy -- they are the instructions that keep AI-generated code safe and consistent.
- **PII is broader than you think.** IP addresses, device IDs, behavioral data, and combinations of seemingly innocent fields are all personal data under GDPR. When in doubt, treat it as PII.
- **Never send real user data to an LLM.** No exceptions. Anonymize, redact, or use fake data. The convenience is not worth the legal and ethical risk.
- **The vibecoding workflow is: prompt, review, test, commit.** Skip any step and you are shipping risk, not code.
- **"When NOT to use it" is as important as "when to use it."** Every tool has failure modes. Knowing them is what separates a professional from a hobbyist.

---

## Discussion Questions

1. **Your company wants to use a fine-tuned LLM trained on internal code and documentation. What GDPR considerations apply to the training data? What about the model's outputs?**

2. **A junior developer on your team vibecodes an entire authentication system and says "it passes all the tests." What is your response, and what process would you put in place to prevent this situation?**

3. **You are building a health insurance API. A client asks you to use AI to generate synthetic test data that "looks realistic." What are the risks, even if no real patient data is used?**

4. **The GDPR "right to be forgotten" requires deleting a user's data. But your LLM has been trained on conversations that included that user's data. Is the data truly deleted? Who is responsible?**

5. **Should there be a legal requirement to disclose when production code was generated by AI? Why or why not? What would the implications be for liability?**

---

## Further Reading / References

### Technical
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code) -- Official documentation for Claude Code
- [AGENTS.md specification](https://github.com/anthropics/agents-md) -- Community standard for AI agent configuration
- [Andrej Karpathy on Vibecoding](https://x.com/karpathy/status/1886192184808149383) -- The original post that coined the term
- [Prompt Engineering Guide](https://www.promptingguide.ai/) -- Comprehensive guide to prompt techniques

### Legal / GDPR
- [GDPR Full Text](https://gdpr-info.eu/) -- The complete regulation, searchable
- [CNPD (Portuguese Data Protection Authority)](https://www.cnpd.pt/) -- Portugal's national authority for GDPR enforcement
- [European Data Protection Board (EDPB) Guidelines on AI](https://edpb.europa.eu/) -- Regulatory guidance on AI and personal data
- [ICO Guide to GDPR](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/) -- Excellent plain-English GDPR guide (UK ICO)

### Tools
- [Faker (Python)](https://faker.readthedocs.io/) -- Library for generating fake data for tests
- [Ruff](https://docs.astral.sh/ruff/) -- Fast Python linter and formatter
- [Ollama](https://ollama.ai/) -- Run LLMs locally for maximum privacy

---

> **End of Masterclass**
> **Total Duration:** 3 hours 20 minutes (including 10-minute break)
> **Files referenced:** CLAUDE.md, AGENTS.md, ARCHITECTURE.md, SPEC.md, SKILL.md
> **Delivery format:** Co-hosted (HOST + LAWYER from Sonae Sierra)
