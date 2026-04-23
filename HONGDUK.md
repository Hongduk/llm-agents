# Hongduk Ahn — LLM Engineering Portfolio

> AI Adoption Architect | Microsoft Korea | LTIMindtree
> Building the bridge between enterprise strategy and AI engineering

---

## About Me

Hongduk Ahn is an AI Adoption Architect at Microsoft Korea, placed through LTIMindtree. With 16+ years of enterprise experience at LG Group (LG CNS and LG Energy Solution), he combines deep business strategy expertise with hands-on AI engineering skills.

**Background:**
- AI Adoption Architect, Microsoft Korea (March 2026 — present)
- Senior Business Developer / Consultant, LG CNS & LG Energy Solution (16+ years)
- Founder of KooRoo — Korea's largest battery swapping station network (400+ stations)
- MBA in Finance, Korea University
- Two PCT patents in battery exchange systems
- Part-time AI lecturer, KCCI (Korea Chamber of Commerce and Industry)

**Goal:** Become an AI Engineer who can navigate and bridge the business and technical sectors in Korea.

---

## Certification Roadmap

| Certification | Target Date | Status |
|---|---|---|
| Microsoft Azure Fundamentals | Completed | ✅ |
| Microsoft Azure Data Fundamentals | Completed | ✅ |
| Microsoft Azure AI Fundamentals | Completed | ✅ |
| Microsoft Security, Compliance & Identity | Completed | ✅ |
| ADsP (Data Analytics Professional) | Completed | ✅ |
| AI-103 (Azure AI App & Agent Developer) | June 2026 | 🔜 |
| CCA Foundations (Claude Certified Architect) | June 2026 | 🔜 |
| TOEFL iBT (110+ target) | June 2026 | 🔜 |
| Georgia Tech OMSCS Application | August 2026 | 🔜 |
| CCA Advanced Architect | 2027 | 🔜 |

---

## Core Track Progress

### Week 5 — RAG (Retrieval Augmented Generation) ✅
- [x] Day 1: Brute-force RAG (keyword matching)
- [x] Day 2: Vector store with ChromaDB + HuggingFace embeddings
- [x] Day 3: RAG pipeline with Gradio UI
- [x] Day 4: RAG evaluation (MRR, NDCG, LLM-as-judge)
- [x] Day 5: Advanced RAG (LLM chunking + query rewriting + BGE reranker)

**Key implementations:**
- Built Insurellm RAG chatbot from scratch
- Implemented BGE reranker (`bge-reranker-v2-m3`) — multilingual, Korean support
- Advanced pipeline: LLM chunking → query rewriting → vector search → reranking
- Evaluation metrics: MRR, NDCG, keyword coverage, answer accuracy

### Week 6 — Fine-Tuning
- [ ] Coming soon

---

## Agentic Track Progress

### Week 1 — Agentic Foundations ✅

#### Lab 2 — Multi-LLM Competition
**Pattern: LLM-as-Judge**

```python
# Flow
GPT generates challenging question
→ Multiple LLMs compete (GPT, Claude, Gemini, Solar, Qwen, Gemma)
→ Judge LLM ranks responses → returns structured JSON
→ Print final rankings
```

**Models used:** GPT-5-nano, Claude Sonnet 4.5, Gemini 2.5 Flash, Solar Pro, Qwen3-32B, Gemma
**Key learning:** Parallel LLM execution, LLM-as-Judge pattern, JSON result parsing

---

#### Lab 3 — Digital Twin with Reflection
**Pattern: Generate → Evaluate → Retry**

```python
# Flow
LinkedIn PDF + summary.txt → injected into system prompt
→ GPT answers as Hongduk Ahn
→ Claude judges answer quality (Pydantic Evaluation model)
→ if is_acceptable = False → rerun with feedback injected
→ return improved answer
```

**Key learning:**
- System prompt persistence vs user message context
- Pydantic BaseModel for structured LLM output
- Reflection pattern — agent evaluates and improves its own output
- Two LLMs (GPT agent + Claude judge) for objectivity

---

#### Lab 4 — Tool Calling + Real Notifications ✅
**Pattern: Tool Calling + Agentic While Loop**

```python
# Two tools built
def record_user_details(email, name="Name not provided", notes="not provided"):
    push(f"Recording interest from {name} with email {email}")
    return {"recorded": "ok"}

def record_unknown_question(question):
    push(f"Recording {question} that I could not answer")
    return {"recorded": "ok"}
```

**Key learning:**
- Tool definition JSON structure (name, description, parameters)
- `globals()` pattern for clean tool routing — no IF statements
- `**arguments` for unpacking dictionary into function parameters
- `while not done` loop for chained tool calls
- `tool_call_id` for linking results to requests
- Pushover API for real push notifications to phone 📱

---

#### Lab 5 Extra — Agent Loop from Scratch ✅
**Pattern: Plan → Execute → Answer**

