# 📝 Decisions Log

## Overview

This document records significant decisions made during the development of OpenCoach. It provides context for why decisions were made and allows future team members (or future us) to understand the reasoning.

**Format:**
- **Date:** When decision was made
- **Decision:** What was decided
- **Context:** Why the decision was needed
- **Options Considered:** Alternatives evaluated
- **Rationale:** Why this option was chosen
- **Consequences:** Impact of the decision
- **Reversible?** Can we change this later?

---

## Decision 001: Project Codename

**Date:** February 13, 2026  
**Decision:** Codename the project "open-coach"

**Context:** Need a working name for the project folder and internal references.

**Options Considered:**
1. coach-ai
2. aicoach
3. open-coach
4. clonecoach

**Rationale:**
- "open" suggests transparency and accessibility
- "coach" is clear about the domain
- Not too generic, not too specific
- Easy to say and spell

**Consequences:**
- Folder name: `projects/open-coach/`
- Internal references use "OpenCoach"
- Final product name may differ

**Reversible?** ✅ Yes, easily changed before launch.

---

## Decision 002: Documentation Structure

**Date:** February 13, 2026  
**Decision:** Create comprehensive markdown documentation in project folder

**Context:** Need to capture all ideas, research, and plans from initial brainstorming session.

**Options Considered:**
1. Single README.md
2. Wiki or Notion
3. Multiple structured .md files
4. Just keep in head/conversations

**Rationale:**
- Multiple files allow better organization
- Markdown is portable and version-controlled
- Works with existing knowledge base system
- Easy to share and collaborate

**Files Created:**
- README.md (overview)
- problem-statement.md
- solution-concept.md
- business-model.md
- competitive-analysis.md
- technical-architecture.md
- risks-and-mitigations.md
- roadmap.md
- decisions-log.md (this file)

**Consequences:**
- 9 files to maintain
- Clear structure for future reference
- Easy to onboard new team members

**Reversible?** ✅ Yes, can restructure anytime.

---

## Decision 003: Start with Web, Not WhatsApp

**Date:** February 13, 2026  
**Decision:** Build web MVP first, add messaging (Telegram/WhatsApp) later

**Context:** WhatsApp integration is complex (API approval, costs, legal). Need to validate core product first.

**Options Considered:**
1. Build WhatsApp first (the "ideal" experience)
2. Build web first, add WhatsApp later
3. Build both in parallel

**Rationale:**
- Web is fastest to build and iterate
- No external dependencies (Meta approval)
- Can validate AI quality before scaling
- Lower technical risk
- Telegram can be added in ~1 month when ready

**Trade-offs:**
- ✅ Faster time to market
- ✅ Easier iteration
- ✅ Lower risk
- ❌ Less "viral" initially
- ❌ Not meeting coaches where they are (yet)

**Consequences:**
- Phase 1 = web only
- Phase 2 = add Telegram
- Phase 3 = apply for WhatsApp API

**Reversible?** ⚠️ Partially. Web will always be needed, but delaying WhatsApp may slow adoption.

---

## Decision 004: BYOK (Bring Your Own Key) Model

**Date:** February 13, 2026  
**Decision:** Coaches bring their own AI API keys (OpenAI, Anthropic, etc.)

**Context:** Need to decide who pays for AI compute - us or the coaches.

**Options Considered:**
1. **We provide AI** (traditional SaaS)
   - We pay OpenAI, charge coaches markup
   - Simple pricing for coaches
   - High costs for us, margin pressure

2. **BYOK** (Bring Your Own Key)
   - Coaches pay OpenAI directly
   - We charge only for platform
   - Zero AI costs for us

**Rationale:**
- BYOK aligns incentives (coaches control quality/cost)
- Transparent pricing (no hidden AI costs)
- Sustainable unit economics
- Differentiator from competitors
- Future-proof as AI costs drop

**Trade-offs:**
- ✅ Zero AI infrastructure costs
- ✅ Transparent to coaches
- ✅ Sustainable margins
- ❌ More complex setup for coaches
- ❌ Can't guarantee AI quality (they choose model)

**Mitigation for complexity:**
- Clear setup guides
- Recommended models
- Default to GPT-4 (most coaches will use this)

**Consequences:**
- Pricing is pure SaaS (no usage variables)
- Coaches need OpenAI account
- We focus on platform value

**Reversible?** ⚠️ Hard to reverse. If we switch to "we provide AI" later, we'd need to renegotiate with all customers.

---

## Decision 005: Target Individual Coaches First, Not Enterprise

**Date:** February 13, 2026  
**Decision:** Start with individual coaches/small practices, expand to enterprise later

**Context:** Need to choose initial target market.

