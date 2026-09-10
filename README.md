# GovNavigator AI 🇸🇦
## Multi-Agent AI for Government Service Navigation

> **From “I have a government problem” to “Here is the right official service, entity, and next step.”**

---

## 1. Overview

**GovNavigator AI** is a multi-agent AI system designed to help users identify the appropriate Saudi government service and responsible entity from a natural-language description of their problem.

Instead of requiring users to know the exact government terminology, service name, responsible entity, or digital platform, GovNavigator starts from the user's real-world need, retrieves relevant official service information, verifies the recommendation, and generates a safe next-step plan.

### Positioning

GovNavigator is designed as a **navigation and orchestration layer over existing government services**.

It does **not** replace government platforms and does **not** execute government transactions on behalf of users.

---

## 2. The Problem

Saudi Arabia has made significant progress in digital government services. The challenge addressed by this project is therefore not simply the availability of digital services, but **helping users discover the appropriate service for their specific need**.

The official GOV.SA Service Directory currently lists **6,028 government services** and provides a centralized way to search for and access government services.

As the digital government ecosystem grows, users may describe their needs in everyday language without knowing:

- The official name of the required service
- Which government entity is responsible
- Which digital platform provides the service
- Whether their need corresponds to a license, registration, inquiry, complaint, certificate, or another procedure

This creates a **Service Discovery & Navigation Gap**:

```text
User's real-world need
        ↓
Exact government service
        ↓
Responsible entity
        ↓
Correct digital channel
```

This focus is aligned with the broader emphasis of Saudi Arabia's Digital Government Authority on digital experience, usability, navigation, and ease of finding information within government services.

### Target Users

| User Group | Need |
|---|---|
| **Citizens & Residents** | Find the correct public service without knowing its official name or the responsible entity |
| **Business Owners / Entrepreneurs** | Identify licensing, registration, and compliance services when starting or running a business |
| **Non-Arabic-speaking residents** | Navigate government terminology through natural-language, multilingual queries |
| **Authorized Government Analysts** (RBAC role) | Review anonymized usage patterns and system behavior for service-improvement insights |

### Why AI Agents Are Suitable

A single rule-based keyword search cannot bridge the gap between how a person *describes* a problem and how the government *names* the corresponding service, because:

- **Natural language is ambiguous** — the same need can be phrased in many ways, in Arabic or English, formally or colloquially. Rule-based matching requires the user to already know the right keywords, which is exactly the gap this project addresses.
- **The task decomposes into distinct reasoning steps** — understanding intent, retrieving candidates, selecting an entity, verifying that selection against evidence, deciding whether to proceed or retry, and producing an action plan. Each step has a different failure mode, so a single LLM call (or a non-agentic script) cannot reliably catch routing errors the way a **dedicated verification agent** can.
- **Safety requires independent checking** — an agentic system can include a reviewer that cross-checks a proposed answer against retrieved evidence *before* it reaches the user, and can safely fall back to human review instead of guessing. This is difficult to achieve with a single-pass model or a static FAQ/keyword search.

For these reasons, a **multi-agent, tool-using, evidence-verifying system** is a better fit for this problem than either a static keyword search or a single unsupervised LLM call.

### Evidence Limitation

There is currently **no publicly available Saudi national statistic that directly measures the percentage or number of users who are routed to the wrong government service**.

Therefore, GovNavigator does **not** claim an unsupported national error rate.

Instead, this project treats the issue as a **Service Discovery & Navigation Challenge** and demonstrates an AI-based approach for understanding, retrieving, verifying, and routing users to relevant official services.

---

## 3. Proposed Solution

GovNavigator transforms a user's natural-language government request into an evidence-backed service recommendation.

### Example

**User:**

> أحتاج أفتح مؤسسة وأصدر سجل تجاري.

The system processes the request through multiple specialized stages:

```text
User Request
     ↓
Understand the problem
     ↓
Discover relevant services
     ↓
Identify responsible entity
     ↓
Verify the recommendation
     ↓
Make a routing decision
     ↓
Generate next-step plan
```

### Example Result

**Recommended Service:**  
A Commercial Registration for an Establishment

**Responsible Entity:**  
Ministry of Commerce

