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

## Architecture

React/Vite -> FastAPI -> hard compatibility checks -> structured scoring -> two Groq-powered profile clones -> persisted local JSON.

Data files: users.json, questionnaires.json, matches.json, negotiation_logs.json under data/.

There is no external database. JSON writes use atomic replacement and file locking. Entities use UUIDs and explicit ID references.

##For the jury

SupplyzPro Award
SupplyzPro → why the project fits → concrete evidence → where to see it → country/eligibility fit
SupplyzPro → The project does not currently demonstrate the “Find the Hidden Failures” use case required by SupplyzPro. Its AI agents are designed for roommate compatibility: they surface deal-breakers, negotiate preferences, and produce a five-dimension compatibility verdict, rather than detecting, grouping, and prioritizing operational failures.
Concrete evidence → The project documentation shows early deal-breaker detection within the first three turns and structured AI negotiation, but it does not provide evidence of failure detection, failure grouping, or evidence-based prioritization. Where to see it: project PDF, pages 3–4, sections “User value” and “Differentiation”; no repository or video evidence was provided in the submitted material. Country/eligibility fit: Not confirmed from the provided materials — do not claim eligibility.
Important: I would not submit a stronger SupplyzPro claim unless you actually have a repository/video showing those three capabilities. The PDF supports the roommate-matching claims, but not the SupplyzPro “hidden failures” criteria. aa hotou fel readme


## Run locally

Backend:
- Create a Python virtual environment.
- Install requirements with: pip install -r requirements.txt
- Copy .env.example to .env and set GROQ_API_KEY for live AI.
- Start with: python -m backend.server

Frontend:
- cd frontend
- npm install
- npm run dev

The frontend defaults to http://localhost:8000 for the API. Set VITE_API_URL if needed.

Without a Groq key, the prototype uses deterministic fallback clone responses and scoring so the full flow remains demonstrable.

## AI contribution

AI represents each participant through a profile-specific clone, conducts the multi-turn compatibility conversation, and summarizes shared ground, friction points, and compromises.

Deterministic code remains responsible for hard deal-breakers and the final compatibility guard. The model cannot override an explicit hard conflict.

English, French, and Arabic are supported, with Tunisian locations and TND budgets. 