**Options Considered:**
1. **Individual coaches** (B2C/SMB)
   - Large market (100K+ coaches)
   - Faster sales cycle
   - Lower ACV, higher volume
   - Easier to reach

2. **Enterprise** (B2B)
   - Smaller market
   - Longer sales cycle
   - Higher ACV, lower volume
   - Harder to reach

**Rationale:**
- Individual coaches are underserved (BetterUp/CoachHub focus on enterprise)
- Faster feedback loop
- Can build product with real users
- OKR Institute network provides initial access
- Enterprise features can be added later

**Trade-offs:**
- ✅ Faster to first revenue
- ✅ More user feedback
- ✅ Clear differentiation
- ❌ Lower initial revenue per customer
- ❌ More support burden
- ❌ Higher churn risk

**Consequences:**
- Pricing optimized for individuals ($49-149/mo)
- Features focused on solo practitioners
- Marketing to individual coaches
- Enterprise tier added in Phase 4

**Reversible?** ✅ Yes, can add enterprise focus anytime.

---

## Decision 006: Use Supabase for Backend

**Date:** February 13, 2026  
**Decision:** Use Supabase (PostgreSQL + Auth + Realtime) as backend infrastructure

**Context:** Need database, auth, and real-time capabilities.

**Options Considered:**
1. **Supabase**
   - Open source, can self-host
   - Built-in auth
   - Real-time subscriptions
   - Vector support (pgvector)
   - Generous free tier

2. **Firebase**
   - Google's platform
   - Mature ecosystem
   - Vendor lock-in concerns
   - Less SQL-like

3. **Self-hosted PostgreSQL + Auth0**
   - Full control
   - More setup/maintenance
   - Higher DevOps burden

**Rationale:**
- Supabase has everything needed in one platform
- Open source = no lock-in (can migrate)
- PostgreSQL is standard (easy to hire for)
- Real-time features needed for chat
- Vector support for RAG

**Trade-offs:**
- ✅ All-in-one solution
- ✅ Open source
- ✅ Easy to start
- ❌ Younger platform (some rough edges)
- ❌ Real-time has limits (need to architect carefully)

**Consequences:**
- All data in PostgreSQL
- Auth handled by Supabase
- Real-time features via Supabase
- Vector search via pgvector

**Reversible?** ⚠️ Moderate effort. Database migration is always possible but requires work.

---

## Decision 007: Hybrid AI + Human Model

**Date:** February 13, 2026  
**Decision:** Build a hybrid system where AI handles routine coaching and humans can take over

**Context:** Core product decision - how much AI vs human?

**Options Considered:**
1. **AI-only** (like Noom)
   - Fully automated
   - Scales infinitely
   - Lower perceived value
   - Can't handle complex situations

2. **Human-only** (traditional)
   - No AI value
   - Doesn't scale
   - Not what we're building

3. **Hybrid** (AI default, human handoff)
   - AI handles 80% of conversations
   - Human steps in for complex moments
   - Best of both worlds

**Rationale:**
- Pure AI can't handle nuanced coaching
- Pure human doesn't scale
- Hybrid gives availability + judgment
- Differentiator from Coachvox (AI-only)
- Coaches feel in control

**Trade-offs:**
- ✅ Scalable (AI handles most)
- ✅ High quality (human backup)
- ✅ Coach control
- ❌ Complex to build (handoff logic)
- ❌ Requires both AI and UX excellence

**Consequences:**
- Need handoff detection system
- Need smooth UX for handoff
- Need context passing (AI summary for human)
- This is a core differentiator

**Reversible?** ⚠️ Hard. This is core to the product identity.

---

## Decision 008: Open Source vs Closed Source (Undecided)

**Date:** February 13, 2026  
**Status:** DECISION PENDING

**Context:** Should we open source the platform or keep it proprietary?

**Options to Evaluate:**
1. **Fully open source**
   - GitHub public repo
   - Community contributions
   - Harder to monetize
   - Builds trust/audience

2. **Open core**
   - Core platform open
   - Premium features closed
   - Best of both worlds

3. **Closed source**
   - Private repo
   - Full control
   - Standard SaaS model

**Considerations:**
- Open source = marketing/SEO benefits
- Open source = community contributions
- Closed source = easier to build business
- Can always open source later

**Decision Needed By:** Before public launch (Week 15)

---

## Decision 009: Use RAG (Not Simple File Reading)

**Date:** February 16, 2026  
**Decision:** Implement full RAG (Retrieval-Augmented Generation) system for coach knowledge bases

**Context:** Coaches will upload 100-1000+ documents (PDFs, videos, notes). Need to decide how to make this content available to AI.

