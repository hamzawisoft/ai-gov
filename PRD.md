# PRODUCT REQUIREMENT DOCUMENT (PRD)

## Project Title: Sovereign Multi-Agent System (MAS) Framework for Frappe & ERPNext v16
**Document Version:** 1.1.0
**Status:** Approved for Implementation
**Target Platform:** Frappe Framework v16 & ERPNext v16 Ecosystem
**Primary Language of Document:** English

---

## 1. Executive Summary & Vision

### 1.1 Executive Overview
This Product Requirement Document (PRD) defines the architectural, functional, and technical specifications for a sovereign **Multi-Agent System (MAS)** built natively on top of the **Frappe v16 / ERPNext v16** platform.

The system aims to interconnect state departments, public institutions, and government ministries into a unified, intelligent digital ecosystem. The core strategy relies on specialized, autonomous **AI Agents** powered by advanced LLM orchestration frameworks (**LangGraph**, **LangChain**, **Model Context Protocol (MCP)**, and **Hybrid Vector Stores** like ChromaDB/Weaviate).

### 1.2 Core Philosophy
1. **Autonomous Execution with Human-in-the-Loop (HITL):** High-risk actions (code deployment, database migrations, public announcements) are staged and held at interactive approval gates for human review before production release.
2. **Infinite Goal-Driven Persistence:** Agents (especially the Developer Agent) operate with unbounded runtime goals, persisting state across days or weeks via deterministic LangGraph state checkpointing.
3. **Hyper-Explicit Specifications for LLM Coders:** Every module, DocType, API contract, and graph edge is explicitly mapped out so that even lower-cost or parameter-constrained LLM coders can execute the implementation without ambiguity.
4. **Sovereign Security & Data Isolation:** Each ministry or department maintains dedicated, isolated database schemas while allowing federated, permissioned access to national decision support agents.

---

## 2. High-Level System Architecture

```mermaid
graph TD
    subgraph Client & Interface Layer
        PublicUser[Public Citizens / Telegram Bot]
        ExpertUser[Ministry Experts / Telegram Bot]
        AdminUser[Technical Admin / Frappe Custom Dashboard]
        ExecutiveUser[Ministers & Leadership / Strategic BI Console]
    end

    subgraph Security & Guardrail Gateway
        Guardrail[NeMo Guardrails / PII Masking / Prompt Defense]
        ComplianceAgent[5. Security & Compliance Agent]
    end

    subgraph Frappe v16 Core Ecosystem (Site Benches)
        SiteA[Ministry Site A - Frappe v16]
        SiteB[Ministry Site B - Frappe v16]
        StagingBench[Isolated Staging Bench / Site]
        ProdBench[Production Bench / Site]
    end

    subgraph Multi-Agent System (MAS) Layer - LangGraph Engine
        DeveloperAgent[1. Developer Agent]
        ExpertAgent[2. Expert Advisory Agent]
        PublicAgent[3. Public Service Agent]
        DecisionAgent[4. Decision-Maker Orchestrator Agent]
        BIAgent[6. Strategic Data Analyst / BI Agent]
    end

    subgraph Protocol & Tooling Layer (MCP)
        MCPRegistry[MCP Dynamic Tool & Skill Registry]
        FrappeAPIs[Frappe REST / RPC / DocType Tools]
        GitTools[Git / Bench / Terminal CLI Tools]
    end

    subgraph Memory & Knowledge Storage
        RedisMemory[Redis - Short-Term Thread State & Checkpoints]
        ChromaWeaviate[ChromaDB / Weaviate - Hybrid RAG & Long-Term Memory]
    end

    PublicUser --> Guardrail --> PublicAgent
    ExpertUser --> Guardrail --> ExpertAgent
    AdminUser --> FrappeAPIs --> DeveloperAgent
    ExecutiveUser --> BIAgent

    DeveloperAgent --> GitTools
    DeveloperAgent --> StagingBench
    DeveloperAgent -- HITL & Compliance Gate --> ComplianceAgent --> ProdBench

    DecisionAgent --> BIAgent
    DecisionAgent --> ChromaWeaviate
    DecisionAgent --> SiteA
    DecisionAgent --> SiteB

    MAS Layer <--> MCPRegistry
    MAS Layer <--> RedisMemory
    MAS Layer <--> ChromaWeaviate
```

---

## 3. Specialized AI Agent Specifications

### 3.1 Agent 1: The Developer Agent (Autonomous Systems Engineer)

#### 3.1.1 Purpose & Role
The Developer Agent is the technical backbone of the system. It is tasked with setting up, customizing, extending, and maintaining Frappe custom apps, DocTypes, Server Scripts, Client Scripts, and workflows based on natural language requirements from administrators.

#### 3.1.2 Key Behavioral Requirements & Goal Persistence
- **Unbounded Task Execution Loop:** Must support multi-day execution tasks using LangGraph persistent checkpointers backed by Redis/PostgreSQL.
- **Intelligent Requirements Elicitation:** Asks clarifying questions if user input is ambiguous before executing modifications.
- **Staging-First Mandate:**
  1. Targets a Staging Frappe Site/Bench.
  2. Applies code modifications, custom DocTypes, or migrations.
  3. Executes unit tests and linter verifications.
  4. Generates an interactive **Staging Preview Link** and change report.
  5. Triggers automated validation with the **Security & Compliance Agent**.
  6. Pauses graph execution at a `HITL_APPROVAL_NODE` waiting for technical admin sign-off.
  7. Upon human approval, automatically triggers production deployment.

