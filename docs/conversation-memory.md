# 💬 Conversation Memory System

## Overview

Coaching is a **relationship** — context from conversation 6 months ago matters today. OpenCoach implements a **4-layer persistent memory system** that captures everything from immediate context to long-term relationship dynamics.

Unlike simple chatbots that forget everything, OpenCoach builds a **rich, evolving understanding** of each coaching relationship.

---

## The 4-Layer Memory Hierarchy

```
┌─────────────────────────────────────────┐
│  LAYER 1: Short-Term (Last 10 messages) │
│  → Full text, included in every prompt  │
│  → Immediate conversation flow          │
├─────────────────────────────────────────┤
│  LAYER 2: Session Summaries             │
│  → AI-generated summaries of sessions   │
│  → Reference past sessions quickly      │
├─────────────────────────────────────────┤
│  LAYER 3: Key Insights (Extracted)      │
│  → Goals, blockers, breakthroughs       │
│  → Structured, persistent facts         │
├─────────────────────────────────────────┤
│  LAYER 4: Relationship Context          │
│  → Personality, preferences, progress   │
│  → Meta-information about the dynamic   │
└─────────────────────────────────────────┘
```

---

## Layer 1: Short-Term Memory

### What It Is
- Last 10-20 messages in the current conversation
- Full text, chronological order
- Always included in AI prompt

### Implementation
```javascript
// Get recent messages
const recentMessages = await db.query(`
  SELECT sender_type, content, timestamp
  FROM messages
  WHERE conversation_id = $1
  ORDER BY timestamp DESC
  LIMIT 10
`, [conversationId]);

// Format for prompt (chronological)
const recentContext = recentMessages
  .reverse()
  .map(m => `${m.sender_type}: ${m.content}`)
  .join('\n');
```

### Why Only 10-20?
- Context window limits (~4K-8K tokens for conversation)
- Recent context is most relevant
- Older context captured in summaries (Layer 2)

### Example
```
client: I'm freaking out about tomorrow's presentation
ai: I hear you. Let's break this down. What specifically are you worried about?
client: I just know I'm going to forget everything
ai: That's a common fear. Let's look at the evidence — how many presentations have you given?
client: About 10, but this one is different
ai: What's different about this one?
client: It's to the executive team
[Current message]
```

---

## Layer 2: Session Summaries

### What It Is
- AI-generated summary after each "session" (conversation block)
- Captures key points without full text
- Triggered automatically or manually

### When to Generate
```javascript
// Trigger summary generation when:
const shouldGenerateSummary = (
  idleTime > 30 minutes ||           // Conversation paused
  messageCount >= 20 ||              // Every 20 messages
  explicitEndSession ||              // Coach/client ends session
  humanHandoffOccurred               // Human took over
);
```

### Generation Prompt
```javascript
async function generateSessionSummary(messages) {
  const prompt = `
Summarize this coaching session:

CONVERSATION:
${formatMessages(messages)}

Extract in JSON:
{
  "summary": "200-word overview",
  "topics": ["main topics discussed"],
  "insights": ["client realizations"],
  "action_items": ["commitments made"],
  "emotional_state": "client mood",
  "breakthroughs": ["key moments"],
  "coach_interventions": ["what worked"]
}
`;
  return await ai.generate(prompt, { format: 'json' });
}
```

### Example Session Summary
```json
{
  "session_date": "2026-02-15",
  "summary": "Client discussed work-life balance struggles. Realized they say 'yes' to everything due to fear of disappointing others. Breakthrough moment when connected this to childhood experience of parent criticizing failures. Committed to practicing 'no' once this week. Left session feeling hopeful but anxious about implementation.",
  "topics": ["boundaries", "people-pleasing", "work-life balance", "childhood patterns"],
  "insights": [
    "Fear of disappointment drives overcommitment",
    "Pattern stems from childhood criticism",
    "Awareness is first step to change"
  ],
  "action_items": [
    "Practice saying no once this week",
    "Journal about childhood memory when it comes up"
  ],
  "emotional_state": "hopeful_anxious",
  "breakthroughs": [
    {
      "description": "Connected overcommitment to childhood pattern",
      "impact": "high",
      "timestamp": "2026-02-15T10:30:00Z"
    }
  ],
  "coach_interventions": [
    "Asked about origin of pattern",
    "Reflected back client's words",
    "Normalized the experience"
  ]
}
```

### Database Schema
```sql
CREATE TABLE session_summaries (
  id UUID PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id),
  summary TEXT,
  topics TEXT[],
  insights TEXT[],
  action_items TEXT[],
  emotional_state VARCHAR(50),
  breakthroughs JSONB,
  coach_interventions TEXT[],
  message_count INT,
  start_timestamp TIMESTAMP,
  end_timestamp TIMESTAMP DEFAULT NOW()
);
```