**Options Considered:**
1. **Simple file reading** — Read documents into prompt
   - Simple to implement
   - ❌ Context window limits (can't read 100MB)
   - ❌ Expensive (millions of tokens)
   - ❌ No semantic search

2. **RAG with vector search** — Embeddings + similarity search
   - Scales to unlimited documents
   - Semantic matching ("fraud" → "imposter syndrome")
   - Cost-effective
   - More complex to build

**Rationale:**
- Coaches will have 10-100MB of content (impossible to read all)
- Need semantic search (client doesn't use exact keywords from docs)
- RAG is standard for document Q&A systems
- Supabase pgvector makes it accessible

**Trade-offs:**
- ✅ Scales infinitely
- ✅ Semantic understanding
- ✅ Cost-effective at scale
- ❌ More complex (embeddings, chunking, vector DB)
- ❌ Added latency (~100-500ms)

**Implementation:**
- Supabase pgvector for vector storage
- OpenAI text-embedding-3-small for embeddings
- Chunk documents into ~500 token pieces
- Cosine similarity search

**Consequences:**
- More complex architecture
- Need document processing pipeline
- Higher initial dev effort
- Future-proof for scale

**Reversible?** ⚠️ Hard. Core architecture decision.

---

## Decision 010: 4-Layer Memory System

**Date:** February 16, 2026  
**Decision:** Implement 4-layer persistent memory for coach-client relationships

**Context:** Coaching is relational — context from 6 months ago matters. Need more than simple chat history.

**Options Considered:**
1. **Simple chat history** — Store all messages, retrieve recent
   - Easy to implement
   - ❌ Doesn't capture insights
   - ❌ Expensive to include all history
   - ❌ No structured understanding

2. **4-layer memory** — Short-term + Summaries + Insights + Relationship
   - Captures different time horizons
   - Structured + unstructured
   - Efficient (compact representations)
   - More complex

**Layers Chosen:**
1. **Short-term** — Last 10 messages (immediate context)
2. **Session summaries** — AI-generated session overviews
3. **Key insights** — Structured facts (goals, blockers, breakthroughs)
4. **Relationship context** — Meta-information about coaching dynamic

**Rationale:**
- Different information has different shelf life
- Short-term: conversation flow
- Summaries: reference past sessions
- Insights: persistent facts about client
- Relationship: coaching style/preferences

**Consequences:**
- More database tables
- Complex context building logic
- Extraction/analysis prompts needed
- Richer, more personalized coaching

**Reversible?** ⚠️ Moderate. Core feature, but could simplify later.

---

## Decision 011: Learning System (3 Levels)

**Date:** February 16, 2026  
**Decision:** Build learning system where AI improves from every human intervention

**Context:** AI should get better over time by learning from human coach's style and effective techniques.

**Options Considered:**
1. **Static AI** — AI doesn't learn, always same behavior
   - Simple
   - ❌ Doesn't improve
   - ❌ Not personalized

2. **Learning system** — Analyze human interventions, adapt AI
   - AI improves continuously
   - Personalized to coach style
   - Client-specific adaptations
   - Complex to build

**Levels Chosen:**
1. **Client-specific** — "What works for THIS client"
2. **Coach-specific** — "THIS coach's unique style"
3. **Universal** — "What works across all coaches"

**Rationale:**
- Every human intervention is a learning opportunity
- Creates flywheel: better AI → less human intervention needed → scale
- Differentiator from generic AI tools
- Authentic coach voice preserved

**Mechanism:**
- Analyze human interventions (style, technique, phrases)
- Update client profile (effective approaches)
- Update coach style profile (signature style)
- Contribute to universal patterns (cross-coach learning)

**Consequences:**
- Complex analysis engine needed
- Database tables for learning data
- Dashboard for coaches to see AI learning
- Continuous improvement over time

**Reversible?** ⚠️ Hard. Core to product value proposition.

---

## Decision 012: Message Attribution System

**Date:** February 16, 2026  
**Decision:** Tag every message with sender type (client/AI/human_coach) and handoff metadata

**Context:** Need to track when human takes over from AI for learning system.

**Options Considered:**
1. **Simple sender field** — Just "client" or "coach"
   - Easy
   - ❌ Can't distinguish AI vs human coach
   - ❌ No handoff tracking

2. **Rich attribution** — sender_type + metadata
   - Tracks AI vs human
   - Records handoff reasons
   - Enables learning analysis

**Metadata Captured:**
- `sender_type`: 'client' | 'ai' | 'human_coach'
- `is_handoff`: Did human take over from AI?
- `handoff_reason`: Why? ('crisis', 'low_confidence', etc.)
- `ai_confidence_before`: AI confidence before handoff

**Rationale:**
- Learning system needs to know which responses were human
- Handoff detection enables intervention analysis
- Understanding why human took over improves AI

**Consequences:**
- More complex message table
- Handoff detection logic needed
- UI for handoff/handback

**Reversible?** ✅ Yes, but would lose learning data.

---

## Decision 013: Cost-Minimizing Infrastructure (Supabase over AWS)

**Date:** February 16, 2026  
**Decision:** Stick with Supabase and cost-minimizing services, avoid AWS for MVP and early stages

**Context:** Evaluated AWS Bedrock Knowledge Bases vs current Supabase + BYOK stack. Need to decide infrastructure path for cost optimization.

**Options Considered:**

1. **AWS Bedrock Knowledge Bases + AWS Infrastructure**
   - Managed RAG, multiple AI models
   - Enterprise scalability
   - Built-in compliance
   - Cost: $500-1,000/mo for 100 coaches
   - ❌ Conflicts with BYOK model (AWS manages AI)
   - ❌ High vendor lock-in
   - ❌ Complex setup (IAM, VPC, Lambda)
   - ❌ 10x more expensive than Supabase

2. **Supabase + Vercel + BYOK (Current Plan)**
   - Supabase: PostgreSQL + Auth + Realtime + pgvector
   - Vercel: Frontend hosting + serverless functions
   - BYOK: Coaches bring own AI keys
   - Cost: ~$50-100/mo for 100 coaches
   - ✅ Aligns with BYOK model
   - ✅ Open source (no lock-in)
   - ✅ Fast to build (weeks not months)
   - ✅ 10x cheaper

**Rationale:**
- MVP needs speed and low burn rate
- BYOK model requires coaches use their own AI keys (not AWS's AI)
- Supabase provides everything needed: database, auth, real-time, vector search
- Can always migrate to AWS later for enterprise tier
- Cost difference: $500-1,000/mo (AWS) vs $50-100/mo (Supabase)

**Cost Breakdown (100 coaches, 1,000 queries/day each):**

| Component | AWS Bedrock | Supabase + BYOK |
|-----------|-------------|-----------------|
| Vector Storage | $100-200/mo | $0 (pgvector free) |
| Embeddings | $0.10/1K docs | $0.02/1M tokens |
| AI Inference | $0.10-0.20/1K tokens | $0 (BYOK) |
| Compute | $50-100/mo | $0-20/mo |
| **Total You Pay** | **$500-1,000/mo** | **~$50-100/mo** |
| **Coach Pays** | Included | ~$20-50/mo (their AI) |

**Stack Confirmed:**
- **Database:** Supabase (PostgreSQL + pgvector)
- **Hosting:** Vercel (frontend + API routes)
- **AI:** BYOK (coaches use OpenAI/Anthropic/Gemini)
- **Storage:** Supabase Storage (knowledge base files)
- **Real-time:** Supabase Realtime (live messages)

**Future AWS Migration Path:**
- Keep Supabase for MVP through 1,000 coaches
- Add AWS as "Enterprise Tier" option later
- Offer AWS for clients who need SOC2/HIPAA compliance
- Maintain BYOK tier for individual coaches

**Trade-offs:**
- ✅ 10x cheaper infrastructure
- ✅ Faster MVP development
- ✅ No vendor lock-in
- ✅ Perfect fit for BYOK model
- ❌ Less enterprise-ready (compliance)
- ❌ Smaller free tier limits than AWS
- ❌ Younger platform (some rough edges)

**Consequences:**
- Infrastructure cost: ~$50-100/mo for early stage
- Can serve 1,000+ coaches before needing to scale infra
- Clear migration path to AWS for enterprise clients later
- Focus budget on product, not infrastructure

**Reversible?** ✅ Yes. Can migrate to AWS anytime if needed. Database is PostgreSQL (portable), code is Next.js (runs anywhere).

---

## Open Questions

Questions that need decisions in future:

1. **Mobile strategy:** Native apps vs PWA?
2. **Internationalization:** Which markets first?
3. **AI providers:** Support beyond OpenAI/Claude/Gemini?
4. **Pricing:** Should we offer annual discounts?
5. **Partnerships:** Integrate with coaching certification bodies?
6. **Video:** Add video coaching features?
7. **Voice:** Voice message support?
8. **Open source:** Open core vs fully closed?

---

## How to Add New Decisions

When making significant decisions:

1. Create new entry in this file
2. Follow the format above
3. Include all perspectives considered
4. Note if reversible
5. Date the decision
6. Review periodically (some decisions may need revisiting)

---

*This log is living documentation. Update as decisions are made or revisited.*
