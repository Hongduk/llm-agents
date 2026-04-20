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

#### 5_extra — Agent Loop from Scratch
**Pattern: Pure Agentic Loop**
- [ ] Coming soon

---

### Week 2 — Advanced Agents
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

## Patents

1. **Battery Exchange Device and Battery Exchange System**
   (배터리 교환 장치 및 배터리 교환 시스템)

2. **Server and Its Operating Method, Battery Exchange System**
   (서버 및 그것의 동작 방법, 배터리 교환 시스템)

Both patents cover hardware/software architecture of battery swapping infrastructure, developed during founding of KooRoo under LG Energy Solution.

---

## Contact

- **Email:** mhda80@gmail.com
- **LinkedIn:** [linkedin.com/in/hongduk-ahn-9a1761149](https://www.linkedin.com/in/hongduk-ahn-9a1761149)
- **Location:** Seoul, Korea
- **Current Role:** AI Adoption Architect, Microsoft Korea (via LTIMindtree)