- Built pure agentic loop with todo planning tools
- create_todos() → agent plans all steps upfront
- mark_complete() → agent executes each step
- Gradio UI shows plan + answer
- Terminal shows real-time tool calls
- Key learning: array parameter type, globals() pattern,
  autonomous problem solving without human input

---

### Week 2 — OpenAI Agents SDK

#### Day 1 — Agents SDK Basics ✅
**Key difference from Week 1:**
```python
# Week 1 — manual (20+ lines)
system_prompt = "..."
messages = [{"role": "system", "content": system_prompt}]
response = openai.chat.completions.create(...)
while not done: ...

# Week 2 — SDK (3 lines)
agent = Agent(name="Jokester", instructions="...", model="gpt-4o-mini")
result = await Runner.run(agent, "Tell a joke")
print(result.final_output)
```
- Agent, Runner, trace
- trace visible at platform.openai.com/traces
- await + async for non-blocking LLM calls

#### Day 2 — Multi-Agent Sales System ✅
**Patterns: Parallelization + LLM-as-Judge + Orchestrator**
```python
# @function_tool replaces all JSON boilerplate
@function_tool
def send_email(body: str):
    """Send email to prospects"""
    print(body)

# agent.as_tool() converts agent to tool
tool1 = sales_agent1.as_tool(tool_name="sales_agent1", tool_description="Write cold email")

# asyncio.gather() runs agents in parallel
results = await asyncio.gather(
    Runner.run(sales_agent1, message),
    Runner.run(sales_agent2, message),
    Runner.run(sales_agent3, message),
)
```
- 3 sales agents (professional/humorous/concise) in parallel
- sales_picker selects best email (LLM-as-Judge)
- emailer_agent formats + sends (handoff)
- Key learning: @function_tool, as_tool(), handoffs vs tools

#### Day 3 — Different Models + Guardrails ✅
**Patterns: Multi-model + Structured Output + Safety**
```python
# Use any LLM with Agents SDK
gemini_client = AsyncOpenAI(base_url=GEMINI_BASE_URL, api_key=google_api_key)
gemini_model = OpenAIChatCompletionsModel(model="gemini-2.5-flash", openai_client=gemini_client)
sales_agent1 = Agent(name="Gemini Agent", model=gemini_model)

# Structured output for guardrail
class NameCheckOutput(BaseModel):
    is_name_in_message: bool
    name: str

# Input guardrail blocks personal names
@input_guardrail
async def guardrail_against_name(ctx, agent, message):
    result = await Runner.run(guardrail_agent, message, context=ctx.context)
    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=result.final_output.is_name_in_message
    )
```
- Agents using: Gemini, Groq (gpt-oss-120b), Upstage (Solar), OpenRouter (Claude)
- AsyncOpenAI + OpenAIChatCompletionsModel for non-OpenAI models
- @input_guardrail blocks messages with personal names
- tripwire_triggered=True → agent never runs

#### Day 4 — Deep Research Agent ✅
**Pattern: Orchestrated Pipeline + Reflection**
```
User query
    ↓
planner_agent → WebSearchPlan (5 searches)
    ↓
5 parallel web searches (asyncio.create_task)
    ↓
write_report_with_reflection:
    writer_agent → initial draft
    critic_agent → evaluates (is_approved, critique_points, rating/10)
    if rejected → refine → repeat (max 3 iterations)
    ↓
email_agent → formats HTML → sends via SendGrid → mhda08@naver.com 📧
```

**Key new concepts:**
```python
# WebSearchTool — built-in internet search
search_agent = Agent(
    tools=[WebSearchTool(search_context_size="low")],
    model_settings=ModelSettings(tool_choice="required")  # force tool use
)

# Field() — better structured output
class WebSearchItem(BaseModel):
    reason: str = Field(description="Why this search matters")
    query: str = Field(description="The search term")

# create_task — dynamic parallel execution
tasks = [asyncio.create_task(search(item)) for item in search_plan.searches]
results = await asyncio.gather(*tasks)

# Custom Reflection for report quality
class Criticism(BaseModel):
    is_approved: bool
    critique_points: list[str]
    rating: int  # quality score 1-10
```

---

## Key Concepts Mastered

### Week 1 vs Week 2 Comparison
```
                    Week 1              Week 2
Tool definition:    JSON dict           @function_tool
Agent creation:     manual prompt       Agent(name, instructions, model)
Agent execution:    while not done      Runner.run()
Tool routing:       globals().get()     handled by SDK
Parallel exec:      manual              asyncio.gather()
Agent→Agent:        manual messages     as_tool() + handoffs
Tracing:            print statements    trace() + OpenAI dashboard
Multi-model:        OpenAI() + base_url AsyncOpenAI + OpenAIChatCompletionsModel
Safety:             manual checks       @input_guardrail + tripwire
Web search:         manual tool         WebSearchTool (built-in)
```