**Digital Channel:**  
Saudi Business Center

**Status:**  
Verified

**Next Step:**  
Proceed through the official service channel.

---

## 4. Why a Multi-Agent Architecture?

A conventional keyword search assumes that the user already knows what they are looking for.

GovNavigator starts from a different assumption:

> **The user knows their problem, but may not know the government's terminology.**

The system therefore separates the task into specialized agents.

| Agent | Responsibility |
|---|---|
| **Problem Understanding Agent** | Understands the user's actual need |
| **Service Discovery Agent** | Retrieves relevant official services |
| **Entity Routing Agent** | Identifies the responsible government entity |
| **Verification / Reviewer Agent** | Checks the proposed route against evidence |
| **Decision Agent** | Determines the final routing decision |
| **Action Planner Agent** | Generates the appropriate next-step plan |

This architecture allows each stage to focus on a defined responsibility and makes the workflow easier to verify, monitor, and control.

---

## 5. System Architecture

![GovNavigator AI System Architecture](diagrams/architecture.png)

<details>
<summary>Text-only fallback diagram</summary>

```text
                         USER
                           │
                           ▼
                 ┌───────────────────┐
                 │ Security Guardrail│
                 └─────────┬─────────┘
                           ▼
              ┌───────────────────────┐
              │ Problem Understanding │
              │        Agent          │
              └───────────┬───────────┘
                          ▼
              ┌───────────────────────┐
              │ Service Discovery     │
              │        Agent          │
              └───────────┬───────────┘
                          ▼
              ┌───────────────────────┐
              │ Entity Routing Agent  │
              └───────────┬───────────┘
                          ▼
              ┌───────────────────────┐
              │ Verification /        │
              │ Reviewer Agent        │
              └───────────┬───────────┘
                          │
                    ┌─────┴─────┐
                    │           │
                  PASS         FAIL
                    │           │
                    ▼           ▼
                Decision    Retry Discovery
                    │           │
                    ▼           │
              Action Planner ◄──┘
                    │
                    ▼
              Output Guardrail
                    │
                    ▼
                FINAL ROUTE
```
</details>

---

## 6. Agentic Workflow

![GovNavigator AI Workflow & Orchestration](diagrams/workflow.png)

GovNavigator is orchestrated using **LangGraph StateGraph**.

The workflow supports:

- Sequential execution
- Conditional routing
- Evidence verification
- Controlled retry
- Human-review fallback
- Shared workflow state

### Verification Loop

```text
Discovery
    ↓
Routing
    ↓
Verification
    │
    ├── PASS → Decision → Action Plan → Finalize
    │
    └── FAIL → Retry Discovery
                       │
                       └── Retry exhausted
                              ↓
                       Human Review
```

The verification loop is important because the system should not blindly accept an LLM-generated recommendation.

---

## 7. Reasoning Pattern

### Reviewer-Based Reflection

GovNavigator uses a **Reviewer-Based Reflection** pattern.

A dedicated Verification / Reviewer Agent evaluates whether the proposed route is supported by the retrieved evidence.

If the evidence is insufficient, the workflow can return to service discovery and attempt another retrieval cycle.

```text
Proposed Route
      ↓
Independent Verification
      │
      ├── Supported → Continue
      │
      └── Not Supported → Retry Retrieval
```

If the system cannot establish a reliable route after the allowed retry, it falls back to:

```text
NEEDS_HUMAN_REVIEW
```

This is preferable to producing an unsupported confident answer.

---

## 8. Retrieval & Knowledge Layer

The MVP uses a curated snapshot of official Saudi government services.

The retrieval layer combines:

- Keyword matching
- Alias matching
- Multilingual embeddings
- FAISS semantic retrieval

The hybrid approach allows the system to handle differences between the user's wording and the official service terminology.

### Current MVP Scope

The current prototype contains **8 curated official-service records**.

This dataset is intentionally presented as a **prototype knowledge base**, not as a complete representation of the Saudi government service catalog.

A production implementation would require continuous synchronization with authoritative government service sources and approved APIs.

---

## 9. Security & Guardrails

GovNavigator includes multiple security and safety layers.

### Input Security

- Input validation
- Prompt-injection detection
- PII masking