#### 3.1.3 Developer Agent Tools & MCP Capabilities
- `frappe_create_doctype`: Schema generator for Frappe v16 DocTypes.
- `frappe_execute_bench_cmd`: Sandboxed CLI tool for running `bench migrate`, `bench new-site`, `bench install-app`.
- `git_branch_commit_pr`: Version control manager for creating git commits and branches.
- `run_frappe_tests`: Executes `bench run-tests --app <custom_app>`.
- `staging_deploy_verifier`: Generates HTTP preview endpoints and validates response status.

---

### 3.2 Agent 2: The Decision-Maker Agent (National Orchestrator)

#### 3.2.1 Purpose & Role
An executive-level agent designed for state leadership, ministers, and top-tier decision-makers. It aggregates data across heterogeneous ministries without violating local tenant data security.

#### 3.2.2 Key Features
- **Cross-Agency Federation:** Queries multiple isolated vector stores (ChromaDB / Weaviate instances) and SQL databases across ministries via secure RPC calls.
- **Real-Time Strategic Analytics:** Synthesizes unstructured reports, financial metrics, HR headcount, and operational bottlenecks into unified executive summaries.
- **Sub-Agent Delegation:** Decomposes complex national queries into sub-tasks delegated to domain-specific Expert Agents or the Strategic Data Analyst / BI Agent.

---

### 3.3 Agent 3: The Expert Agent (Advisory & Consulting Specialist)

#### 3.3.1 Purpose & Role
Serves as an embedded domain consultant for specific ministry departments (e.g., Finance, Human Resources, Infrastructure, Procurement).

#### 3.3.2 Interfaces & Capabilities
- **Dual Interface Access:** Accessible via Frappe Custom Workspace UI and a dedicated secure Telegram Bot.
- **Full ERPNext Integration:** Holds authorized read/write tool bindings to Frappe DocTypes (`Sales Invoice`, `Purchase Order`, `Employee`, `Project`, etc.).
- **Long-Term Memory:** Implements Hermes-inspired episodic and declarative memory to recall past policy decisions and ministerial preferences.

---

### 3.4 Agent 4: The Public Agent (Citizen Services Specialist)

#### 3.4.1 Purpose & Role
Handles public inquiries, citizen complaints, application status tracking, and general public guidance.

#### 3.4.2 Safety & Guardrails
- **Public Telegram Bot Interface:** Multi-lingual (Arabic & English primary).
- **Zero Raw Database Access:** Operates strictly through sanitized public API endpoints with input/output PII masking.
- **Automated Ticket Creation:** Translates public complaints into Frappe `Issue` or `HD Ticket` DocTypes in the relevant ministry site.

---

### 3.5 Agent 5: The Security & Compliance Agent (Sovereignty & Audit Safeguard)

#### 3.5.1 Purpose & Role
Acts as an autonomous compliance auditor and continuous monitoring agent. It inspects all generated code, schema migrations, API outputs, and agent actions to ensure strict compliance with national cybersecurity standards, legal frameworks, and sovereign data laws.

#### 3.5.2 Key Capabilities
- **AST & Code Vulnerability Inspection:** Scans all Python and JavaScript code generated by the Developer Agent for SQL injection, command injection, insecure imports, and unauthorized file system access before staging/production approval.
- **Policy Violation Blocking:** Intercepts agent tool calls that violate data classification or cross-ministry access policies.
- **Automated Compliance Logging:** Generates audit certificates attached to every Staging Release PR.

---

### 3.6 Agent 6: The Strategic Data Analyst / BI Agent (National Intelligence Engine)

#### 3.6.1 Purpose & Role
A specialized data intelligence agent dedicated to decision-makers, ministers, and cabinet members. It reads across the unified knowledge base and federated database connections to generate real-time Business Intelligence (BI) dashboards, trend analysis, and predictive models without requiring manual navigation across individual applications.

#### 3.6.2 Key Capabilities
- **NL-to-SQL & Multi-Site Aggregation:** Converts natural language queries (e.g., "Compare quarterly expenditure versus project completion rates across Ministry of Health and Ministry of Education") into optimized, read-only SQL queries executed safely across federated instances.
- **Automated Visual Chart Generation:** Generates dynamic charting configurations (JSON chart specs rendered directly in Frappe custom UI dashboards).
- **Proactive Threat & Opportunity Alerting:** Identifies operational anomalies (e.g., supply chain bottlenecks, budget overruns) and pushes real-time alert digests to leadership.

---

## 4. Memory Architecture & Knowledge Base Setup

### 4.1 Hermes Agent Memory Model Adaptation
To achieve human-like long-term recall and contextual consistency, all agents implement a 3-tier memory model:

