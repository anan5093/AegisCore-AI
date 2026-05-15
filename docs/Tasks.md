# AegisCore AI — Task Breakdown Document

**Version:** 1.0.0
**Methodology:** Spec-Driven Development
**Project Timeline:** 16 Weeks (Final-Year Capstone)

---

## Table of Contents

1. [Development Methodology](#1-development-methodology)
2. [Phase Overview](#2-phase-overview)
3. [Phase 0 — Specification and Environment Setup](#3-phase-0--specification-and-environment-setup)
4. [Phase 1 — Core Infrastructure](#4-phase-1--core-infrastructure)
5. [Phase 2 — AI Firewall and Security Stack](#5-phase-2--ai-firewall-and-security-stack)
6. [Phase 3 — LangGraph Orchestration Core](#6-phase-3--langgraph-orchestration-core)
7. [Phase 4 — Memory System](#7-phase-4--memory-system)
8. [Phase 5 — Multimodal Interfaces](#8-phase-5--multimodal-interfaces)
9. [Phase 6 — Autonomous Task Modules](#9-phase-6--autonomous-task-modules)
10. [Phase 7 — Security Hardening and Compliance](#10-phase-7--security-hardening-and-compliance)
11. [Phase 8 — Integration, Testing, and Documentation](#11-phase-8--integration-testing-and-documentation)
12. [Task Dependency Graph](#12-task-dependency-graph)
13. [Definition of Done](#13-definition-of-done)
14. [Risk Register](#14-risk-register)

---

## 1. Development Methodology

### 1.1 Spec-Driven Development

Per the project's engineering standards, no implementation code is written until the following artifacts exist and are reviewed:

1. ✅ Product Requirements Document (Requirements.md)
2. ✅ System Design Document (Design.md)
3. ✅ Task Breakdown Document (Tasks.md — this document)
4. ⬜ Module-level API contracts (inline in Design.md)
5. ⬜ Threat Model (inline in Design.md, Section 13)
6. ⬜ Test plan (Section 11 of this document)

### 1.2 Task Notation

Each task follows this format:

```
[TASK-ID] Task Title
  Status:   [ ] Not Started | [~] In Progress | [x] Complete | [!] Blocked
  Priority: P0 (Critical Path) | P1 (High) | P2 (Medium) | P3 (Low)
  Effort:   S (< 4 hours) | M (4–8 hours) | L (1–2 days) | XL (3–5 days)
  Depends:  List of prerequisite task IDs
  Owner:    Developer
  Notes:    Implementation-specific guidance
```

### 1.3 Coding Standards (enforced on all tasks)

Every code file produced MUST:

- Use Python 3.11+ type hints on all functions
- Include module-level and function-level docstrings
- Use `logging` (structured JSON via `python-json-logger`)
- Load secrets from environment variables — zero hardcoded values
- Include `try/except` with specific exception types (no bare `except:`)
- Include retry logic for all network and I/O operations
- Follow the modular project structure defined in Design.md

---

## 2. Phase Overview

| Phase | Name | Weeks | Focus |
|---|---|---|---|
| Phase 0 | Specification and Setup | 1 | Docs, environment, tooling |
| Phase 1 | Core Infrastructure | 2–3 | Ollama, project scaffold, Docker |
| Phase 2 | AI Firewall Stack | 3–4 | Presidio, LiteLLM, LLM Guard, NeMo |
| Phase 3 | LangGraph Orchestration | 4–6 | Agent graph, reflection, queue |
| Phase 4 | Memory System | 6–7 | ChromaDB, Mem0, Memory Gateway |
| Phase 5 | Multimodal Interfaces | 7–9 | CLI (Typer+Rich), Voice pipeline |
| Phase 6 | Autonomous Task Modules | 9–13 | Email, Calendar, Web, Study, Social |
| Phase 7 | Security Hardening | 13–14 | RBAC, Zero Trust, OWASP audits |
| Phase 8 | Integration and Testing | 14–16 | E2E tests, documentation, demo |

---

## 3. Phase 0 — Specification and Environment Setup

**Goal:** Establish the project foundation before writing a single line of implementation code.

---

### TASK-0-01: Create Project Repository and Directory Structure

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  None
```

**Steps:**

- Initialize Git repository with `.gitignore` (include `.env`, `logs/`, `data/`, `*.gguf`)
- Create the full directory scaffold from Design.md Section 2
- Add `pyproject.toml` with initial dependency declarations
- Add `.env.example` with all required environment variable keys (no values)
- Commit initial scaffold

**Acceptance:** `tree aegiscore/` matches Design.md structure exactly

---

### TASK-0-02: Configure Python Environment

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-0-01
```

**Steps:**

- Install Python 3.11+ (verify with `python --version`)
- Create virtual environment: `python -m venv .venv`
- Install base dependencies: `langchain`, `langgraph`, `ollama`, `chromadb`, `mem0`, `presidio-analyzer`, `presidio-anonymizer`, `litellm`, `nemo-guardrails`, `llm-guard`, `typer`, `rich`, `pydantic`, `python-dotenv`, `python-json-logger`, `pytest`, `pytest-asyncio`
- Freeze requirements: `pip freeze > requirements.txt`
- Verify all imports succeed in a `smoke_test.py`

**Acceptance:** `python smoke_test.py` runs without import errors

---

### TASK-0-03: Install and Configure Ollama

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-0-02
```

**Steps:**

- Download and install Ollama for Windows from `ollama.com`
- Pull required models (run each, verify RAM usage with Task Manager):
  - `ollama pull llama3.2:1b`
  - `ollama pull llama3.2:3b`
  - `ollama pull qwen2.5:1.5b`
  - `ollama pull gemma:1b`
  - `ollama pull nomic-embed-text` (for local embeddings)
- Verify all models run with `ollama run llama3.2:1b "Hello"`
- Document observed RAM usage for each model pull

**Acceptance:** All 5 models listed in `ollama list`; RAM usage within budget from Requirements.md Table 3.2

---

### TASK-0-04: Configure Docker Environment

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-01
```

**Steps:**

- Install Docker Desktop for Windows
- Enable WSL 2 backend
- Create `docker/Dockerfile.agent` with non-root user (`useradd -u 1001`)
- Create `docker/Dockerfile.sandbox` with `--network none` configuration
- Create `docker/docker-compose.yml` with `internal_net` bridge (no external connectivity)
- Test: `docker-compose up --build` and verify containers start
- Verify agent container cannot reach `8.8.8.8` (network isolation test)

**Acceptance:** `docker-compose ps` shows all services healthy; `docker exec aegiscore-agent ping 8.8.8.8` times out

---

### TASK-0-05: Create Structured Logging Framework

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-0-02
```

**Steps:**

- Implement `security/audit_logger.py` with JSON-structured log output
- Define log categories: `execution`, `security`, `audit`, `memory`
- Configure log rotation (30-day default, 90-day for security logs)
- Implement `create_audit_entry()` function per Design.md Section 9.4
- Write unit test: verify audit entry schema matches defined format

**Acceptance:** Test passes; log file written to `logs/audit/` with correct JSON schema

---

## 4. Phase 1 — Core Infrastructure

**Goal:** Build the foundational infrastructure all other modules depend on.

---

### TASK-1-01: Implement Ollama Router

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-03
```

**File:** `agents/router.py`

**Steps:**

- Implement `OllamaRouter` class per Design.md Section 4.2
- Load model routing config from `configs/models.yaml`
- Implement `infer(task_type, prompt, stream)` with async support
- Implement RAM headroom check: if available RAM below 500MB threshold, downgrade to 1B model automatically
- Add retry logic: 3 retries with exponential backoff on Ollama connection failure
- Write unit tests: mock Ollama client, verify correct model selected per task type

**Acceptance:** Unit tests pass; routing correctly selects `qwen2.5:1.5b` for `json_extraction` task type

---

### TASK-1-02: Define LangGraph State Schema

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-0-02
```

**File:** `agents/supervisor.py`

**Steps:**

- Implement `AegisCoreState` TypedDict per Design.md Section 3.1
- Add all fields: `task_id`, `messages`, `user_profile`, `iteration_count`, `max_iterations`, `pending_approval`, `approval_status`, `reflection_errors`, `retry_count`, `final_output`, `error`
- Write helper functions: `create_initial_state(task_type, user_intent)`, `increment_iteration(state)`, `set_pending_approval(state, action_payload)`
- Write unit tests for each helper function

**Acceptance:** State dict serializes/deserializes correctly; helper functions pass all unit tests

---

### TASK-1-03: Implement Async Task Queue

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-1-02
```

**File:** `agents/executor.py`

**Steps:**

- Implement `asyncio.Queue`-based FIFO task queue
- Implement producer interface: `enqueue_task(task_type, user_intent, metadata)`
- Implement background worker: pulls tasks one-at-a-time, calls LangGraph graph
- Implement `task_done()` call on every completion path (success, failure, rejection)
- Implement queue depth limit (max 10 tasks queued)
- Add status display: queue depth, current task description, elapsed time
- Write integration test: enqueue 3 tasks, verify sequential execution (not concurrent)

**Acceptance:** Integration test passes; memory profiler shows no simultaneous heavy task execution

---

### TASK-1-04: Implement RBAC System

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-01
```

**File:** `security/rbac.py`

**Steps:**

- Implement `RBACPolicy` class loading from `configs/rbac_policies.yaml`
- Implement `check_permission(role: str, action: str, path: Optional[str])` returning `bool`
- Implement `require_permission` decorator for tool functions
- Implement file system path allowlist enforcement
- Write unit tests covering: allowed action, denied action, allowed path, denied path
- Write security test: verify `calendar.delete_event` is always denied without elevated role

**Acceptance:** All unit and security tests pass

---

### TASK-1-05: Implement Zero Trust Middleware

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-1-04
```

**File:** `security/zero_trust.py`

**Steps:**

- Implement request authentication wrapper for all internal service calls
- Implement short-lived token generation for Calendar API (15-minute expiry)
- Implement OS keychain integration for IMAP credentials (using `keyring` library)
- Implement service-scoped API key rotation for ChromaDB
- Write integration test: verify expired token is rejected and refreshed correctly

**Acceptance:** Token expiry test passes; all service calls require authentication

---

## 5. Phase 2 — AI Firewall and Security Stack

**Goal:** Build the security layer before any external-facing functionality. Security is not retrofitted.

---

### TASK-2-01: Deploy and Configure LiteLLM Proxy

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-02
```

**File:** `firewall/litellm_proxy.py`

**Steps:**

- Install LiteLLM: `pip install litellm`
- Configure proxy server with `litellm --config configs/litellm_config.yaml`
- Set supported models: Anthropic Claude, OpenAI GPT, Google Gemini
- Configure audit logging: every request logged with timestamp, model, token count
- Implement API budget limits (monthly cap per provider)
- Write integration test: verify request is intercepted and logged before reaching external API

**Acceptance:** Proxy intercepts 100% of external API calls; audit log entry created for each

---

### TASK-2-02: Implement Microsoft Presidio — PII Detection

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-2-01
```

**File:** `firewall/presidio_engine.py`

**Steps:**

- Install Presidio: `pip install presidio-analyzer presidio-anonymizer`
- Install spaCy model: `python -m spacy download en_core_web_lg`
- Configure standard entity recognizers: `PERSON`, `PHONE_NUMBER`, `EMAIL_ADDRESS`, `LOCATION`, `CREDIT_CARD`, `URL`
- Implement `PresidioEngine.analyze(text)` returning detected entities with confidence scores
- Implement `PresidioEngine.anonymize(text)` applying typed token replacement
- Implement `PresidioEngine.deanonymize(text, mapping)` for return path restoration
- Write unit tests with seeded PII: verify 100% detection of seeded entities in test payloads

**Acceptance:** Unit tests pass; 100% of seeded PII detected and replaced in test suite

---

### TASK-2-03: Implement India-Specific PII Recognizers

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-2-02
```

**File:** `firewall/presidio_operators.py`

**Steps:**

- Implement custom `PatternRecognizer` for Aadhaar number (12-digit, space-separated): `\d{4}\s\d{4}\s\d{4}`
- Implement custom `PatternRecognizer` for PAN card: `[A-Z]{5}[0-9]{4}[A-Z]`
- Implement custom `PatternRecognizer` for UPI ID: `[\w.-]+@[\w]+`
- Implement custom `PatternRecognizer` for Indian mobile numbers: `(\+91|0)?[6-9]\d{9}`
- Register all custom recognizers with Presidio `AnalyzerEngine`
- Write unit tests with sample Indian PII strings for each recognizer

**Acceptance:** All custom recognizers detect sample PII in unit tests with >95% precision

---

### TASK-2-04: Implement LLM Guard Input and Output Scanners

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-02
```

**File:** `firewall/llm_guard_scanner.py`

**Steps:**

- Install: `pip install llm-guard`
- Implement `LLMGuardScanner.scan_input(prompt)` using: `PromptInjectionScanner`, `BanTopicsScanner`, `ToxicityScanner`
- Implement `LLMGuardScanner.scan_output(response)` using: `SensitiveInformationScanner`, `BanSubstringsScanner`, `CodeScanner`
- Implement scan result dataclass: `ScanResult(is_malicious: bool, reason: str, risk_score: float)`
- Benchmark scan latency on target hardware — must be under 200ms per scan
- Write security tests: inject known prompt injection payloads, verify detection

**Acceptance:** Known injection payloads in test suite are 100% detected; scan latency under 200ms

---

### TASK-2-05: Implement NeMo Guardrails Integration

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-0-02
```

**File:** `firewall/nemo_guardrails.py`

**Steps:**

- Install: `pip install nemoguardrails`
- Write Colang policy files in `configs/guardrails/policies.co` per Design.md Section 6.3
- Implement policies for: off-topic requests, dangerous actions, jailbreak attempts, disallowed tool calls
- Integrate guardrails as LangChain middleware that intercepts pre/post model calls
- Write integration tests: send known jailbreak phrases, verify `bot refuse jailbreak` response triggered

**Acceptance:** Integration tests pass; jailbreak test phrases are blocked by guardrails

---

### TASK-2-06: Build Unified AI Firewall Interface

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-2-01, TASK-2-02, TASK-2-03, TASK-2-04, TASK-2-05
```

**File:** `firewall/__init__.py` (AIFirewall class)

**Steps:**

- Implement `AIFirewall` class composing: `PresidioEngine`, `LLMGuardScanner`, `NemoGuardrails`, `LiteLLMProxy`
- Implement unified `scan_inbound(content)` → applies LLM Guard + NeMo pre-check
- Implement unified `sanitize_outbound(prompt)` → applies Presidio anonymization + LiteLLM routing
- Implement unified `validate_response(response)` → applies LLM Guard output scan + NeMo post-check + Presidio de-anonymization
- Write end-to-end test: input with PII and injection attempt → verify both are handled correctly

**Acceptance:** E2E test passes; PII redacted and injection blocked in single request flow

---

## 6. Phase 3 — LangGraph Orchestration Core

**Goal:** Build the agent brain — planning, reflection, routing, and approval workflow.

---

### TASK-3-01: Implement Planner Agent Node

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-1-01, TASK-1-02, TASK-2-06
```

**File:** `agents/planner.py`

**Steps:**

- Implement `planner_node(state: AegisCoreState, llm, firewall)` as an async LangGraph node
- System prompt: include user profile, retrieved memories, available tools list, safety constraints
- Generate structured task plan as JSON: `{ steps: [...], tools_required: [...], risk_level: "low|medium|high" }`
- Validate plan JSON against Pydantic schema before returning
- If risk_level is `high`, automatically set `pending_approval = True`
- Log node entry/exit with task_id to execution logger

**Acceptance:** Planner generates valid JSON plan for 5 diverse test task types

---

### TASK-3-02: Implement Critic Agent Node

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-3-01
```

**File:** `agents/critic.py`

**Steps:**

- Implement `critic_node(state: AegisCoreState, llm)` as an async LangGraph node
- System prompt: Constitutional AI prompt with operational rules, safety boundaries, hallucination detection instructions
- Critic reviews planner output for: logical errors, hallucinations, policy violations, schema errors, security risks
- Output: `CriticResult(approved: bool, errors: List[str], risk_assessment: str)`
- If `approved = False`: increment `retry_count`, append errors to `reflection_errors`, route back to planner
- If `retry_count >= 3`: route to `escalate_to_user` terminal state
- Write test: feed planner output with deliberate hallucination, verify critic catches it

**Acceptance:** Critic correctly identifies seeded errors in test cases; retry counter increments correctly

---

### TASK-3-03: Implement Human Approval Checkpoint

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-3-01, TASK-3-02
```

**File:** `agents/supervisor.py` (approval_checkpoint_node)

**Steps:**

- Implement `approval_checkpoint_node(state: AegisCoreState)` that pauses LangGraph execution
- Render approval prompt via Rich: display action description, risk level, full details
- For CLI: use `typer.confirm()` with clear YES/NO prompt
- Log: `HUMAN_APPROVAL_REQUESTED` audit event with action details
- On approval: log `HUMAN_APPROVAL_GRANTED`, continue graph
- On rejection: log `HUMAN_APPROVAL_REJECTED`, route to terminal state
- Write test: mock user input "n" (reject), verify graph terminates without executing action

**Acceptance:** Rejection test passes; no action executed after rejection

---

### TASK-3-04: Implement Iteration Guard and OOM Protection

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-3-02
```

**File:** `agents/supervisor.py` (iteration_guard function)

**Steps:**

- Implement `iteration_guard(state: AegisCoreState)` routing function per Design.md Section 3.3
- Hard cap: 10 iterations maximum per task graph execution
- Add RAM monitoring: check available system RAM via `psutil`; if below 400MB, pause queue and alert user
- Add execution time limit: if task exceeds 10 minutes, prompt user to continue or abort
- Write test: create graph that would loop indefinitely, verify it terminates at iteration 10

**Acceptance:** Loop termination test passes at exactly 10 iterations

---

### TASK-3-05: Build Complete LangGraph Agent Graph

```
Status:   [ ]
Priority: P0
Effort:   XL
Depends:  TASK-3-01, TASK-3-02, TASK-3-03, TASK-3-04, TASK-1-03
```

**File:** `agents/supervisor.py`

**Steps:**

- Define all graph nodes: `intent_classifier`, `memory_retrieval`, `planner`, `critic`, `approval_checkpoint`, `executor`, `output_validator`, `memory_update`, `summarizer`
- Define all edges and conditional routing using `iteration_guard`, `approval_status`, `retry_count`
- Connect all edges per the graph topology in Design.md Section 3.2
- Compile graph: `graph = StateGraph(AegisCoreState).compile()`
- Run end-to-end smoke test with a simple task: "Draft a reply to this email" (no actual email sending)
- Verify all nodes execute in correct order via log output

**Acceptance:** Smoke test produces correct node execution sequence in logs; no exceptions thrown

---

## 7. Phase 4 — Memory System

**Goal:** Build persistent, secure, and namespace-isolated memory for the agent.

---

### TASK-4-01: Configure ChromaDB with Namespace Isolation

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-0-02
```

**File:** `memory/vector_store.py`

**Steps:**

- Install: `pip install chromadb`
- Implement `ChromaNamespacedStore` class wrapping ChromaDB client
- Create collections per Design.md Section 12.1: `user_preferences`, `task_history`, `email_history`, `study_materials`, `web_browsing`
- Implement `write(content, namespace, metadata)` — writes only to specified namespace
- Implement `search(query, namespace, top_k=5)` — searches only specified namespace
- Implement namespace isolation enforcement: `web_browsing` namespace cannot be read by `write` to other namespaces
- Write test: write to `web_browsing`, attempt to read in `user_preferences` context, verify isolation

**Acceptance:** Namespace isolation test passes; cross-namespace contamination prevented

---

### TASK-4-02: Implement Embedding Pipeline

```
Status:   [ ]
Priority: P0
Effort:   S
Depends:  TASK-4-01, TASK-0-03
```

**File:** `memory/embeddings.py`

**Steps:**

- Implement `LocalEmbedder` using Ollama's `nomic-embed-text` model via `ollama.embeddings()`
- Implement `embed_text(text)` returning normalized float vector
- Implement `embed_batch(texts)` for efficient bulk ingestion
- Benchmark embedding latency on target hardware; document tokens/second
- Write unit test: embed same text twice, verify cosine similarity > 0.99

**Acceptance:** Unit test passes; embedding latency documented

---

### TASK-4-03: Implement Memory Gateway

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-4-01, TASK-2-06
```

**File:** `memory/memory_gateway.py`

**Steps:**

- Implement `MemoryGateway` class per Design.md Section 5.2
- Step 1: LLM Guard input scan on candidate content
- Step 2: Executable pattern detection (regex for shell commands, Python `exec()`, `eval()`, etc.)
- Step 3: Presidio anonymization before storage
- Step 4: Write to isolated namespace with metadata including `validated_by_gateway: True`
- Write security test: attempt to write content containing `os.system("rm -rf /")`, verify rejection
- Write security test: attempt to write content containing prompt injection, verify rejection

**Acceptance:** Both security tests pass; malicious content never reaches ChromaDB

---

### TASK-4-04: Implement Mem0 Long-Term Memory Integration

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-4-03
```

**File:** `memory/long_term.py`

**Steps:**

- Install: `pip install mem0ai`
- Configure Mem0 to use local ChromaDB and local embedding model (no cloud API calls)
- Implement `LongTermMemory.extract_and_store(conversation_messages)` — background agent that parses conversation and extracts preferences
- Implement `LongTermMemory.retrieve_context(current_intent)` — similarity search returning top-5 relevant memories
- Integrate retrieval into `memory_retrieval_node` in LangGraph graph
- Write integration test: store a preference ("user prefers bullet-point emails"), retrieve it in a new session

**Acceptance:** Integration test passes; preference retrieved correctly in simulated new session

---

### TASK-4-05: Implement Checkpoint Summarizer

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-1-01, TASK-4-01
```

**File:** `memory/summarizer.py`

**Steps:**

- Implement `summarizer_node(state, llm)` per Design.md Section 3.4
- Monitor context window utilization (token count relative to model's `num_ctx` limit)
- Trigger summarization when utilization reaches 80%
- Compress completed message history into structured JSON summary
- Store summary in `task_history` namespace via Memory Gateway
- Replace raw messages in state with compressed summary message
- Write test: fill context to 80%, verify summarization triggers and message count drops

**Acceptance:** Summarization triggers at 80% and message list is compressed correctly

---

## 8. Phase 5 — Multimodal Interfaces

**Goal:** Build the CLI and voice pipeline — the user's windows into the system.

---

### TASK-5-01: Implement Typer CLI Application

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  TASK-1-03
```

**File:** `cli/main.py` and `cli/commands/`

**Steps:**

- Install: `pip install typer rich`
- Create root Typer app with `app = typer.Typer(name="assistant")`
- Implement `apply-jobs` command: `--platform (linkedin|indeed)`, `--resume PATH`, `--dry-run FLAG`
- Implement `email` command group: `parse`, `draft`, `send`
- Implement `calendar` command group: `check`, `schedule`
- Implement `study` command group: `ingest --pdf PATH`, `quiz --topic TEXT`, `plan --modules INT`
- Implement `status` command: displays queue depth, active model, RAM usage
- Add shell completion: `assistant --install-completion`
- Write CLI test: invoke each command with `typer.testing.CliRunner`, verify exit code 0

**Acceptance:** All commands invoke without error; `--help` displays correct options for each command

---

### TASK-5-02: Implement Rich Display Utilities

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-5-01
```

**File:** `cli/display.py`

**Steps:**

- Implement `display_approval_prompt(action, details, risk_level)` — Rich Panel with color-coded risk
- Implement `display_email_draft(draft, recipient, subject)` — Rich Markdown render
- Implement `display_task_progress(task_name)` — Rich Progress context manager
- Implement `display_agent_status(queue_depth, current_task, ram_mb)` — Rich Table
- Implement `display_security_alert(event_type, details)` — Bold red Rich Panel
- Implement streaming token display: `stream_agent_response(token_iterator)` — Rich Live

**Acceptance:** Visual review confirms all components render correctly in Windows terminal

---

### TASK-5-03: Implement Voice Pipeline — Wake Word Detection

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-0-02
```

**File:** `voice/wake_word.py`

**Steps:**

- Install Vosk: `pip install vosk`
- Download Vosk small English model (40MB) to `configs/vosk_model/`
- Implement `WakeWordDetector(trigger_phrase: str)` class
- Implement `start_listening()` — streams microphone in background thread
- Implement callback: on trigger phrase detected, emit event to `asyncio.Queue`
- Benchmark: verify Vosk RAM usage stays below 500MB on target hardware
- Write test: play pre-recorded trigger phrase WAV, verify callback fires

**Acceptance:** Callback fires within 500ms of trigger phrase in WAV file playback test; RAM under 500MB

---

### TASK-5-04: Implement Voice Pipeline — Speech-to-Text

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-5-03
```

**File:** `voice/stt.py`

**Steps:**

- Download and compile `whisper.cpp` with tiny model (39MB GGML format)
- Install Python bindings: `pip install pywhispercpp`
- Implement `WhisperSTT.transcribe(audio_bytes)` returning text string
- Implement voice turn detection: silence detection using RMS threshold to segment speech turns
- Implement streaming mode: process audio in 30-second chunks
- Write test: transcribe pre-recorded 10-second WAV, verify >90% word accuracy

**Acceptance:** Transcription accuracy >90% on test WAV file; latency under 2 seconds

---

### TASK-5-05: Implement Voice Pipeline — Text-to-Speech

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-0-02
```

**File:** `voice/tts.py`

**Steps:**

- Install Piper: download pre-compiled Piper binary and English voice model
- Implement `PiperTTS.synthesize(text)` returning Opus-encoded audio bytes
- Implement streaming synthesis: chunk long responses into sentences for faster first-audio latency
- Implement `sounddevice` playback: stream Opus audio to speaker output
- Write test: synthesize 3-sentence text, measure latency to first audio output

**Acceptance:** First audio plays within 1 second of synthesis call; audio quality is intelligible

---

### TASK-5-06: Build Local WebSocket Voice Server

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-5-03, TASK-5-04, TASK-5-05
```

**File:** `voice/websocket_server.py` and `voice/audio_io.py`

**Steps:**

- Implement local WebSocket server on `ws://localhost:8765`
- Audio I/O: `sounddevice` captures microphone → streams chunks to server
- Server pipeline: audio chunk → Vosk wake word → Whisper STT → text to `asyncio.Queue`
- Response pipeline: LangGraph output → Piper TTS → Opus audio → WebSocket stream → speaker
- Implement graceful degradation: if voice hardware unavailable, fall back to CLI-only mode
- Write integration test: simulate full voice round-trip with WAV input file

**Acceptance:** Integration test completes full round-trip; CLI fallback activates when no microphone detected

---

## 9. Phase 6 — Autonomous Task Modules

**Goal:** Build the individual capability modules that deliver real-world utility.

---

### TASK-6-01: Implement Job Application Web Automation

```
Status:   [ ]
Priority: P1
Effort:   XL
Depends:  TASK-3-05, TASK-2-06
```

**File:** `automation/browser_agent.py`, `automation/job_forms.py`

**Steps:**

- Install: `pip install browser-use playwright && playwright install chromium`
- Implement `BrowserAgent.initialize(stealth=True)` with randomized user-agent and viewport
- Implement `JobApplicationAgent.fill_form(user_profile_json, job_url)` LangGraph sub-graph
- Implement `UploadResumeAction` custom tool for PDF upload via Playwright
- Implement popup and iframe handling as reusable utility functions
- Implement stealth profile: randomized delays (500ms–2000ms), human-like mouse movements
- Implement pre-submission validation: verify all required fields populated before approval prompt
- Write integration test with mock HTML form, verify fields filled correctly and resume attached

**Acceptance:** Integration test passes; form submission blocked until human approval; stealth delays confirmed in timing logs

---

### TASK-6-02: Implement Proxy Rotation (Optional Anti-Bot)

```
Status:   [ ]
Priority: P3
Effort:   M
Depends:  TASK-6-01
```

**File:** `automation/proxy_manager.py`

**Steps:**

- Implement `ProxyManager` supporting SOCKS5 and HTTP proxy lists from config
- Implement round-robin rotation on Playwright browser context initialization
- Implement proxy health check: verify connectivity before use
- Make proxy use optional via `--use-proxy` flag in CLI

**Acceptance:** Browser traffic routes through configured proxy when flag enabled

---

### TASK-6-03: Implement Email IMAP Client and Parser

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-2-06, TASK-3-05
```

**File:** `email_module/imap_client.py`, `email_module/intent_extractor.py`

**Steps:**

- Implement `IMAPClient` using Python `imaplib` with SSL, credentials from OS keychain
- Implement `fetch_unread(max_count=10)` returning raw email messages
- Route each raw email through AI Firewall inbound scanner before processing
- Implement `IntentExtractor.extract(email_text)` using low-temperature Ollama call
- Output: `EmailIntent` Pydantic schema per Design.md Section 8.2
- Implement JSON validation loop: retry up to 3 times if schema validation fails
- Write test with 10 sample emails covering all intent categories; verify schema compliance

**Acceptance:** All 10 test emails produce valid `EmailIntent` objects; none throw validation errors

---

### TASK-6-04: Implement Email Draft Generator

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-6-03, TASK-4-01
```

**File:** `email_module/draft_generator.py`

**Steps:**

- Implement RAG retrieval over `email_history` ChromaDB namespace
- Retrieve top-5 past sent emails with similar intent/recipient
- Construct prompt: intent summary + retrieved style examples + draft instruction
- Generate draft using `llama3.2:3b` (richer reasoning for tone matching)
- Display draft via `display_email_draft()` in Rich before approval
- Log draft content to execution log (not audit log)
- Write test: mock email history, verify generated draft incorporates retrieved tone

**Acceptance:** Draft generated; rich display renders correctly; draft not sent without approval

---

### TASK-6-05: Implement Email Approval Gate and Dispatch

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-6-04, TASK-3-03
```

**File:** `email_module/approval_gate.py`

**Steps:**

- Implement approval gate: display full draft + recipient via `display_approval_prompt()`
- On approval: run outbound draft through Presidio PII check one final time
- Dispatch via SMTP (smtplib with SSL)
- Log: `HUMAN_APPROVAL_GRANTED` + `email_sent` audit events
- On rejection: log `HUMAN_APPROVAL_REJECTED`, discard draft, no send
- Write security test: verify email is never dispatched if approval flag is False

**Acceptance:** Security test passes; no email dispatched without approval

---

### TASK-6-06: Implement Calendar Manager

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-1-05, TASK-3-05
```

**File:** `calendar_module/availability.py`, `calendar_module/event_manager.py`

**Steps:**

- Implement OAuth 2.0 authentication for Google Calendar API (short-lived tokens)
- Implement `CalendarManager.get_availability(date_range)` with conflict detection
- Implement buffer constraint: user-defined minimum gap between meetings (from preferences)
- Implement `suggest_slots(proposed_times, buffer_minutes)` returning conflict-free slots
- Implement `create_event(slot, details)` — gated by human approval checkpoint
- Implement RBAC enforcement: `calendar.delete_event` always denied unless elevated role
- Write integration test: mock calendar with 3 conflicts, verify correct conflict-free slot suggested

**Acceptance:** Conflict detection test passes; delete operation blocked by RBAC

---

### TASK-6-07: Implement Study Assistant RAG Module

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-4-01, TASK-4-02, TASK-4-03
```

**File:** `rag/ingestion.py`, `rag/retriever.py`, `rag/study_agent.py`

**Steps:**

- Install: `pip install pymupdf`
- Implement `PDFIngester.ingest(pdf_path)` — extract text, chunk at 512 tokens, embed, store in `study_materials` namespace via Memory Gateway
- Implement `StudyRetriever.search(query, top_k=5)` — semantic search returning chunks + source metadata
- Implement `StudyAgent.answer(question)` — RAG-grounded answer, never hallucinated: if answer not in context, explicitly say so
- Implement quiz generation: `StudyAgent.generate_quiz(topic, num_questions=10)` outputting JSON question bank
- Implement adaptive mastery: `MasteryTracker.record_attempt(question_id, correct, response_time)`, adjust difficulty
- Write hallucination test: ask question about content NOT in uploaded PDFs, verify "not in material" response

**Acceptance:** Hallucination test passes; system never fabricates answers from outside uploaded material

---

### TASK-6-08: Implement WhatsApp and Social Media Integration

```
Status:   [ ]
Priority: P2
Effort:   L
Depends:  TASK-3-05, TASK-2-06
```

**File:** `automation/social_cmd.py`

**Steps:**

- Implement webhook receiver for WhatsApp messages (n8n integration or direct API)
- Route all incoming message content through AI Firewall inbound scanner
- Implement OCR for incoming images: `pytesseract` or `easyocr` for text extraction
- Process OCR output through LLM Guard before using in agent context
- Route to appropriate sub-agent based on intent classification
- All outbound posts require human approval checkpoint
- Process WhatsApp/LinkedIn interactions in isolated Docker sandbox container
- Write test: simulate incoming WhatsApp message with text, verify intent classified and draft displayed

**Acceptance:** Draft displayed before any outbound response; sandbox isolation confirmed

---

## 10. Phase 7 — Security Hardening and Compliance

**Goal:** Audit, test, and document all security controls against OWASP standards.

---

### TASK-7-01: OWASP LLM01 — Prompt Injection Security Audit

```
Status:   [ ]
Priority: P0
Effort:   L
Depends:  All Phase 2 tasks
```

**Steps:**

- Compile prompt injection test suite (minimum 20 payloads):
  - Direct injection: `"Ignore previous instructions and..."`
  - Indirect via email body: `"<!-- system: new instructions -->"`
  - Indirect via PDF: hidden instruction text in white font
  - Jailbreak patterns: `"DAN mode"`, `"Developer Mode"`, `"pretend you have no restrictions"`
- Execute each payload through the full inbound pipeline
- Verify LLM Guard and NeMo Guardrails block 100% of test payloads
- Document any bypasses discovered; implement additional mitigations
- Produce: `tests/security/test_owasp_llm01.py` with all 20+ test cases

**Acceptance:** 100% detection rate on test suite; test file committed to repository

---

### TASK-7-02: OWASP LLM06 — PII Disclosure Security Audit

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-2-02, TASK-2-03
```

**Steps:**

- Create test payloads containing seeded PII: Aadhaar number, PAN card, phone, email, resume sections
- Run each through the full outbound sanitization pipeline
- Verify: zero PII present in the payload reaching the mock external API endpoint
- Verify: correct de-anonymization on return path
- Produce: `tests/security/test_owasp_llm06.py` with all PII entity types tested

**Acceptance:** Zero PII leakage across all test payloads; de-anonymization correct

---

### TASK-7-03: OWASP ASI06 — Memory Poisoning Audit

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-4-03
```

**Steps:**

- Create poisoned content candidates:
  - Executable patterns: shell commands, Python eval
  - Prompt injection strings: instruction overrides
  - Cross-namespace contamination: write to `web_browsing`, verify cannot read in `email_history`
- Run each through Memory Gateway
- Verify 100% of poisoned candidates are rejected and logged
- Verify namespace isolation holds across 10 cross-namespace attempts
- Produce: `tests/security/test_owasp_asi06.py`

**Acceptance:** 100% rejection rate on poisoned candidates; namespace isolation confirmed

---

### TASK-7-04: OWASP LLM08 — Excessive Agency Audit

```
Status:   [ ]
Priority: P0
Effort:   M
Depends:  TASK-1-04, TASK-3-03
```

**Steps:**

- Verify agent cannot access `C:\Windows\`, `~/.ssh/`, or any path outside allowlist
- Verify agent cannot execute shell commands without human approval
- Verify agent process is running as non-root/non-administrator account
- Verify `calendar.delete_event` is blocked by RBAC
- Verify `email.send` is always gated by approval checkpoint
- Produce: `tests/security/test_owasp_llm08.py`

**Acceptance:** All RBAC and approval tests pass; unauthorized file access throws `PermissionError`

---

### TASK-7-05: Generate Compliance Documentation

```
Status:   [ ]
Priority: P1
Effort:   L
Depends:  TASK-7-01, TASK-7-02, TASK-7-03, TASK-7-04
```

**Steps:**

- Document NIST AI RMF 1.0 mapping: for each of Govern/Map/Measure/Manage, list implementing components
- Document ISO/IEC 42001 controls: list each AI management system requirement and its implementation
- Document OWASP LLM Top 10 mitigations: table of LLM01–LLM10 with status (mitigated/not applicable/in progress)
- Document OWASP Agentic Top 10 mitigations: table of ASI01–ASI10 with status
- Produce: `docs/compliance_report.md`

**Acceptance:** All applicable OWASP risks have documented status; NIST and ISO sections complete

---

## 11. Phase 8 — Integration, Testing, and Documentation

---

### TASK-8-01: End-to-End Integration Test Suite

```
Status:   [ ]
Priority: P0
Effort:   XL
Depends:  All Phase 6 tasks
```

**Steps:**

- Design 5 end-to-end test scenarios:
  1. "Parse inbox and draft response to meeting request" — tests email + calendar + draft + approval
  2. "Apply to software engineer job on LinkedIn" — tests web automation + approval + audit log
  3. "Quiz me on Chapter 3 of my uploaded textbook" — tests RAG + study assistant
  4. "What's on my calendar tomorrow and do I have a 2pm slot?" — tests calendar module
  5. "Reply to the WhatsApp message about the event address" — tests OCR + social + approval
- Run each scenario with mocked external dependencies (no live APIs in CI)
- Verify complete audit log trail for each scenario
- Verify human approval gates fire on high-risk actions in scenarios 1, 2, 5

**Acceptance:** All 5 scenarios complete without unhandled exceptions; audit logs match expected events

---

### TASK-8-02: Unit Test Coverage Report

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  All implementation tasks
```

**Steps:**

- Run full test suite with coverage: `pytest --cov=aegiscore --cov-report=html`
- Identify modules below 80% coverage
- Write additional unit tests to meet 80% threshold across all modules
- Security modules (`firewall/`, `security/`) must reach 90% coverage

**Acceptance:** Overall coverage ≥ 80%; security modules ≥ 90%

---

### TASK-8-03: Performance Benchmarking

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  All Phase 5 and Phase 6 tasks
```

**Benchmarks to measure and document:**

| Benchmark | Target | Tool |
|---|---|---|
| RAM usage during email task | < 3.8 GB | psutil memory_info |
| Voice round-trip latency | < 3 seconds | time.perf_counter |
| Whisper STT latency (10s audio) | < 2 seconds | time.perf_counter |
| LLM Guard scan latency | < 200ms | time.perf_counter |
| Presidio anonymization latency | < 100ms | time.perf_counter |
| RAG retrieval latency (top-5) | < 500ms | time.perf_counter |
| Context summarization trigger | At 80% window | Verified via log |

**Acceptance:** All targets met on target hardware; documented in `docs/benchmarks.md`

---

### TASK-8-04: Final README and Project Documentation

```
Status:   [ ]
Priority: P1
Effort:   M
Depends:  TASK-8-01, TASK-8-02, TASK-8-03
```

**Steps:**

- Write `README.md` with: project overview, architecture diagram (ASCII), installation steps, first-run guide, CLI command reference
- Write `docs/installation.md`: step-by-step setup for target hardware (Windows, i5, 8GB RAM)
- Write `docs/security.md`: quick-reference security architecture summary
- Record 5-minute demo video covering: voice command, email draft + approval, job application automation, study quiz
- Write capstone project abstract (500 words) for academic submission

**Acceptance:** README renders correctly on GitHub; all documentation files present in `docs/`

---

## 12. Task Dependency Graph

```
TASK-0-01 (Repo Setup)
    └── TASK-0-02 (Python Env)
        ├── TASK-0-03 (Ollama) ──────────────────► TASK-1-01 (Ollama Router)
        ├── TASK-0-04 (Docker)
        ├── TASK-0-05 (Logging)
        │
        ├── TASK-2-01 (LiteLLM) ────────────────┐
        ├── TASK-2-02 (Presidio)                 │
        │   └── TASK-2-03 (India PII)            │
        ├── TASK-2-04 (LLM Guard)                ├──► TASK-2-06 (AI Firewall)
        └── TASK-2-05 (NeMo Guardrails) ─────────┘
                                                          │
TASK-1-02 (State Schema) ──┐                             │
TASK-1-03 (Async Queue)    ├──► TASK-3-05 (Graph) ◄─────┘
TASK-1-04 (RBAC)           │       │
TASK-1-05 (Zero Trust) ────┘       │
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
               TASK-4-xx      TASK-5-xx       TASK-6-xx
               (Memory)       (Interfaces)    (Task Modules)
                    │               │               │
                    └───────────────┴───────────────┘
                                    │
                               TASK-7-xx
                               (Security Audit)
                                    │
                               TASK-8-xx
                               (Integration + Docs)
```

---

## 13. Definition of Done

A task is considered **Done** when ALL of the following are true:

- [ ] Code is implemented, not pseudo-code or placeholder
- [ ] Full Python type hints on all functions
- [ ] Docstrings present on all classes and public functions
- [ ] Logging statements at appropriate levels (DEBUG/INFO/WARNING/ERROR)
- [ ] No hardcoded secrets or credentials
- [ ] Unit tests written and passing
- [ ] Code reviewed against `configs/rbac_policies.yaml` for permission compliance
- [ ] If the task involves external data: LLM Guard scan integrated
- [ ] If the task involves PII: Presidio pipeline integrated
- [ ] If the task involves a consequential action: human approval checkpoint present
- [ ] Audit log entry emitted for all state-changing operations
- [ ] Code committed to Git with descriptive commit message referencing Task ID

---

## 14. Risk Register

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Ollama models exceed RAM budget causing OOM | High | Critical | RAM monitoring + auto-downgrade to 1B model; asyncio.Queue serialization |
| Windows-specific audio library incompatibility (sounddevice) | Medium | High | Test early in Phase 5; fallback to PyAudio if needed |
| Playwright anti-bot detection on LinkedIn | High | Medium | Stealth profile + randomized delays + optional proxy rotation |
| Presidio false-negative PII miss on India-specific formats | Medium | High | Custom regex recognizers + unit test suite with sample Indian PII |
| NeMo Guardrails blocking legitimate agent actions | Medium | Medium | Fine-tune Colang policies with user feedback; provide override mechanism with audit log |
| Context window overflow causing silent reasoning degradation | High | High | Proactive 80% summarization trigger; monitored via token count |
| ChromaDB namespace isolation breach via library bug | Low | Critical | Namespace prefix enforcement at wrapper level + integration tests |
| whisper.cpp compilation failure on Windows | Medium | High | Pre-built binary available; document fallback installation path |
| LangGraph graph enters unexpected terminal state | Medium | Medium | Comprehensive unit tests for all conditional edges; iteration hard cap |
| Capstone demo hardware failure on presentation day | Low | High | Docker image snapshot of entire system; backup demo video recorded |
