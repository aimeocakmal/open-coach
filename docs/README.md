# 🎯 OpenCoach

**Codename:** open-coach  
**Status:** Concept/Idea Phase  
**Created:** February 13, 2026  
**Last Updated:** February 16, 2026

---

## Elevator Pitch

OpenCoach is an AI-augmented coaching platform that enables coaches to scale their expertise through AI cloning while maintaining human touch. Unlike pure AI coaching tools, OpenCoach creates a hybrid model where AI handles routine coaching conversations and humans intervene when needed — all while preserving context and building persistent memory of each coaching relationship.

**The Secret Sauce:** Every human intervention makes the AI smarter. The system learns coach style, client preferences, and universal best practices — creating a flywheel of continuous improvement.

---

## The Core Insight

> "AI will replace thousands of traditional coaches. Only coaches using AI can sustain."

The future of coaching isn't AI-only or human-only — it's **AI-amplified humans**.

---

## Target Users

**Primary:** Business coaches, executive coaches, life coaches  
**Secondary:** Fitness coaches, career coaches, relationship coaches  
**Tertiary:** Internal L&D coaches at enterprises

---

## Key Differentiators

1. **Hybrid AI + Human Model** — AI handles 80%, humans handle complex moments
2. **Persistent Memory (4 Layers)** — Long-term context across AI and human sessions
3. **Learning System** — AI learns from every human intervention
4. **BYOK (Bring Your Own Key)** — Coaches use their own AI APIs (OpenAI, Gemini, Kimi)
5. **WhatsApp-First** (Eventual) — Meet coaches where they already are
6. **Knowledge Management** — RAG-based access to coach's documents

---

## Repository Structure

```
open-coach/
├── README.md                    # This file - overview
├── problem-statement.md         # The problem we're solving
├── solution-concept.md          # How we solve it
├── business-model.md            # Pricing & monetization
├── competitive-analysis.md      # Market landscape
├── technical-architecture.md    # Technical approach & stack
├── conversation-memory.md       # 4-layer memory system
├── learning-system.md           # How AI learns from humans
├── risks-and-mitigations.md     # What could go wrong
├── roadmap.md                   # Development phases
└── decisions-log.md             # Key decisions record
```

---

## Quick Links

### Business & Strategy
- [Problem Statement](problem-statement.md) — The coaching industry problems
- [Solution Concept](solution-concept.md) — How OpenCoach solves them
- [Business Model](business-model.md) — Pricing & BYOK model
- [Competitive Analysis](competitive-analysis.md) — vs Coachvox, BetterUp, etc.
- [Risks & Mitigations](risks-and-mitigations.md) — What could go wrong

### Technical
- [Technical Architecture](technical-architecture.md) — Next.js + Supabase stack
- [Conversation Memory](conversation-memory.md) — 4-layer persistent memory
- [Learning System](learning-system.md) — How AI learns from human interventions

### Execution
- [Roadmap](roadmap.md) — 18-month development plan
- [Decisions Log](decisions-log.md) — Key architectural decisions

---

## Market Context

| Metric | Value |
|--------|-------|
| Global Coaching Industry | $5.34B (2026) |
| Executive Coaching | $103.6B → $161.1B by 2030 |
| Coaching Platform Market | $3.7B → $10.4B by 2034 (10.9% CAGR) |
| Active Coaches Globally | 122,974+ (54% growth in 6 years) |

**Verdict:** Large, growing market with clear demand for technology solutions.

---

## Technical Highlights

### RAG (Retrieval-Augmented Generation)
- Vector search through coach's knowledge base
- Supabase pgvector for embeddings
- Semantic matching (not just keyword)

### 4-Layer Memory System
1. **Short-term** — Last 10 messages
2. **Session summaries** — AI-generated session overviews
3. **Key insights** — Structured facts (goals, blockers, breakthroughs)
4. **Relationship context** — Meta-information about coaching dynamic

### 3-Level Learning System
1. **Client-specific** — "What works for THIS client"
2. **Coach-specific** — "THIS coach's unique style"
3. **Universal** — "What works across all coaches"

---

## Origin

This project emerged from discussions between Akmal (founder of OKR Institute, Pandai, FlowyTeam) and Aime (AI assistant) on February 13-16, 2026. The concept leverages Akmal's deep experience in coaching (OKR Institute) combined with modern AI capabilities.

---

*Last updated: February 16, 2026*
