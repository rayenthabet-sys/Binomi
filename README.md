# Binomi

AI-powered roommate compatibility matching for Tunisia.

## What Binomi does

1. Enter a name — no account, password, OAuth, or login.
2. Complete a profile covering location, TND budget, routines, lifestyle, hobbies, priorities, and deal-breakers.
3. Browse other completed profiles.
4. Run an AI match: two profile-specific clones hold an ACP/1.0-style conversation.
5. Receive Strong, Conditional, or Incompatible results with dimension scores and explanations.
6. Open the complete saved clone negotiation later.
7. Chat with your own clone to verify how your profile is represented.

---

## System Architecture & Jury Call Flow

### 1. High-Level Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer (Frontend)"]
        UI["React 18 + Vite SPA"]
        I18N["Trilingual Engine (EN / FR / AR)"]
        Landing["Landing & Session Auth"]
        QuestUI["Questionnaire Form"]
        MatchUI["Candidate Explorer & Match Viewer"]
        PlayUI["Clone Chat Playground"]
    end

    subgraph Server["Application Layer (FastAPI Backend)"]
        Router["FastAPI REST Router"]
        AuthHandler["Session & Profile Manager"]
        MatchEngine["Negotiation Orchestrator"]
        GuardEngine["Deterministic Conflict Guard"]
        ScoreEngine["5-Dimension Scoring Engine"]
    end

    subgraph AI["AI Inference Layer (Groq Cloud)"]
        CloneA["Persona Clone A (Agent)"]
        CloneB["Persona Clone B (Agent)"]
        Evaluator["Compatibility Synthesizer (LLM)"]
    end

    subgraph Data["Persistence Layer (Atomic JSON)"]
        UJSON[("users.json")]
        QJSON[("questionnaires.json")]
        MJSON[("matches.json")]
        NJSON[("negotiation_logs.json")]
    end

    UI --> Router
    Router --> AuthHandler
    Router --> MatchEngine

    AuthHandler --> UJSON
    AuthHandler --> QJSON

    MatchEngine --> GuardEngine
    MatchEngine --> ScoreEngine
    MatchEngine --> CloneA
    MatchEngine --> CloneB
    MatchEngine --> Evaluator

    GuardEngine -.->|"Hard deal-breakers enforce Incompatible"| MatchEngine
    CloneA <-->|"ACP/1.0 Multi-Turn Dialogue"| CloneB
    Evaluator -->|"Structured Verdict"| MatchEngine

    MatchEngine --> MJSON
    MatchEngine --> NJSON
```

---

### 2. End-to-End Call Flow (Jury Sequence)

The diagram below maps every call from user onboarding, autonomous 8-turn agent negotiation, and evaluation synthesis, to self-verification:

```mermaid
sequenceDiagram
    autonumber
    actor User as User / Jury
    participant FE as Frontend (React + Vite)
    participant API as FastAPI Backend
    participant Rules as Deterministic Engine
    participant Groq as Groq AI Cloud
    participant Storage as Local JSON Storage

    Note over User,Storage: Phase 1: Onboarding & Profile Setup
    User->>FE: Enter name or restore Session ID
    FE->>API: POST /api/users/enter {name, language}
    API->>Storage: Create or update user in users.json
    API-->>FE: Return User Object (UUID)
    User->>FE: Fill questionnaire (budget, lifestyle, deal-breakers)
    FE->>API: PUT /api/users/{id}/questionnaire {questionnaire}
    API->>Storage: Upsert into questionnaires.json
    API-->>FE: Status 200 OK

    Note over User,Storage: Phase 2: Autonomous Multi-Agent Matching
    User->>FE: Select Candidate & Click "Run Match"
    FE->>API: POST /api/matches/run {user_id, candidate_id}
    API->>Storage: Fetch Profile A & Profile B
    API->>Rules: hard_conflicts(a, b) [City mismatch, budget non-overlap, smoking, pets]
    API->>Rules: scores(a, b) [Deterministic baseline across 5 dimensions]

    loop 8 Autonomous Negotiation Turns (4 Phases)
        Note over API,Groq: Turns 1-2: Intro | Turns 3-4: Lifestyle | Turns 5-6: Negotiation | Turns 7-8: Wrap-up
        API->>Groq: clone_turn() with system_prompt(persona) + sliding history
        Groq-->>API: Natural dialogue turn + clarifying question
    end

    API->>Groq: analyze() [Synthesize positives, frictions, compromises]
    Groq-->>API: JSON evaluation verdict
    API->>Rules: Safety guardrail (Hard conflicts strictly override AI optimism)
    API->>Storage: Atomic append to negotiation_logs.json
    API->>Storage: Atomic append to matches.json
    API-->>FE: Return match verdict, 5D scores, and full dialogue
    FE-->>User: Display Compatibility Card & Turn-by-Turn Replay

    Note over User,Storage: Phase 3: Persona Clone Sandbox (Self-Inspection)
    User->>FE: Open "Chat with your clone" & send query
    FE->>API: POST /api/clone/chat {user_id, message, history}
    API->>Groq: clone_turn() using user's questionnaire persona
    Groq-->>API: In-character response
    API->>Storage: Append turn to negotiation_logs.json
    API-->>FE: Return clone reply
    FE-->>User: Display clone response