### Workflow Security

- Evidence-based verification
- Retry limits
- Human-review fallback
- Role-Based Access Control (RBAC)

### Output Security

The final output is validated before being returned.

The output guardrail checks that:

- Required fields are present
- Confidence is within the expected range
- The selected route exists among retrieved candidates
- The action plan is consistent with the final decision

---

## 10. Monitoring & Observability

The system records execution information to support debugging, evaluation, and future production monitoring.

Tracked information includes:

- Request ID
- Agent events
- Tool calls
- Retrieval count
- Verification status
- Confidence
- Security events
- Errors
- Execution latency

This provides visibility into the agentic workflow rather than treating the LLM as a black box.

---

## 11. Testing

The project includes tests covering:

- Normal user requests
- Invalid / short inputs
- Prompt injection
- PII masking
- RBAC
- Output guardrails
- Ambiguous requests
- Retry behavior
- Dictionary-output normalization
- Retrieval behavior
- Verification requirements

### Safe Ambiguity Handling

When the system cannot confidently determine the appropriate route, it does not force a recommendation.

Instead:

```text
NEEDS_HUMAN_REVIEW
```

This behavior is an intentional safety mechanism.

---

## 12. Why This Matters

Saudi Arabia already has a mature digital government ecosystem and centralized access points such as GOV.SA.

GovNavigator is **not another government-services portal**.

It focuses on the layer between:

```text
What the user says
        ↓
What the government service is officially called
        ↓
Which entity provides it
        ↓
Where the user should go next
```

The project's value proposition is therefore:

> **Use agentic AI to translate a user's real-world government need into a verified official service route.**

---

## 13. Future Improvements (Production Vision)

The current implementation is an MVP. Planned future improvements include integrating with approved official sources such as:

- GOV.SA service catalogs
- Government service APIs
- Real-time service information
- Official knowledge bases
- Service eligibility APIs
- Approved identity and authorization mechanisms

The system could then evolve from a curated prototype into a **continuously synchronized government-service navigation layer**.

### Potential Future Capabilities

```text
Natural-language request
        ↓
Service discovery
        ↓
Eligibility verification
        ↓
Official service routing
        ↓
Personalized next steps
        ↓
Optional authorized handoff
```

---

## 14. Current Limitations

This project is a prototype and has the following limitations:

1. The current knowledge base contains a limited number of services.
2. Government service information may change over time.
3. The prototype does not execute government transactions.
4. External API availability may affect optional components.
5. No national statistic is claimed for incorrect service routing.
6. Production deployment would require official integrations, data governance, security review, and appropriate authorization.

---

## 15. Technology Stack

| Category | Technology |
|---|---|
| Language | Python |
| Agent Orchestration | LangGraph |
| LLM | Groq — `openai/gpt-oss-120b` |
| Retrieval | Hybrid Keyword + Semantic Retrieval |
| Embeddings | Multilingual Embeddings |
| Vector Search | FAISS |
| Knowledge Base | JSON |
| Security | Prompt-Injection Detection, PII Masking, RBAC |
| Guardrails | Input & Output Validation |
| Monitoring | Runtime Logging & Observability |
| Environment | Google Colab |

---

## 16. Installation and Setup Instructions

### Requirements