### Agentic Patterns Covered
```
Pattern 1: LLM-as-Judge        → one LLM evaluates another
Pattern 2: Reflection          → generate → evaluate → retry
Pattern 3: Tool Calling        → agent triggers real-world actions
Pattern 4: Plan → Execute      → agent plans before acting
Pattern 5: Parallelization     → multiple agents simultaneously
Pattern 6: Orchestrator        → manager coordinates subagents
Pattern 7: Handoffs            → agent delegates permanently
Pattern 8: Guardrails          → safety checks before agent runs
Pattern 9: Pipeline            → sequential steps with parallel middle
```
- [ ] Coming soon

### Week 3 — Multi-Agent Systems
- [ ] Coming soon

### Week 4 — Production Agents
- [ ] Coming soon

### Week 5 — Agent Deployment
- [ ] Coming soon

---

## Production Track Progress

### Coming August 2026
- [ ] To be updated

---

## Key Concepts Mastered

### Tool Calling
```python
# Tool definition — three levels
{
    "type": "function",           # OpenAI required wrapper
    "function": {
        "name": "function_name",  # must match Python function exactly
        "description": "...",     # THE PROMPT — tells LLM when to call
        "parameters": {
            "type": "object",
            "properties": {...},
            "required": [...],    # LIST of mandatory parameters
            "additionalProperties": False  # prevent LLM inventing extras
        }
    }
}

# Elegant tool routing
tool = globals().get(tool_name)       # find function by name
result = tool(**arguments) if tool else {}  # unpack dict into kwargs
```

### Reflection Pattern
```python
reply = agent.generate(question)
evaluation = judge.evaluate(reply)    # Pydantic structured output
if not evaluation.is_acceptable:
    reply = agent.retry(reply, evaluation.feedback)
```

### Agentic While Loop
```python
while not done:
    response = openai.chat.completions.create(..., tools=tools)
    if response.finish_reason == "tool_calls":
        results = handle_tool_calls(...)
        messages.extend(results)      # feed results back to LLM
    else:
        done = True                   # LLM finished     
```

### Agent Loop (Pure Form)
- create_todos ONCE → plan all steps
- mark_complete × N → execute each step
- while loop exits when finish_reason = "stop"
- array parameter type for multiple values at once


### RAG Pipeline (Advanced)
```python
# Day 5 advanced RAG flow
fetch_documents()           # load 76 knowledge base files
↓
LLM chunking                # semantic chunks with headline + summary + text
↓
text-embedding-3-large      # 3072 dimensional vectors
↓
ChromaDB                    # vector store
↓
rewrite_query()             # query rewriting for better retrieval
↓
fetch_context_unranked()    # retrieve top-K candidates
↓
BGE reranker                # bge-reranker-v2-m3 (Korean support)
↓
answer_question()           # final LLM answer with context
```

### Pydantic Structured Output
```python
from pydantic import BaseModel

class Evaluation(BaseModel):
    is_acceptable: bool    # enforces True/False
    feedback: str          # enforces string

# Force LLM to return structured JSON
response = client.beta.chat.completions.parse(
    model="claude-sonnet-4-5-20251001",
    messages=messages,
    response_format=Evaluation    # Pydantic model enforces structure
)
evaluation = response.choices[0].message.parsed
# → evaluation.is_acceptable  always bool
# → evaluation.feedback       always string
```

---

## Tech Stack

```
LLMs:
  OpenAI:     GPT-5-nano, GPT-5-mini, GPT-4.1-nano, GPT-4.1-mini
  Anthropic:  Claude Sonnet 4.5, Claude Opus 4.6
  Google:     Gemini 2.0 Flash, Gemini 2.5 Flash, Gemma
  Upstage:    Solar Pro
  Groq:       Qwen3-32B, GPT-OSS-120B, LLaMA 3.3
  DeepSeek:   DeepSeek models

Frameworks:
  Gradio      UI for chatbots
  Pydantic    Structured LLM output
  ChromaDB    Vector store
  LangChain   Document loading (Day 2-4 RAG)
  LiteLLM     Multi-provider LLM calls

Tools & APIs:
  OpenAI Embeddings   text-embedding-3-large (3072 dims)
  FlagEmbedding       BGE reranker (bge-reranker-v2-m3)
  Pushover            Push notifications
  PyPDF               PDF reading

Language: Python 3.12
Environment: Jupyter notebooks + VS Code + uv
```

---

## Contact

- **Email:** mhda80@gmail.com
- **LinkedIn:** [linkedin.com/in/hongduk-ahn-9a1761149](https://www.linkedin.com/in/hongduk-ahn-9a1761149)
- **Location:** Seoul, Korea
- **Current Role:** AI Adoption Architect, Microsoft Korea (via LTIMindtree)