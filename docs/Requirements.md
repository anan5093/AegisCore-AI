# AegisCore AI — Product Requirements Document (PRD)

**Version:** 1.0.0
**Status:** Approved for Implementation
**Project Type:** Final-Year B.Tech Computer Science & Engineering Capstone
**Classification:** Confidential — Local Execution Only

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Stakeholders and Users](#2-stakeholders-and-users)
3. [Hardware and Environmental Constraints](#3-hardware-and-environmental-constraints)
4. [Functional Requirements](#4-functional-requirements)
5. [Non-Functional Requirements](#5-non-functional-requirements)
6. [Security Requirements](#6-security-requirements)
7. [Compliance Requirements](#7-compliance-requirements)
8. [Interface Requirements](#8-interface-requirements)
9. [Memory and Persistence Requirements](#9-memory-and-persistence-requirements)
10. [AI Firewall Requirements](#10-ai-firewall-requirements)
11. [Human-in-the-Loop Requirements](#11-human-in-the-loop-requirements)
12. [Observability and Logging Requirements](#12-observability-and-logging-requirements)
13. [Exclusions and Constraints](#13-exclusions-and-constraints)
14. [Acceptance Criteria](#14-acceptance-criteria)
15. [Glossary](#15-glossary)

---

## 1. Project Overview

### 1.1 Product Name

**AegisCore AI**

> *Aegis* — protection, shield, governance
> *Core* — local-first intelligent engine

### 1.2 Vision Statement

AegisCore AI is a fully local, multimodal, autonomous AI assistant engineered to manage a saturated personal task load — including job applications, email orchestration, calendar management, social media maintenance, academic test preparation, and content creation — while enforcing uncompromising data sovereignty, zero external PII leakage, and enterprise-grade security governance.

### 1.3 Problem Statement

Existing AI assistants (ChatGPT, Gemini, Claude) are cloud-hosted, posing severe data privacy risks when processing personally identifiable information such as resumes, academic transcripts, private correspondence, and financial data. No existing solution simultaneously provides:

- True local inference with no mandatory cloud dependency
- Autonomous multi-step task execution (not just Q&A)
- Multimodal interaction (voice + CLI)
- Enterprise-level PII protection with audit trails
- Hardware-constrained CPU-only operation

### 1.4 Proposed Solution

A locally orchestrated, multi-agent AI assistant built on LangGraph + Ollama, secured by a multi-layered AI Firewall (LiteLLM + Presidio + LLM Guard + NeMo Guardrails), governed by NIST AI RMF 1.0, ISO/IEC 42001, and OWASP Top 10 frameworks, deployed on commodity hardware without GPU dependency.

### 1.5 Project Boundaries

| In Scope | Out of Scope |
|---|---|
| Local inference on CPU | GPU-accelerated inference |
| LangGraph multi-agent orchestration | Simple sequential LLM chains |
| Voice + CLI interfaces | Mobile app or web UI |
| AI Firewall with PII redaction | Bypassing cloud with VPN alone |
| Autonomous web automation | Manual scripting without agent oversight |
| Local vector memory (ChromaDB) | Cloud-hosted vector databases |
| OWASP-compliant security layers | Ad hoc security workarounds |

---

## 2. Stakeholders and Users

### 2.1 Primary User

A final-year B.Tech CSE student operating on a constrained personal laptop with the following concurrent responsibilities:

- Active job searching across LinkedIn, Indeed, and similar platforms
- Academic examination preparation with study material ingestion
- Email and calendar management for professional correspondence
- Social media maintenance across LinkedIn and WhatsApp
- Content drafting for applications and professional communications

### 2.2 Stakeholder Roles

| Role | Responsibility |
|---|---|
| Primary User | Interacts via CLI and voice; approves all high-risk actions |
| System Architect | Defines module boundaries and security policies |
| Security Reviewer | Validates OWASP compliance and threat mitigations |
| Academic Evaluator | Assesses capstone project deliverables |

---

## 3. Hardware and Environmental Constraints

### 3.1 Target Hardware Profile

| Component | Specification |
|---|---|
| Processor | Intel Core i5, 11th Generation |
| RAM | 8 GB total system memory |
| Available RAM for AI | ~3.5 GB to 4.0 GB (after OS overhead) |
| Dedicated GPU | None |
| Operating System | Windows 11 / Windows 10 |
| Storage | Minimum 50 GB free (models + vector DB + logs) |
| Network | Required only for external API calls (firewall-gated) |

### 3.2 Model RAM Budget

| Model | Estimated RAM Usage | Use Case |
|---|---|---|
| `llama3.2:1b` (GGUF 4-bit) | ~0.8 GB | Routine tasks, email drafting |
| `llama3.2:3b` (GGUF 4-bit) | ~1.9 GB | Reasoning, form filling |
| `qwen2.5:1.5b` (GGUF 4-bit) | ~1.0 GB | Code execution, JSON parsing |
| `gemma:1b` (GGUF 4-bit) | ~0.7 GB | Fast classification tasks |
| Whisper tiny (STT) | ~75 MB | Speech recognition |
| Vosk wake word | ~500 MB | Wake-word detection |
| ChromaDB + embeddings | ~200 MB | Vector search |
| System overhead budget | ~1.5 GB | OS + background processes |

**Hard Constraint:** Total active AI memory consumption MUST NOT exceed 3.8 GB at any time.

### 3.3 Inference Performance Expectations

- CPU-bound inference: 5–15 tokens/second (acceptable for async tasks)
- Voice response latency: under 3 seconds for short queries
- Task queue throughput: one heavy task processed at a time

---

## 4. Functional Requirements

### 4.1 Core Orchestration (FR-ORCH)

| ID | Requirement |
|---|---|
| FR-ORCH-01 | The system SHALL use LangGraph as the primary orchestration framework with stateful, cyclical graph execution |
| FR-ORCH-02 | The system SHALL NOT use simple sequential LangChain chains as the primary architecture |
| FR-ORCH-03 | The system SHALL implement periodic summarization checkpoints to prevent context window overflow |
| FR-ORCH-04 | The system SHALL dynamically route tasks to the appropriate local model based on task complexity |
| FR-ORCH-05 | The system SHALL implement a Reflection (Reflexion) loop with a Planner Agent and a Critic Agent |
| FR-ORCH-06 | The Critic Agent SHALL detect hallucinations, schema violations, and policy deviations before execution |
| FR-ORCH-07 | The reflection loop SHALL retry failed tasks a maximum of 3 times before escalating to the user |
| FR-ORCH-08 | The system SHALL implement `asyncio.Queue` for sequential, FIFO task processing |
| FR-ORCH-09 | Each queued task SHALL call `task_done()` upon completion to release memory before the next task begins |

### 4.2 Local Model Execution (FR-MODEL)

| ID | Requirement |
|---|---|
| FR-MODEL-01 | All primary inference SHALL be executed locally via Ollama |
| FR-MODEL-02 | All deployed models SHALL be GGUF format with 4-bit quantization |
| FR-MODEL-03 | The system SHALL dynamically select lightweight models (1B) for routine tasks and larger models (3B) for reasoning |
| FR-MODEL-04 | External API calls (Claude, ChatGPT, Gemini) SHALL only be initiated after AI Firewall sanitization |
| FR-MODEL-05 | The inference temperature for structured JSON extraction SHALL be set at or below 0.001 |
| FR-MODEL-06 | The system SHALL enforce hard iteration limits on LangGraph execution to prevent infinite loops |

### 4.3 Web Automation and Job Applications (FR-WEB)

| ID | Requirement |
|---|---|
| FR-WEB-01 | The system SHALL use `browser-use` (built on Playwright) for autonomous web interaction |
| FR-WEB-02 | The agent SHALL accept a structured JSON payload as the canonical source of truth for user information |
| FR-WEB-03 | The agent SHALL handle cross-origin iframes, pop-ups, and dynamic form elements autonomously |
| FR-WEB-04 | The agent SHALL validate all form fields before submission |
| FR-WEB-05 | The agent SHALL pause and request explicit user approval before submitting any job application |
| FR-WEB-06 | The system SHALL support stealth browser profiles with randomized interaction timing to avoid bot detection |
| FR-WEB-07 | The system SHALL support optional proxy rotation for platforms with aggressive anti-bot measures |
| FR-WEB-08 | All web-scraped content SHALL be treated as untrusted and scanned before entering the agent's reasoning context |

### 4.4 Email Automation (FR-EMAIL)

| ID | Requirement |
|---|---|
| FR-EMAIL-01 | The system SHALL connect to email via IMAP with secure credential management |
| FR-EMAIL-02 | The agent SHALL perform entity extraction to identify sender intent, meeting times, deadlines, and action items |
| FR-EMAIL-03 | Extracted entities SHALL be output as strictly validated JSON schemas |
| FR-EMAIL-04 | The agent SHALL draft responses leveraging RAG over past sent emails for tone matching |
| FR-EMAIL-05 | The system SHALL NOT send any email without explicit user approval |
| FR-EMAIL-06 | All outbound email content SHALL be scanned by the AI Firewall before dispatch |
| FR-EMAIL-07 | All email drafts SHALL be displayed in the CLI with Rich markdown rendering before approval |

### 4.5 Calendar Management (FR-CAL)

| ID | Requirement |
|---|---|
| FR-CAL-01 | The system SHALL query calendar availability via authenticated API or local calendar store |
| FR-CAL-02 | The agent SHALL detect scheduling conflicts and apply user-defined buffer time constraints |
| FR-CAL-03 | The agent SHALL propose conflict-free scheduling slots with chronological ordering |
| FR-CAL-04 | The system SHALL require explicit user confirmation for high-priority calendar modifications |
| FR-CAL-05 | Calendar write operations (create, delete, modify) SHALL be gated by human-in-the-loop approval |

### 4.6 Social Media and External Application Integration (FR-SOCIAL)

| ID | Requirement |
|---|---|
| FR-SOCIAL-01 | WhatsApp message interception SHALL use webhook triggers within an isolated Docker container |
| FR-SOCIAL-02 | The agent SHALL perform local OCR on incoming images to extract addresses, pin codes, or text data |
| FR-SOCIAL-03 | When routing tasks to external APIs (Claude, ChatGPT, Gemini), all PII SHALL be redacted by Presidio first |
| FR-SOCIAL-04 | All social media posting actions SHALL require user approval before execution |
| FR-SOCIAL-05 | Social media interactions SHALL be processed in a Docker container isolated from the memory partition for email and financial data |

### 4.7 Academic Study Assistant (FR-STUDY)

| ID | Requirement |
|---|---|
| FR-STUDY-01 | The system SHALL ingest syllabus PDFs and textbooks into ChromaDB via a local embedding pipeline |
| FR-STUDY-02 | The study agent SHALL ONLY answer questions from uploaded material via RAG — never from hallucinated knowledge |
| FR-STUDY-03 | The system SHALL generate adaptive quizzes tracking user performance, response speed, and accuracy |
| FR-STUDY-04 | Study plans SHALL be generated in structured, digestible modules (e.g., 8-module plans) |
| FR-STUDY-05 | For high-stakes answers, the system SHALL cross-check responses across multiple local models |
| FR-STUDY-06 | The adaptive mastery algorithm SHALL dynamically increase or decrease question difficulty based on performance history |

---

## 5. Non-Functional Requirements

### 5.1 Performance (NFR-PERF)

| ID | Requirement |
|---|---|
| NFR-PERF-01 | The system SHALL process voice queries with an end-to-end response latency under 3 seconds for short inputs |
| NFR-PERF-02 | Total active RAM consumption SHALL NOT exceed 3.8 GB under any single-task scenario |
| NFR-PERF-03 | The system SHALL NOT trigger Windows page-file swapping during normal operation |
| NFR-PERF-04 | One task at a time SHALL be processed from the async queue to prevent CPU saturation |
| NFR-PERF-05 | Context summarization checkpoints SHALL activate before the context window reaches 80% capacity |

### 5.2 Reliability (NFR-REL)

| ID | Requirement |
|---|---|
| NFR-REL-01 | The system SHALL recover gracefully from OOM events without data loss or corrupted state |
| NFR-REL-02 | The LangGraph execution graph SHALL enforce maximum iteration limits to prevent runaway loops |
| NFR-REL-03 | The Critic Agent reflection loop SHALL NOT exceed 3 retry attempts per task |
| NFR-REL-04 | The system SHALL log all task failures with reason codes for post-mortem analysis |
| NFR-REL-05 | The voice pipeline SHALL degrade gracefully (fall back to CLI) if audio hardware is unavailable |

### 5.3 Maintainability (NFR-MAIN)

| ID | Requirement |
|---|---|
| NFR-MAIN-01 | The codebase SHALL follow a modular architecture with clear separation of concerns |
| NFR-MAIN-02 | All functions SHALL include full Python type hints and docstrings |
| NFR-MAIN-03 | All modules SHALL include structured logging using Python's `logging` library |
| NFR-MAIN-04 | No secrets, API keys, or credentials SHALL be hardcoded — all SHALL use environment variables |
| NFR-MAIN-05 | The project SHALL include a complete test suite covering unit, integration, and security tests |

### 5.4 Scalability (NFR-SCALE)

| ID | Requirement |
|---|---|
| NFR-SCALE-01 | New agent modules SHALL be addable without modifying core orchestration logic |
| NFR-SCALE-02 | New tools SHALL be registerable via the MCP server interface for high tool-volume scenarios |
| NFR-SCALE-03 | The memory system SHALL support namespace-isolated vector stores for different task domains |

### 5.5 Privacy (NFR-PRIV)

| ID | Requirement |
|---|---|
| NFR-PRIV-01 | All user data SHALL remain on local hardware by default |
| NFR-PRIV-02 | Any outbound data SHALL pass through the PII redaction pipeline before transmission |
| NFR-PRIV-03 | Long-term memory SHALL be stored in encrypted local vector database partitions |
| NFR-PRIV-04 | No telemetry, analytics, or usage data SHALL be sent to any external server |

---

## 6. Security Requirements

### 6.1 Prompt Injection Defense (SR-INJ) — OWASP LLM01

| ID | Requirement |
|---|---|
| SR-INJ-01 | All external inputs (emails, web content, OCR text, PDFs, social media messages) SHALL be treated as untrusted |
| SR-INJ-02 | All inputs SHALL be scanned by LLM Guard before entering the LangGraph reasoning context |
| SR-INJ-03 | NeMo Guardrails SHALL intercept all model calls to enforce topic scoping and action restrictions |
| SR-INJ-04 | Prompt injection attempts detected in parsed emails SHALL quarantine the email and alert the user |
| SR-INJ-05 | Web-scraped content SHALL be processed in a sandboxed context partition, isolated from memory writes |

### 6.2 Sensitive Information Disclosure (SR-PII) — OWASP LLM06

| ID | Requirement |
|---|---|
| SR-PII-01 | Microsoft Presidio SHALL be deployed as the primary PII detection and anonymization engine |
| SR-PII-02 | Presidio SHALL detect: names, phone numbers, email addresses, financial data, credentials, national ID numbers |
| SR-PII-03 | All detected PII SHALL be replaced with typed placeholder tokens (e.g., `<PHONE_NUMBER>`, `<ADDRESS>`) before any outbound transmission |
| SR-PII-04 | The LiteLLM proxy SHALL be the mandatory chokepoint for all external API calls |
| SR-PII-05 | De-anonymization SHALL occur locally, on the return path, before presenting results to the user |
| SR-PII-06 | Custom regex recognizers SHALL be implemented for India-specific PII formats (Aadhaar, PAN, UPI IDs) |

### 6.3 Excessive Agency (SR-AGENCY) — OWASP LLM08

| ID | Requirement |
|---|---|
| SR-AGENCY-01 | The agent SHALL NEVER operate with root or administrator system privileges |
| SR-AGENCY-02 | Role-Based Access Control (RBAC) SHALL be enforced at the tool level |
| SR-AGENCY-03 | File system access SHALL be restricted to explicitly whitelisted directories |
| SR-AGENCY-04 | Calendar write permissions SHALL default to "append only"; delete operations require user authentication |
| SR-AGENCY-05 | Email sending SHALL always require explicit user approval, regardless of context |
| SR-AGENCY-06 | OAuth 2.0 credentials used by the agent SHALL be short-lived and scoped to minimum necessary permissions |

### 6.4 Memory and Context Poisoning (SR-MEM) — OWASP ASI06

| ID | Requirement |
|---|---|
| SR-MEM-01 | A Memory Gateway SHALL validate all data before writing to the long-term vector database |
| SR-MEM-02 | The Memory Gateway SHALL route all prospective memory writes through the AI Firewall |
| SR-MEM-03 | Vector namespaces SHALL be strictly partitioned: web browsing memory SHALL NOT contaminate email, financial, or academic memory partitions |
| SR-MEM-04 | The memory store SHALL be integrity-validated on each session startup |
| SR-MEM-05 | Memories containing executable code patterns or injection strings SHALL be rejected and flagged |

### 6.5 Agent Behavior Hijacking (SR-HIJACK) — OWASP ASI01 / ASI02

| ID | Requirement |
|---|---|
| SR-HIJACK-01 | All agent tool calls SHALL be logged with a full invocation trace |
| SR-HIJACK-02 | Emails SHALL be parsed in a quarantined Docker sandbox with network isolation |
| SR-HIJACK-03 | The agent SHALL NOT execute shell commands embedded in parsed emails or web content |
| SR-HIJACK-04 | A whitelist of permitted tool calls SHALL be enforced by NeMo Guardrails Colang policies |

### 6.6 Identity and Privilege Abuse (SR-PRIV) — OWASP ASI03

| ID | Requirement |
|---|---|
| SR-PRIV-01 | The agent process SHALL run under a restricted, non-privileged OS user account |
| SR-PRIV-02 | All inter-service authentication SHALL use short-lived tokens, not persistent credentials |
| SR-PRIV-03 | Pipelock SHALL provide independent binary-level attestation and process containment |
| SR-PRIV-04 | Network egress from the agent process SHALL be restricted to explicitly whitelisted domains |

### 6.7 Insecure Output Handling (SR-OUTPUT) — OWASP LLM02

| ID | Requirement |
|---|---|
| SR-OUTPUT-01 | All LLM output SHALL be parsed through strict JSON schema validators before use in downstream logic |
| SR-OUTPUT-02 | Generated shell commands or script content SHALL NEVER be executed without human approval |
| SR-OUTPUT-03 | Output containing HTML, JavaScript, or executable patterns SHALL be sanitized before rendering |

### 6.8 Unbounded Consumption (SR-CONSUME) — OWASP LLM10

| ID | Requirement |
|---|---|
| SR-CONSUME-01 | LangGraph SHALL enforce a hard maximum iteration count per task execution graph |
| SR-CONSUME-02 | The async task queue SHALL impose a maximum queue depth limit |
| SR-CONSUME-03 | API call budgets SHALL be tracked and capped to prevent runaway external API spend |

---

## 7. Compliance Requirements

### 7.1 NIST AI RMF 1.0

| Function | Implementation Requirement |
|---|---|
| **Govern** | Documented operational boundaries, agent roles, and mandatory human-in-the-loop policies for all high-risk actions |
| **Map** | Comprehensive inventory of all local systems, files, databases, and APIs the agent can access, with attack surface analysis |
| **Measure** | AI Firewall telemetry and anomaly detection used to evaluate system safety and performance continuously |
| **Manage** | Continuous update policy for LLM Guard rules and NeMo Guardrails based on measured metrics and emerging threats |

### 7.2 ISO/IEC 42001

The system SHALL maintain an auditable AI management system including:

- Documented policies for each security component
- Certifiable evidence of Zero Trust enforcement
- Workload identity assigned to the agent for all local API calls
- Access control documentation for every file system and calendar endpoint

### 7.3 CSA AI Controls Matrix (AICM)

The system SHALL align with applicable AICM controls covering:

- Data governance and classification
- AI model lifecycle management
- Access management for AI-driven automation
- Incident detection and response for agentic systems

### 7.4 OWASP Top 10 for LLM Applications (2025)

The system SHALL document explicit mitigations for: LLM01, LLM02, LLM06, LLM08, and LLM10 (detailed in Section 6).

### 7.5 OWASP Top 10 for Agentic Applications (2026)

The system SHALL document explicit mitigations for: ASI01, ASI02, ASI03, and ASI06 (detailed in Section 6).

---

## 8. Interface Requirements

### 8.1 Command Line Interface (IR-CLI)

| ID | Requirement |
|---|---|
| IR-CLI-01 | The CLI SHALL be built with Typer using Python type hints for automatic command parsing |
| IR-CLI-02 | The CLI SHALL use Rich for progress bars, markdown rendering, colorized output, and telemetry tables |
| IR-CLI-03 | Commands SHALL follow the pattern: `assistant <module> <action> [options]` |
| IR-CLI-04 | Example commands SHALL include: `assistant apply-jobs --platform linkedin --resume ./cv.pdf` |
| IR-CLI-05 | The CLI SHALL display real-time agent state, current graph node, and tool invocations |
| IR-CLI-06 | Approval prompts for high-risk actions SHALL be rendered clearly with risk level indicators |

### 8.2 Voice Interface (IR-VOICE)

| ID | Requirement |
|---|---|
| IR-VOICE-01 | Wake-word detection SHALL use Vosk with a custom trigger phrase, consuming under 500 MB RAM |
| IR-VOICE-02 | Speech-to-text SHALL use `whisper.cpp` with the "tiny" model (under 75 MB) |
| IR-VOICE-03 | Text-to-speech SHALL use Piper or Kokoro, both CPU-optimized and fully offline |
| IR-VOICE-04 | The voice pipeline SHALL operate over a local WebSocket server with streaming audio chunks |
| IR-VOICE-05 | Audio processing SHALL use `sounddevice` for microphone I/O |
| IR-VOICE-06 | Voice responses SHALL be encoded as Opus and played back with minimal latency |
| IR-VOICE-07 | The system SHALL NOT use ElevenLabs, Google Cloud Speech, or any cloud voice API |

---

## 9. Memory and Persistence Requirements

### 9.1 Short-Term Memory

- Implemented via LangGraph state (in-memory, session-scoped)
- Contains: current task state, active tool context, temporary reasoning traces
- Cleared at task completion

### 9.2 Long-Term Memory

- Implemented via Mem0 or LangMem + ChromaDB
- Contains: user preferences, tone profiles, scheduling habits, frequent contacts, task history
- Extracted by a background memory agent at session end
- Stored as encrypted vector embeddings in isolated ChromaDB namespaces
- Injected into system prompt context at session start via similarity search

### 9.3 Context Management

- Rolling summaries SHALL replace raw logs once a task completes
- Semantic compression SHALL reduce completed workflow sections to dense summaries
- Context injection SHALL be retrieval-based, not full-history replay
- Context window utilization SHALL be monitored; summarization SHALL trigger at 80% capacity

---

## 10. AI Firewall Requirements

### 10.1 LiteLLM Proxy

- Mandatory chokepoint for all external LLM API calls
- Provides: routing, request interception, audit logging, rate limiting
- All requests and responses SHALL be logged with timestamps, model used, and token counts

### 10.2 Microsoft Presidio

- Analyzer module SHALL use spaCy NLP backend with custom regex recognizers
- Shall detect: PERSON, PHONE_NUMBER, EMAIL_ADDRESS, ADDRESS, CREDIT_CARD, IN_AADHAAR, IN_PAN
- Anonymizer SHALL apply token replacement operators (not masking with `***`)
- De-anonymization SHALL restore original entities on the return path before local display
- Custom operators SHALL be implemented for India-specific data formats

### 10.3 LLM Guard

- Input scanners SHALL detect: prompt injection, jailbreak attempts, toxic language
- Output scanners SHALL detect: sensitive data in responses, code injection, policy violations
- SHALL operate on CPU with acceptable latency (under 200ms per scan)

### 10.4 NeMo Guardrails

- Colang policy files SHALL define: allowed topics, permitted tool calls, disallowed actions
- Guardrails SHALL intercept execution BEFORE and AFTER each LangGraph model call
- Guardrails violations SHALL be logged and escalated to the user

### 10.5 Pipelock (Network-Level Containment)

- Independent binary monitoring outbound network requests from the agent process
- SHALL enforce a whitelist of permitted external domains
- SHALL provide independently verifiable proof of every network action executed

---

## 11. Human-in-the-Loop Requirements

The system SHALL ALWAYS pause and require explicit user approval before executing the following actions:

| Action Category | Approval Mechanism |
|---|---|
| Sending any email | CLI prompt with full draft displayed |
| Submitting a job application | CLI prompt with form field summary |
| Posting to social media | CLI prompt with post content preview |
| Modifying or deleting calendar events | CLI prompt with event details |
| Deleting any local file | CLI prompt with file path and size |
| Executing generated shell commands | CLI prompt with command and risk assessment |
| Routing a prompt to external API | CLI prompt with sanitized payload preview |

There SHALL be no override mechanism that bypasses human approval for any of the above categories.

---

## 12. Observability and Logging Requirements

| Log Type | Contents | Retention |
|---|---|---|
| Execution Trace | LangGraph node transitions, tool invocations, model calls | 30 days |
| Security Event Log | Firewall blocks, injection attempts, policy violations | 90 days |
| Audit Trail | All human approval events, external API calls | Indefinite |
| Task Log | Task status, start/end times, failure reasons | 30 days |
| Memory Log | Memory writes, rejections by Memory Gateway | 30 days |

All logs SHALL:

- Remain stored locally (no external log shipping)
- Be structured as JSON for machine-parseable analysis
- Be rotated and compressed to prevent unbounded disk usage
- Include timestamps in ISO 8601 format

---

## 13. Exclusions and Constraints

The following are explicitly out of scope or prohibited:

- GPU acceleration (system must function without dedicated GPU)
- Cloud-hosted vector databases (ChromaDB must run locally)
- Cloud-hosted voice APIs (ElevenLabs, Google Cloud Speech, AWS Polly)
- Root or administrator execution of the agent process
- Hardcoded API keys or credentials in source code
- Sequential LangChain chains as the primary orchestration mechanism
- Browser automation without human approval for form submission
- Memory writes without Memory Gateway validation
- External API calls without Presidio PII redaction

---

## 14. Acceptance Criteria

The system SHALL be considered complete when:

1. All LangGraph agent workflows complete without triggering OOM errors on target hardware
2. The AI Firewall correctly redacts 100% of seeded PII in test payloads before external API calls
3. Human-in-the-loop approval is demonstrably required for all actions in Section 11
4. Voice pipeline achieves end-to-end response under 3 seconds for short utterances
5. The study assistant answers questions exclusively from uploaded documents (RAG-only, zero hallucination)
6. OWASP LLM01, LLM02, LLM06, LLM08, LLM10, ASI01, ASI02, ASI03, ASI06 mitigations are documented and tested
7. All modules pass unit and integration tests with >80% code coverage
8. A complete threat model document is produced and reviewed

---

## 15. Glossary

| Term | Definition |
|---|---|
| GGUF | A binary format for quantized LLM models optimized for CPU inference |
| RAG | Retrieval-Augmented Generation — grounding model responses in retrieved documents |
| RBAC | Role-Based Access Control — restricting operations based on assigned roles |
| PII | Personally Identifiable Information — any data that can identify an individual |
| TEE | Trusted Execution Environment — a secure enclave protecting computation from external access |
| Colang | NeMo Guardrails' policy definition language |
| LangGraph | A stateful, cyclical multi-agent orchestration framework by LangChain |
| Ollama | A local inference engine for running quantized LLMs on CPU |
| Presidio | Microsoft's open-source PII detection and anonymization library |
| LiteLLM | An open-source LLM proxy providing unified API access and request interception |
| Pipelock | An agent-external binary monitoring tool for network containment and process attestation |
| Memory Gateway | The validation layer that filters all candidate data before it is written to long-term memory |
| ASI | Agentic Security Incident — OWASP's classification for agentic-specific vulnerabilities |