- Python 3.10+ (Google Colab's default runtime works out of the box)
- A [Groq API key](https://console.groq.com) (**required** — powers all LLM agent calls)
- A [Tavily API key](https://tavily.com) (**optional** — enables the supplementary web-search tool)

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/GovNavigator.git
cd GovNavigator
```

### 2. Install dependencies

```bash
pip install -q langgraph langchain langchain-groq scikit-learn pandas requests sentence-transformers faiss-cpu
```

*(This exact command is also the first executable cell of the notebook, so running the notebook top-to-bottom installs everything automatically.)*

### 3. Configure API keys

**In Google Colab (recommended):** open the 🔑 **Secrets** panel and add:

| Secret name | Required | Purpose |
|---|---|---|
| `GROQ_API_KEY` | Yes | Powers all agent LLM calls (`openai/gpt-oss-120b` via Groq) |
| `TAVILY_API_KEY` | No | Enables the optional web-search tool in the Service Discovery agent |

**Locally / other environments:** export the same keys as environment variables before launching:

```bash
export GROQ_API_KEY="your-groq-key"
export TAVILY_API_KEY="your-tavily-key"   # optional
```

If `TAVILY_API_KEY` is not set, the system runs normally and simply skips the optional web-search tool — no other component is affected.

---

## 17. How to Run the Project

### Option A — Run the full notebook (recommended)

1. Open `notebook/GovNavigator_AI_FINAL_CORRECTED_READY.ipynb` in Google Colab.
2. Add your API key(s) via **Runtime → Secrets** as described above.
3. Select **Runtime → Restart session**, then **Runtime → Run all**.
4. The final cells run the compiled LangGraph workflow end-to-end, including the demo query, the security/PII/RBAC tests, and all regression tests — confirm every check prints as expected (`ROUTED`, `BLOCKED` for the injection test, `NEEDS_HUMAN_REVIEW` for the ambiguous test, and `PASS` for each regression test).

### Option B — Call the workflow directly

Once the notebook's setup cells have executed (state graph compiled, tools and guardrails loaded), you can invoke the system directly with a single natural-language query:

```python
result = run_govnavigator(
    "أحتاج أفتح مؤسسة وأصدر سجل تجاري",
    role="citizen"   # or "analyst" / "admin" for elevated RBAC actions
)

print(result["status"])          # ROUTED / BLOCKED / NEEDS_HUMAN_REVIEW
print(result["decision"])        # recommended service, entity, and confidence
print(result["action_plan"])     # concrete next steps for the user
print(result["logs"])            # full execution/observability trace
```

---

## 18. Project Structure

```text
GovNavigator/
│
├── notebook/
│   └── GovNavigator_AI_FINAL_CORRECTED_READY.ipynb
│
├── data/
│   └── government_services.json
│
├── diagrams/
│   ├── architecture.png
│   └── workflow.png
│
├── README.md
└── requirements.txt
```

---

## 19. Course Requirements Coverage

| Requirement | Implementation |
|---|---|
| Problem Definition | Government service discovery & navigation |
| 3+ Specialized Agents | 6 specialized agents |
| Orchestration | LangGraph StateGraph |
| Decision Point | Verification PASS / FAIL |
| Reasoning Pattern | Reviewer-Based Reflection |
| Tool Integration | Service knowledge retrieval / file reader |
| Security | Prompt injection, PII masking, RBAC, guardrails |
| Monitoring | Logs, tool calls, errors, latency, security events |
| Testing | Functional, security, ambiguity, retry, retrieval tests |
| Human-in-the-Loop | `NEEDS_HUMAN_REVIEW` fallback |

---

## 20. Project Context

Developed as a final project for:

**Advanced Agentic AI Systems Engineering**  
**SDAIA Academy**

The project demonstrates practical implementation of:

- Multi-agent systems
- Agent orchestration
- LangGraph
- Retrieval-Augmented workflows
- Tool integration
- Reviewer-based reflection
- Security guardrails
- Failure handling
- Observability
- Testing
- Production-oriented architecture

---

## 21. References

### Official Saudi Government Sources

**GOV.SA — Service Directory**  
https://my.gov.sa/en/services

**GOV.SA — Digital Government Strategy**  
https://my.gov.sa/en/content/digital-strategy

**Digital Government Authority — Digital Experience Maturity Index for Government Services 2025**  
https://dga.gov.sa/sites/default/files/2025-09/Digital%20Experience%20Maturity%20Index%20for%20Government%20Services%20%282025%29-V1.0.pdf

---

## 22. Disclaimer

GovNavigator AI is an educational and technical prototype developed to demonstrate agentic AI architecture for government-service navigation.

It is **not an official Saudi government service**, and its recommendations should not be treated as official government decisions.

For actual transactions and requirements, users should always rely on the relevant official government service and entity.

---

## ⭐ Summary

**GovNavigator AI** explores how multi-agent AI can improve the journey from a user's natural-language government need to a **verified official service route**.

The core idea is simple:

> **Understand the need. Discover the service. Verify the route. Guide the user.**

Built with **Python, LangGraph, LLMs, hybrid retrieval, security guardrails, and observability**.