```
+-----------------------------------------------------------------------+
|                         AGENT MEMORY SYSTEM                           |
+-----------------------------------------------------------------------+
| 1. Short-Term Context (LangGraph Thread Checkpoints / Redis)           |
|    - Active conversation window, prompt stack, step-by-step reasoning |
+-----------------------------------------------------------------------+
| 2. Long-Term Episodic Memory (ChromaDB / Weaviate Vector DB)          |
|    - Past interaction logs, historic problem solutions, past feedback |
+-----------------------------------------------------------------------+
| 3. Long-Term Declarative Memory (Frappe Custom DocType: Agent Skill)   |
|    - Curated knowledge, system rules, procedural guidelines, MCPs    |
+-----------------------------------------------------------------------+
```

### 4.2 Hybrid RAG Engine
- **Vector Database:** ChromaDB (for single-node light deployments) or Weaviate (for multi-tenant cluster scale).
- **Dense Embeddings:** `text-embedding-3-large` or open-weight `bge-m3` for Arabic/English multi-lingual support.
- **Sparse Search:** BM25 lexical keyword search combined via Reciprocal Rank Fusion (RRF).
- **Re-ranking Stage:** Cohere / FlashRank re-ranker to ensure 99.9% precision for legal/governmental regulatory documents.

---

## 5. Model Context Protocol (MCP) & Dynamic Skill Registry

### 5.1 Architecture
The system utilizes Anthropic's **Model Context Protocol (MCP)** to decouple tools from agents:
- Each Frappe App exports an **MCP Server API** interface.
- Agents act as **MCP Clients**, dynamically discovering available tools based on the user's role and tenant site.

### 5.2 Dynamic Skill DocType Schema (`Agent Skill`)
```json
{
  "doctype": "Agent Skill",
  "module": "Frappe MAS Engine",
  "fields": [
    {"fieldname": "skill_name", "fieldtype": "Data", "reqd": 1, "unique": 1},
    {"fieldname": "target_agent", "fieldtype": "Select", "options": "Developer\nExpert\nPublic\nDecision Maker\nSecurity Compliance\nData Analyst BI"},
    {"fieldname": "mcp_server_url", "fieldtype": "Data"},
    {"fieldname": "execution_script", "fieldtype": "Code", "options": "Python"},
    {"fieldname": "required_permission", "fieldtype": "Link", "options": "Role"}
  ]
}
```

---

## 6. Frappe Custom UI & Visual Reasoning Dashboard

Based on the reference UI requirements (ERPNext Integration with AI Assistants), a dedicated Frappe Custom Workspace module named **"MAS Control Tower"** shall be provided.

### 6.1 Dashboard Components
1. **Agent Execution Graph View:** Real-time visual representation of LangGraph nodes, active states, and sub-agent delegates.
2. **Reasoning Trace Console:** Displays LLM thought logs, tool execution outputs, and token metrics.
3. **Staging Review & Approval Portal:**
   - Displays pending deployments generated by the Developer Agent.
   - Shows Security & Compliance audit scores, git diffs, modified DocType structures, and unit test results.
   - Action buttons: `[ Approve & Deploy to Production ]` | `[ Request Revision ]` | `[ Reject ]`.
4. **Strategic BI & Executive Intelligence Panel:** Displays auto-generated charts, cross-ministry KPIs, and proactive anomaly alerts generated by the Strategic Data Analyst / BI Agent.
5. **Telegram Bot Conversation Inspector:** Allows authorized admins to review citizen and expert interactions for quality control.

---

## 7. Security, Guardrails & Compliance Framework

1. **Prompt Injection Defense:** NeMo Guardrails layer filtering input prompts for injection, jailbreaks, or exfiltration attempts.
2. **Automated Compliance Gate:** Managed by the **Security & Compliance Agent** to enforce code security standards before human sign-off.
3. **PII Masking Filter:** Automated scrubbing of national identification numbers, phone numbers, and private addresses prior to passing contexts to external LLM providers.
4. **Multi-Tenant Data Isolation:** Direct database connections between ministry sites are blocked. Communication occurs exclusively through authenticated secure REST API endpoints with JWT payload signing.
5. **Immutable Audit Logging:** Every LLM invocation, tool call, and human approval action is recorded in a write-once Frappe DocType (`MAS Audit Log`).

---

## 8. Implementation Roadmap & Milestones

| Phase | Description | Deliverables |
| :--- | :--- | :--- |
| **Phase 1** | Core MAS Engine & Frappe v16 Module | `frappe_mas` custom app, LangGraph engine bindings, Redis checkpointer |
| **Phase 2** | Developer Agent, Compliance Agent & Staging Pipeline | Developer Agent graph, Security & Compliance Agent, Git/Bench tools, HITL Approval Portal |
| **Phase 3** | Memory System & Hybrid RAG | ChromaDB/Weaviate integration, Hermes episodic/declarative memory model |
| **Phase 4** | Expert & Public Telegram Agents | Telegram Bot handlers, PII masking guardrails, Public Service agent |
| **Phase 5** | Strategic BI Agent, Decision-Maker Federation & Control Tower | Cross-site query orchestrator, Strategic BI Agent, MAS Control Tower visual UI |

---
*End of Product Requirement Document (PRD)*