---

## Layer 3: Key Insights (The "Gold")

### What It Is
- Structured data extracted from conversations
- Persistent facts about the client
- Updated continuously

### Data Model
```javascript
interface ClientInsights {
  client_id: string;
  
  // Goals (active, achieved, abandoned)
  goals: [{
    goal: string;
    status: 'active' | 'achieved' | 'abandoned';
    created_at: Date;
    progress_notes: string[];
    target_date?: Date;
  }];
  
  // Blockers (Current challenges)
  blockers: [{
    blocker: string;
    severity: 'high' | 'medium' | 'low';
    first_mentioned: Date;
    status: 'active' | 'resolved';
    resolution?: string;
    times_discussed: number;
  }];
  
  // Breakthroughs (Key moments)
  breakthroughs: [{
    description: string;
    date: Date;
    context: string;
    impact: 'low' | 'medium' | 'high';
    recurrence_trigger?: string; // When might this insight be relevant again
  }];
  
  // Patterns (Recurring themes)
  patterns: [{
    pattern: string;
    evidence: string[]; // Message IDs or quotes
    first_observed: Date;
    frequency: 'daily' | 'weekly' | 'monthly';
    triggers: string[];
  }];
  
  // Preferences (How they like to be coached)
  preferences: {
    communication_style: 'direct' | 'gentle' | 'analytical' | 'intuitive';
    check_in_frequency: 'daily' | 'weekly' | 'as_needed';
    preferred_exercises: string[];
    sensitivities: string[]; // Topics to approach carefully
    effective_approaches: string[]; // What has worked historically
  };
  
  // Progress Metrics
  progress: {
    overall_trend: 'improving' | 'stable' | 'declining';
    key_metrics: Record<string, number>; // Custom per client
    last_assessment: Date;
  };
}
```

### Insight Extraction
```javascript
// After each conversation, extract insights
async function extractInsights(conversationId, clientId) {
  const conversation = await getRecentMessages(conversationId, 50);
  
  const extractionPrompt = `
Analyze this coaching conversation and extract insights:

CONVERSATION:
${conversation}

Extract in JSON:
{
  "new_goals": ["goals mentioned or progress on existing"],
  "blockers": ["current challenges or obstacles"],
  "breakthroughs": ["aha moments or realizations"],
  "patterns": ["recurring behaviors or themes"],
  "preference_updates": ["how client responds to coaching"],
  "progress_indicators": ["signs of growth or regression"]
}

Be specific and quote relevant parts of the conversation.
`;
  
  const insights = await ai.generate(extractionPrompt, { format: 'json' });
  await mergeInsights(clientId, insights);
}
```

### Example: Before & After Extraction

**Before Conversation:**
```javascript
goals: [
  { goal: "Improve work-life balance", status: "active" }
]
blockers: []
```

**After Conversation:**
```javascript
goals: [
  { 
    goal: "Improve work-life balance", 
    status: "active",
    progress_notes: ["Identified root cause: fear of disappointing others"]
  },
  { 
    goal: "Practice saying no", 
    status: "active", 
    created_at: "2026-02-15",
    target_date: "2026-02-22"
  }
]

blockers: [
  {
    blocker: "Fear of disappointing others",
    severity: "high",
    first_mentioned: "2026-02-15",
    status: "active",
    times_discussed: 1
  }
]

breakthroughs: [
  {
    description: "Connected overcommitment to childhood pattern of parental criticism",
    date: "2026-02-15",
    impact: "high",
    recurrence_trigger: "When client mentions feeling overwhelmed or overcommitted"
  }
]

patterns: [
  {
    pattern: "Says yes to everything, then feels overwhelmed",
    evidence: ["message_123", "message_124"],
    first_observed: "2026-02-15",
    frequency: "weekly",
    triggers: ["New request at work", "Family obligations"]
  }
]
```

---

## Layer 4: Relationship Context

### What It Is
- Meta-information about the coaching relationship
- How long they've worked together
- Rapport level
- Coaching phase

