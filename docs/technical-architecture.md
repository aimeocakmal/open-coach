# 🏗️ Technical Architecture

## Overview

OpenCoach is a modern SaaS platform built on a serverless/microservices architecture. The system handles real-time messaging, AI orchestration, persistent memory at scale, and continuous learning from human interventions.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
├─────────────────────────────────────────────────────────────────┤
│  Web App  │  Telegram Bot  │  WhatsApp (future)  │  Mobile App  │
└──────────────────┬──────────────────────────────────────────────┘
                   │ HTTPS/WebSocket
┌──────────────────▼──────────────────────────────────────────────┐
│                       GATEWAY LAYER                             │
│              (Cloudflare/AWS API Gateway)                       │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│                      API LAYER (Node.js)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │  Auth    │  │ Coaches  │  │ Clients  │  │ Messages │       │
│  │ Service  │  │ Service  │  │ Service  │  │ Service  │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│                   AI ORCHESTRATION LAYER                        │
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐   │
│  │           AI Coach Clone Engine                        │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│  │  │  Prompt  │  │ Context  │  │ Response │            │   │
│  │  │ Engineer │  │ Manager  │  │ Handler  │            │   │
│  │  └──────────┘  └──────────┘  └──────────┘            │   │
│  └──────────────────┬────────────────────────────────────┘   │
│                     │                                         │
│  ┌──────────────────▼────────────────────────────────────┐   │
│  │              Handoff Detection System                  │   │
│  │  • Sentiment Analysis  • Keyword Detection            │   │
│  │  • Confidence Scoring  • Escalation Rules             │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐   │
│  │         Learning & Adaptation Engine                   │   │
│  │  • Style Extraction    • Pattern Recognition          │   │
│  │  • Intervention Analysis • Continuous Improvement     │   │
│  └────────────────────────────────────────────────────────┘   │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│                      DATA LAYER                                 │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   PostgreSQL │  │    Redis     │  │ Vector Store │         │
│  │  (Primary DB)│  │   (Cache)    │  │  (Pinecone/  │         │
│  │              │  │              │  │   Supabase)  │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐                           │
│  │ Object Store │  │   Queue      │                           │
│  │   (S3/R2)    │  │  (Redis/     │                           │
│  │              │  │   SQS)       │                           │
│  └──────────────┘  └──────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Systems

### 1. RAG (Retrieval-Augmented Generation)

OpenCoach **must** implement RAG because coaches upload 100-1000+ documents (PDFs, videos, notes) that exceed context window limits.

**Document Processing Pipeline:**
```
Coach Uploads Document
        ↓
┌───────────────────┐
│ 1. Extract Text   │  ← PDF, DOCX, video transcript
│    (OCR if needed)│
└─────────┬─────────┘
          ↓
┌───────────────────┐
│ 2. Chunk Content  │  ← Split into ~500 token chunks
│    (with overlap) │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│ 3. Generate       │  ← OpenAI text-embedding-3-small
│    Embeddings     │    Cost: ~$0.02/1M tokens
└─────────┬─────────┘
          ↓
┌───────────────────┐
│ 4. Store in       │  ← Supabase pgvector
│    Vector DB      │
└───────────────────┘
```

**Query Flow:**
```javascript
// When client sends message
const relevantChunks = await vectorSearch({
  query: "I'm struggling with work-life balance",
  coachId: coachId,
  topK: 5,
  similarityThreshold: 0.7
});

// Retrieved chunks automatically injected into AI prompt
// "work-life balance strategies" → 0.92 similarity
// "burnout prevention framework" → 0.89 similarity
// etc.
```

**Tech Stack:**
| Component | Technology | Cost |
|-----------|------------|------|
| Vector DB | Supabase pgvector | Free tier: 500MB |
| Embeddings | OpenAI text-embedding-3-small | $0.02/1M tokens |
| Chunking | LangChain or custom | - |
| Retrieval | Cosine similarity | - |

See [conversation-memory.md](conversation-memory.md) for full memory system details.

