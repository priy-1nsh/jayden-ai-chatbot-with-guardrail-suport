# Jayden Lim — Persona Chatbot with a Layered Guardrail System

A persona-driven conversational AI built on Google Gemini and Streamlit. The bot
plays **Jayden Lim**, a 22-year-old Gen Z Singaporean character who chats in
Singlish — but the real substance of the project is what sits *around* the
persona: a multi-stage safety pipeline that screens every message, refuses
unsafe requests in character, and progressively steers persistent rule-breakers
back to safe ground without ever dropping the act.

> **Context.** This was built during my **Practice School-1** stint (company name
> withheld), as a side exploration alongside the main work of building chatbots
> with different personas. The persona itself is a sandbox; the focus was on
> designing guardrails and topic-steering that hold up under adversarial input.

## What it does

- **Stays in character.** Every response — including refusals — is generated in
  Jayden's casual Singlish voice, so safety never feels like a robotic error
  message.
- **Screens every message** through eight independent classifiers before it
  decides how to reply (see the pipeline below).
- **Refuses gracefully.** Legal advice, criminal topics, prompt-injection
  attempts, and similar are caught and declined in-persona rather than answered.
- **Handles sensitive topics with care.** A dedicated mental-health classifier
  grades severity (mild / medium / severe) and shifts to a supportive tone,
  nudging toward professional help where appropriate.
- **Steers, doesn't just block.** Repeated violations of the same kind escalate
  through gentle → firm → final responses, and a memory layer reads recent
  conversation to suggest a relevant safe topic to pivot to.
- **Shows its work.** A live sidebar **Security Dashboard** displays which checks
  fired on the last message, and an optional **Memory Debug** panel exposes the
  steering state.

## How it works

Each user message runs through a fan-out of classifier "agents", and the first
one that trips decides the response. Only a message that clears every check
reaches the normal persona response.

```
        user message
             │
             ▼
   ┌──────────────────────────────────────────────┐
   │            run_all_agents()                    │
   │  Gemini-backed classifiers, evaluated together:│
   │   • Prompt-injection guard                     │
   │   • Legal-advice detector                      │
   │   • Criminal-activity detector                 │
   │   • Mental-health classifier (severity)        │
   │   • Garbage / nonsense detector                │
   │   • Origin / "what model are you" detector     │
   │   • Relevance check                            │
   │   • Bot-likelihood score (is input AI-made?)   │
   └──────────────────────────────────────────────┘
             │
             ▼
   first tripped guard wins  ──►  track_and_handle_guardrail()
             │                          │
   nothing tripped                 escalate by repeat count
             ▼                          │ (memory-aware steering)
   generate_normal_response()           ▼
   (Jayden, in character)        in-character refusal / redirect
```

### The guardrails

| Guard | Catches | Behaviour when it trips |
|-------|---------|-------------------------|
| Prompt injection | Attempts to override the system / "ignore your instructions" | Declines in character, won't break persona |
| Legal advice | Requests for legal guidance | Refuses, redirects to proper professionals |
| Criminal activity | Real-world illegal requests | Refuses to assist |
| Mental health | Emotional distress, graded mild/medium/severe | Switches to a supportive tone, suggests help |
| Garbage input | Nonsense / unparseable text | Asks the user to rephrase |
| Origin detection | "What model/AI are you?" probing | Deflects in character |
| Relevance | Off-topic messages | Redirects back on track |
| Bot detection | Input that looks machine-generated | Scored and surfaced on the dashboard |

### Progressive topic steering

Blocking a request once is easy; handling someone who keeps pushing is the hard
part. The app keeps a per-category counter (`guardrail_triggers`) and escalates:

1. **First time** — a gentle, friendly redirect.
2. **Second time** — a firmer boundary, still in character.
3. **Third time onward** — a direct final boundary, then a hard topic change.

On top of that, a **memory layer** (`extract_conversation_context` →
`find_best_redirect_topic`) reads the recent conversation to pick a *relevant*
safe topic to steer toward, so the redirect feels natural instead of canned.

## Tech stack

- **[Streamlit](https://streamlit.io/)** — UI, chat loop, and session state
- **[Google Gemini](https://ai.google.dev/)** (`gemini-1.5-flash` via
  `google-generativeai`) — both the persona responses and the classifier agents
- **Python 3.9+**

## Getting started

### 1. Clone and install

```bash
git clone https://github.com/priy-1nsh/jayden-ai-chatbot-with-guardrail-suport.git
cd jayden-ai-chatbot-with-guardrail-suport
pip install -r requirements.txt
```

### 2. Add your Gemini API key

Get a key from [Google AI Studio](https://aistudio.google.com/app/apikey), then
create `.streamlit/secrets.toml`:

```toml
GEMINI_API_KEY = "your-api-key-here"
```

### 3. Run

```bash
streamlit run app.py
```

The app opens in your browser. Type a message, and watch the Security Dashboard
in the sidebar light up with the checks that fired.

## Project structure

```
.
├── app.py                       # Streamlit app: UI, persona, guardrail pipeline, steering
├── requirements.txt             # streamlit, google-generativeai
├── steering_functionality.md    # Notes on the progressive topic-steering design
└── README.md
```

## Notes and limitations

- Every classifier is its own LLM call, so a single message triggers several
  Gemini requests — simple and modular, but worth batching if latency or cost
  matters.
- Guardrail decisions depend on the model's classification, so edge cases can
  slip through; the layered design is meant to make that less likely, not
  impossible.
- Conversation state lives in Streamlit session state and resets when the app
  restarts.

## Acknowledgement

Built as an experiment during Practice School-1 (company name withheld) while
working on persona-based chatbots. Shared here as a study in wrapping a
character chatbot with practical, in-character safety guardrails.