### Data Model
```javascript
interface RelationshipContext {
  client_id: string;
  coach_id: string;
  
  // Relationship metadata
  relationship: {
    start_date: Date;
    session_count: number;
    total_messages: number;
    total_coaching_hours: number;
    coaching_phase: 'discovery' | 'action' | 'integration' | 'completion';
    rapport_level: 'building' | 'established' | 'strong';
  };
  
  // Coaching approach for this client
  coaching_approach: {
    style_adjustments: string[]; // "Use more direct language"
    effective_interventions: string[]; // What has worked
    ineffective_approaches: string[]; // What to avoid
    preferred_frameworks: string[]; // Which coach frameworks resonate
  };
  
  // Conversation patterns
  patterns: {
    typical_session_length: number; // minutes
    response_time_client: number; // avg minutes
    response_time_coach: number; // avg minutes
    common_topics: string[]; // Top 5
    emotional_journey: 'stable' | 'volatile' | 'improving';
    crisis_frequency: 'rare' | 'occasional' | 'frequent';
  };
  
  // Milestones
  milestones: [{
    date: Date;
    event: string;
    significance: string;
  }];
}
```

### Example Relationship Context
```javascript
{
  relationship: {
    start_date: "2025-08-15",
    session_count: 24,
    total_messages: 342,
    total_coaching_hours: 18.5,
    coaching_phase: "action",
    rapport_level: "strong"
  },
  
  coaching_approach: {
    style_adjustments: [
      "Use sports analogies (client responds well)",
      "Give time to reflect before asking next question"
    ],
    effective_interventions: [
      "Reframing anxiety as excitement",
      "Powerful questions about values",
      "Grounding techniques"
    ],
    ineffective_approaches: [
      "Direct advice giving (client resists)",
      "Homework assignments (doesn't complete)"
    ],
    preferred_frameworks: ["GROW", "Values-based coaching"]
  },
  
  patterns: {
    typical_session_length: 45,
    response_time_client: 12, // minutes
    response_time_coach: 8,
    common_topics: ["work stress", "confidence", "leadership"],
    emotional_journey: "improving",
    crisis_frequency: "occasional"
  },
  
  milestones: [
    { date: "2025-08-15", event: "First session", significance: "Relationship began" },
    { date: "2025-10-03", event: "Promotion at work", significance: "Major goal achieved" },
    { date: "2025-12-12", event: "First crisis intervention", significance: "Rapport deepened" }
  ]
}
```

---

## Combining All Layers: Full Context

### When Client Sends Message, Build Context:

```javascript
async function buildCoachingContext(clientId, coachId, currentMessage) {
  
  // Layer 1: Recent messages (immediate context)
  const recentMessages = await getRecentMessages(clientId, 10);
  
  // Layer 2: Previous session summaries (last 5 sessions)
  const recentSummaries = await getSessionSummaries(clientId, 5);
  
  // Layer 3: Key insights (structured data)
  const insights = await getClientInsights(clientId);
  
  // Layer 4: Relationship context
  const relationship = await getRelationshipContext(clientId, coachId);
  
  // RAG: Relevant knowledge from coach's documents
  const relevantKnowledge = await vectorSearch({
    query: currentMessage,
    coachId: coachId
  });
  
  // Combine into mega-prompt
  const context = `
RELATIONSHIP CONTEXT:
You have been coaching ${relationship.client_name} since ${relationship.start_date}. 
This is session #${relationship.session_count}. 
Rapport level: ${relationship.rapport_level}.
Coaching phase: ${relationship.coaching_phase}.
Typical session length: ${relationship.patterns.typical_session_length} minutes.

CLIENT PROFILE & INSIGHTS:
Active Goals: ${insights.goals.filter(g => g.status === 'active').map(g => g.goal).join(', ')}
Current Blockers: ${insights.blockers.filter(b => b.status === 'active').map(b => b.blocker).join(', ')}
Recent Breakthroughs: ${insights.breakthroughs.slice(-3).map(b => b.description).join('; ')}
Key Patterns: ${insights.patterns.map(p => p.pattern).join(', ')}
Preferences: ${JSON.stringify(insights.preferences)}
Effective Approaches: ${insights.preferences.effective_approaches.join(', ')}

RECENT SESSIONS (Summaries):
${recentSummaries.map(s => `- ${s.date}: ${s.summary}`).join('\n')}

RECENT CONVERSATION:
${recentMessages.map(m => `${m.sender}: ${m.content}`).join('\n')}

RELEVANT KNOWLEDGE FROM COACH'S MATERIALS:
${relevantKnowledge.map(k => k.content).join('\n\n')}

CURRENT MESSAGE FROM CLIENT:
${currentMessage}

Respond as the coach would, using all the context above. Reference previous conversations and insights where relevant.
`;
  
  return context;
}
```

---

## Real Example: Full Context in Action

**Client Message:** *"I'm freaking out about tomorrow's presentation"*