---

### 2. Conversation Memory System (4 Layers)

Unlike simple chat history, OpenCoach maintains **multi-layered persistent memory**:

```
┌─────────────────────────────────────────┐
│  LAYER 1: Short-Term (Last 10 messages) │
│  → Full text, included in every prompt  │
├─────────────────────────────────────────┤
│  LAYER 2: Session Summaries             │
│  → AI-generated summaries of sessions   │
├─────────────────────────────────────────┤
│  LAYER 3: Key Insights (Extracted)      │
│  → Goals, blockers, breakthroughs       │
├─────────────────────────────────────────┤
│  LAYER 4: Relationship Context          │
│  → Personality, preferences, progress   │
└─────────────────────────────────────────┘
```

**Example:** Client "Sarah" mentions work-life balance crisis
- **Layer 1:** Last 10 messages (immediate context)
- **Layer 2:** Previous session summaries ("Discussed boundaries 3 days ago")
- **Layer 3:** Insights ("Fear of disappointing others" blocker identified)
- **Layer 4:** Relationship ("Prefers direct, evidence-based reassurance")

**Result:** AI response: *"I hear you — this anxiety spike before big events is a pattern we've seen. Remember what you realized two weeks ago? This fear isn't about tomorrow — it's that 3rd-grade voice talking. Let's use the grounding technique from your course..."*

See [conversation-memory.md](conversation-memory.md) for implementation details.

---

### 3. Learning from Human Coaches

**The Core Flywheel:** Every human intervention makes the AI smarter.

```
Coachee Message
      ↓
AI Responds ←── Default path
      ↓
Human Takes Over ←── Low confidence / High stakes
      ↓
┌─────────────────────────────────────┐
│ System Analyzes:                    │
│ • Style (tone, pace, formality)     │
│ • Technique (question, reflection)  │
│ • Key phrases                       │
│ • Why it worked                     │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ Updates:                            │
│ • Client profile (what works)       │
│ • Coach style profile               │
│ • Universal patterns                │
└─────────────┬───────────────────────┘
              ↓
        AI Gets Smarter
              ↓
        Next Time: Better Response
```

**Three Levels of Learning:**

| Level | What | Example |
|-------|------|---------|
| **Client-specific** | "What works for Sarah" | Sarah responds to "pause + grounding" in crisis |
| **Coach-specific** | "Akmal's style" | Uses sports analogies, opens with "Let's get real..." |
| **Universal** | "What works for everyone" | 87% success rate for "crisis → pause → question" pattern |

See [learning-system.md](learning-system.md) for full implementation.

---

### 4. Message Attribution System

Every message tracked by source:

```javascript
interface Message {
  id: string;
  sender_type: 'client' | 'ai' | 'human_coach';
  sender_id: string; // coach_id or 'ai_clone'
  
  metadata: {
    is_handoff: boolean;           // Human took over from AI?
    is_handback: boolean;          // AI resumed?
    handoff_reason: string;        // 'crisis', 'low_confidence', etc.
    ai_confidence_before: number;  // 0.0 - 1.0
    human_duration_minutes: number;
  };
}
```

**Visual Flow:**
```
[10:00] Client: "I want to quit my job right now"
[10:01] AI: [Confidence: 0.4] → HANDOFF TRIGGERED
[10:08] HUMAN: "I hear you. Let's pause..."
          └── Tagged: sender_type='human_coach'
               metadata.is_handoff=true
               metadata.handoff_reason='high_stakes_decision'
[10:25] HUMAN: HANDBACK → AI resumes with learned style
```

---

## Phase 1: Web MVP (Months 1-3)

### Tech Stack

| Layer | Technology | Reason |
|-------|------------|--------|
| **Frontend** | Next.js 14 + Tailwind | React framework, great DX |
| **Backend** | Next.js API Routes | Full-stack in one codebase |
| **Database** | Supabase (PostgreSQL) | Auth, real-time, vector support |
| **AI** | OpenAI API (BYOK) | Coaches bring own keys |
| **Vector DB** | Supabase pgvector | RAG for knowledge base |
| **Storage** | Supabase Storage | Knowledge base files |
| **Hosting** | Vercel | Fast deploys, edge network |
| **Real-time** | Supabase Realtime | Live message updates |

