# 🎯 OpenCoach

> AI-augmented coaching platform that learns from every human intervention

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Concept](https://img.shields.io/badge/Status-Concept-blue.svg)]()

---

## The Problem

Traditional coaches face a **scaling ceiling**:
- Limited to ~11 billable hours per week
- Can't serve clients 24/7
- Repetitive conversations drain energy
- Context gets lost between sessions

Meanwhile, pure AI coaching lacks the **human judgment** that makes coaching effective.

---

## The Solution

**OpenCoach** creates a hybrid AI + human coaching model:

```
AI handles 80% → Routine conversations, 24/7 availability
Human handles 20% → Complex moments, breakthroughs, judgment
```

### Key Innovation: Learning System

Unlike other AI coaching tools, **OpenCoach learns from every human intervention**:

1. **Client-specific** — "What works for THIS client"
2. **Coach-specific** — "THIS coach's unique style"  
3. **Universal** — "What works across all coaches"

The result: AI that coaches more like you over time.

---

## Core Features

### 🤖 Hybrid AI + Human Model
- AI responds by default
- Human takes over when needed (crisis, low confidence, high stakes)
- Seamless handoff with full context

### 🧠 4-Layer Memory System
- **Short-term:** Recent conversation
- **Session summaries:** AI-generated session overviews
- **Key insights:** Goals, blockers, breakthroughs (structured)
- **Relationship context:** Coaching dynamic, preferences

### 📚 RAG-Powered Knowledge Base
- Upload PDFs, videos, course content
- Vector search for semantic matching
- AI references coach's frameworks automatically

### 🔑 BYOK (Bring Your Own Key)
- Coaches use their own OpenAI/Anthropic/Gemini API keys
- Transparent AI costs
- Platform fee only ($49-149/mo)

### 💬 Multi-Channel (Future)
- Web (Phase 1)
- Telegram (Phase 2)
- WhatsApp Business API (Phase 3)

---

## Target Market

| Segment | Description |
|---------|-------------|
| **Primary** | Business coaches, executive coaches, life coaches |
| **Secondary** | Fitness coaches, career coaches, relationship coaches |
| **Tertiary** | Corporate L&D coaches |

**Market Size:**
- Global coaching industry: **$5.34B**
- Executive coaching: **$103.6B → $161.1B by 2030**
- Active coaches: **122,974+ globally**

---

## Business Model

| Tier | Price | Features |
|------|-------|----------|
| **Free** | $0 | 1 client, basic AI |
| **Pro** | $49/mo | 10 clients, full features, Telegram |
| **Team** | $149/mo | Unlimited clients, multi-coach, WhatsApp |
| **Enterprise** | Custom | SSO, white-label, API access |

**BYOK Model:** Coaches pay for their own AI usage (transparent, no markup)

---

## Technical Stack

- **Frontend:** Next.js 14 + Tailwind CSS
- **Backend:** Next.js API Routes + Supabase
- **Database:** PostgreSQL (Supabase)
- **Vector DB:** pgvector (RAG)
- **AI:** BYOK (OpenAI, Anthropic, Gemini, Kimi)
- **Hosting:** Vercel

---

## Documentation

Full technical documentation in [`docs/`](docs/):

| Document | Description |
|----------|-------------|
| [Problem Statement](docs/problem-statement.md) | The coaching industry problems |
| [Solution Concept](docs/solution-concept.md) | How OpenCoach solves them |
| [Business Model](docs/business-model.md) | Pricing & monetization |
| [Competitive Analysis](docs/competitive-analysis.md) | vs Coachvox, BetterUp, etc. |
| [Technical Architecture](docs/technical-architecture.md) | System design & stack |
| [Conversation Memory](docs/conversation-memory.md) | 4-layer memory system |
| [Learning System](docs/learning-system.md) | How AI learns from humans |
| [Risks & Mitigations](docs/risks-and-mitigations.md) | What could go wrong |
| [Roadmap](docs/roadmap.md) | 18-month development plan |
| [Decisions Log](docs/decisions-log.md) | Key architectural decisions |

---

## Roadmap

| Phase | Timeline | Focus |
|-------|----------|-------|
| **0: Foundation** | Weeks 1-2 | Coach interviews, landing page, waitlist |
| **1: MVP** | Weeks 3-10 | Web app, core AI, basic memory |
| **2: Telegram** | Weeks 11-14 | Bot integration, AI improvements |
| **3: Launch** | Weeks 15-22 | Public launch, first 100 coaches |
| **4: Scale** | Months 6-9 | WhatsApp, enterprise features |
| **5: Platform** | Months 10-18 | Marketplace, ecosystem |

---

## Differentiation

| Competitor | Their Model | OpenCoach |
|------------|-------------|-----------|
| **Coachvox** | AI-only clone | ✅ AI + human hybrid |
| **BetterUp** | Enterprise only | ✅ Individual coaches first |
| **ChatGPT** | Generic advice | ✅ Coach's knowledge + style |
| **Simply.Coach** | Admin tools | ✅ AI does the coaching |

**Our Moat:** The learning flywheel — more coaching → more learning → better AI → scale

---

## Origin

OpenCoach was conceptualized by **Akmal** (founder of [OKR Institute](https://okrinstitute.org), [Pandai](https://pandai.org), [FlowyTeam](https://flowyteam.com)) in collaboration with **Aime** (AI assistant) in February 2026.

The concept leverages Akmal's deep experience in coaching (OKR Institute) combined with modern AI capabilities.

---

## Contributing

This project is currently in **concept phase**. We welcome:
- Feedback from coaches
- Technical contributions
- Design input
- Beta testers

Contact: [Your contact info]

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

> *"AI will replace thousands of traditional coaches. Only coaches using AI can sustain."*

**OpenCoach:** AI-amplified humans, not AI-replaced humans.