**Context Built:**
```
RELATIONSHIP: Coaching for 6 months, 24 sessions, strong rapport

CLIENT INSIGHTS:
- Goal: "Become confident speaker" (active since Jan)
- Blocker: "Fear of judgment" (high severity, mentioned 8 times)
- Breakthrough: "Realized fear is from 3rd grade humiliation" (2 weeks ago)
- Pattern: "Anxiety spikes 24hrs before big events"
- Preference: "Responds well to evidence-based reassurance"
- Effective approaches: "Reframing anxiety as excitement", "Grounding techniques"

RECENT SESSIONS:
- 3 days ago: Practiced presentation structure, client felt prepared
- 1 week ago: Discussed reframing anxiety as excitement

RECENT CONVERSATION:
AI: How are you feeling about the presentation prep?
Client: Good, I've practiced a lot
[2 hours pass]
Client: I'm freaking out about tomorrow's presentation

RELEVANT KNOWLEDGE:
- "Pre-presentation anxiety protocol" from coach's course
- "The 5-4-3-2-1 grounding technique"
```

**AI Response:**
> "Sarah, I hear you — this anxiety spike right before a big presentation is a pattern we've seen before (remember the client meeting last month?). Let's use what worked then: 
>
> First, the facts — you've practiced, you know the material. This feeling is your brain preparing you, not a sign you're unprepared.
>
> Remember what you realized two weeks ago? This fear isn't about tomorrow — it's that 3rd-grade voice talking. You've got this.
>
> Want to try the 5-4-3-2-1 grounding technique from your course? Name 5 things you see, 4 you can touch..."

---

## Context Window Management

### The Challenge
- GPT-4: ~128K context window
- Kimi K2.5: ~256K context window
- But we can't fill it all with conversation history

### Smart Truncation Strategy

```javascript
function smartContextTruncation(context, maxTokens = 8000) {
  // Priority order (highest to lowest)
  const priority = [
    'currentMessage',      // Always include (required)
    'recentMessages',      // Include full (10 messages)
    'clientInsights',      // Include full (structured, compact)
    'relationshipContext', // Include full (small)
    'relevantKnowledge',   // Include top 5 chunks
    'recentSummaries',     // Include last 3
    'olderSummaries',      // Summarize further if needed
  ];
  
  // Build context in priority order until limit reached
  let currentTokens = 0;
  const included = [];
  
  for (const section of priority) {
    const sectionTokens = estimateTokens(context[section]);
    if (currentTokens + sectionTokens <= maxTokens) {
      included.push(section);
      currentTokens += sectionTokens;
    } else {
      // Summarize or truncate
      const summary = summarizeSection(context[section]);
      included.push(summary);
      break;
    }
  }
  
  return included;
}
```

---

## Database Schema

```sql
-- Messages (Layer 1)
CREATE TABLE messages (
  id UUID PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id),
  sender_type VARCHAR(20) CHECK (sender_type IN ('client', 'ai', 'human_coach')),
  sender_id VARCHAR(50),
  content TEXT,
  ai_confidence FLOAT,
  tokens_used INT,
  metadata JSONB,
  timestamp TIMESTAMP DEFAULT NOW()
);

-- Session Summaries (Layer 2)
CREATE TABLE session_summaries (
  id UUID PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id),
  summary TEXT,
  topics TEXT[],
  insights TEXT[],
  action_items TEXT[],
  emotional_state VARCHAR(50),
  breakthroughs JSONB,
  message_count INT,
  start_timestamp TIMESTAMP,
  end_timestamp TIMESTAMP
);

-- Client Insights (Layer 3)
CREATE TABLE client_insights (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  insight_type VARCHAR(50), -- 'goal', 'blocker', 'breakthrough', 'pattern', 'preference'
  content JSONB,
  effectiveness_score FLOAT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  is_active BOOLEAN DEFAULT true
);

-- Relationship Context (Layer 4)
CREATE TABLE relationship_context (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  coach_id UUID REFERENCES coaches(id),
  relationship_data JSONB,
  coaching_approach JSONB,
  patterns JSONB,
  milestones JSONB,
  updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Summary

| Layer | What | When to Use | Storage |
|-------|------|-------------|---------|
| **Short-term** | Last 10 messages | Every response (immediate context) | Messages table |
| **Session summaries** | AI summaries | Reference past sessions | session_summaries table |
| **Insights** | Structured facts | Always (goals, blockers, patterns) | client_insights table |
| **Relationship** | Meta-context | Personalize coaching approach | relationship_context table |

**This multi-layered system is what makes OpenCoach different from ChatGPT:**
- Persistent, evolving memory
- Relationship continuity across months/years
- Insights extracted and remembered
- Context that grows stronger over time

---

*See also: [Learning System](learning-system.md) - How AI learns from human interventions*