---

## Database Schema (Key Tables)

```sql
-- Messages with attribution
CREATE TABLE messages (
  id UUID PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id),
  sender_type VARCHAR(20) CHECK (sender_type IN ('client', 'ai', 'human_coach')),
  sender_id VARCHAR(50), -- coach_id or 'ai_clone'
  content TEXT,
  metadata JSONB, -- handoff info, confidence scores
  timestamp TIMESTAMP DEFAULT NOW()
);

-- RAG: Document chunks with embeddings
CREATE TABLE knowledge_chunks (
  id UUID PRIMARY KEY,
  coach_id UUID REFERENCES coaches(id),
  document_id UUID,
  content TEXT,
  embedding VECTOR(1536), -- OpenAI embedding dimension
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Session summaries (Layer 2 memory)
CREATE TABLE session_summaries (
  id UUID PRIMARY KEY,
  conversation_id UUID,
  summary TEXT,
  topics TEXT[],
  insights TEXT[],
  action_items TEXT[],
  emotional_state VARCHAR(50),
  message_count INT
);

-- Client insights (Layer 3 memory)
CREATE TABLE client_insights (
  id UUID PRIMARY KEY,
  client_id UUID,
  insight_type VARCHAR(50), -- 'goal', 'blocker', 'breakthrough'
  content JSONB,
  effectiveness_score FLOAT,
  is_active BOOLEAN DEFAULT true
);

-- Human intervention analysis (for learning)
CREATE TABLE human_intervention_analyses (
  id UUID PRIMARY KEY,
  message_id UUID,
  style_tone VARCHAR(50),
  technique_type VARCHAR(50),
  key_phrases TEXT[],
  effectiveness_score FLOAT,
  transferable_to_others BOOLEAN
);

-- Coach style profiles (learned over time)
CREATE TABLE coach_style_profiles (
  id UUID PRIMARY KEY,
  coach_id UUID,
  style_signature JSONB,
  preferred_techniques JSONB,
  total_interventions_analyzed INT DEFAULT 0,
  last_updated TIMESTAMP
);
```

---

## Cost Estimates

### Per Coach, Monthly

| Component | Cost | Notes |
|-----------|------|-------|
| **Vector DB** | $0 | Supabase free tier: 500MB |
| **Embeddings** | ~$0.50 | One-time per document upload |
| **Query embeddings** | ~$1.00 | ~50K queries/month |
| **AI tokens** | $0 | BYOK - coach pays OpenAI |
| **Platform** | $49-149 | OpenCoach subscription |
| **Total** | **~$1.50/mo** + subscription | Negligible infra costs |

---

## Key Architectural Decisions

### 1. Why RAG (Not Simple File Reading)
- Coaches upload 100-1000+ documents (10-100MB)
- Can't read everything (context window limits)
- Need semantic search ("fraud" → "imposter syndrome")
- Scales to unlimited knowledge base size

### 2. Why 4-Layer Memory
- Short-term: Immediate conversation flow
- Session summaries: Reference past sessions
- Insights: Structured facts about client
- Relationship: Meta-context about coaching dynamic

### 3. Why Learning System
- AI improves with every human intervention
- Client-specific adaptation
- Coach style authenticity
- Universal best practices across all coaches

### 4. Why BYOK (Bring Your Own Key)
- Zero AI infrastructure costs
- Transparent pricing for coaches
- Sustainable unit economics
- Coaches control quality/cost

---

## Related Documentation

- [Conversation Memory System](conversation-memory.md) - 4-layer memory implementation
- [Learning System](learning-system.md) - How AI learns from human interventions
- [RAG Implementation](rag-implementation.md) - Vector search and document processing

---

*Last updated: February 16, 2026*