```

---

### 3. API Call Reference for the Jury

| Endpoint | Method | Trigger / Purpose | Processing Logic | Deterministic vs AI |
| :--- | :--- | :--- | :--- | :--- |
| `/api/users/enter` | `POST` | Onboarding / Login | Creates new UUID or resumes existing session ID. | **Deterministic** |
| `/api/questionnaire/schema` | `GET` | Form Builder | Delivers localized form schema (Tunisian cities, TND brackets). | **Deterministic** |
| `/api/users/{id}/questionnaire` | `PUT` | Save Profile | Validates & persists housing/lifestyle preferences. | **Deterministic** |
| `/api/candidates/{id}` | `GET` | Discovery Screen | Filters candidates who have completed questionnaires. | **Deterministic** |
| `/api/matches/run` | `POST` | **Core Matching Pipeline** | 1. Checks hard deal-breakers.<br>2. Computes baseline scores.<br>3. Runs 8-turn ACP/1.0 clone dialogue.<br>4. Generates synthesized verdict. | **Hybrid**: Deterministic guardrails enforce non-negotiables; Groq LLM powers dialogues & synthesis. |
| `/api/matches/{id}` | `GET` | Match History | Retrieves historical verdicts, dimension scores, and partner profiles. | **Deterministic** |
| `/api/negotiations/{log_id}` | `GET` | Audit / Replay | Delivers the complete turn-by-turn negotiation transcript. | **Deterministic** |
| `/api/clone/chat` | `POST` | Persona Sandbox | Allows the user to interrogate their own clone to verify how they are represented. | **AI**: Single-persona Groq prompt with conversation memory. |

---

## Run locally

### Backend:
- Create a Python virtual environment.
- Install requirements with: `pip install -r requirements.txt`
- Copy `.env.example` to `.env` and set `GROQ_API_KEY` for live AI.
- Start with: `python -m backend.server`

### Frontend:
- `cd frontend`
- `npm install`
- `npm run dev`

The frontend defaults to `http://localhost:8000` for the API. Set `VITE_API_URL` if needed.

Without a Groq key, the prototype uses deterministic fallback clone responses and scoring so the full flow remains demonstrable.

---

## AI Contribution & Compatibility Intelligence

AI represents each participant through a profile-specific clone, conducts multi-turn compatibility conversations, and summarizes shared ground, friction points, and compromises.

- **Early Deal-Breaker Detection**: Evaluates critical deal-breakers within the first three turns of negotiation.
- **Five-Dimension Verdict**: Analyzes compatibility across living habits, rhythm, financial expectations, social boundaries, and communication style.
- **Deterministic Guardrails**: Hard deal-breakers (budget ceilings, non-negotiable locations, smoking/pet constraints) are guarded deterministically — the AI cannot override an explicit hard conflict.
- **Trilingual Support**: English, French, and Arabic are supported, with Tunisian locations and TND budgets.

---

## Hackathon & Sponsor Award Alignment

### SupplyzPro Award Assessment

| Criteria | Analysis & Evidence |
| :--- | :--- |
| **Track & Focus** | **SupplyzPro Award** (*Find the Hidden Failures*) |
| **Why the project fits** | The project does not currently demonstrate the "Find the Hidden Failures" use case required by SupplyzPro. Its AI agents are designed for roommate compatibility: they surface deal-breakers, negotiate preferences, and produce a five-dimension compatibility verdict, rather than detecting, grouping, and prioritizing operational failures. |
| **Concrete evidence** | The project documentation shows early deal-breaker detection within the first three turns and structured AI negotiation, but it does not provide evidence of failure detection, failure grouping, or evidence-based prioritization. |
| **Where to see it** | Project PDF, pages 3–4, sections "User value" and "Differentiation"; backend negotiation logic (`backend/server.py`) and logs (`data/negotiation_logs.json`). |
| **Country / Eligibility fit** | Not confirmed from the provided materials — do not claim eligibility. |

> **Important Assessment Note:** A stronger SupplyzPro claim should not be submitted unless a repository or video demonstration explicitly showcases failure detection, grouping, and operational prioritization. The materials reliably validate the roommate-matching and deal-breaker negotiation claims.
