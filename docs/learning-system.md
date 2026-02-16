# 🧠 Learning System

## Overview

**The secret sauce of OpenCoach:** Every human intervention makes the AI smarter.

When a human coach takes over from AI, the system doesn't just log it — it **analyzes, extracts, and learns** from that intervention. Over time, the AI coaches more like the human coach, with personalized approaches for each client.

This creates a **flywheel effect:**
```
More Coaching → More Human Interventions → More Learning → 
Better AI → Less Human Intervention Needed → Scale
```

---

## The Core Learning Loop

```
┌─────────────────────────────────────────┐
│  1. AI Responds (Default)               │
│     ↓ Low confidence / High stakes      │
│  2. Human Takes Over                    │
│     ↓ Analysis Engine                   │
│  3. Extract Style & Technique           │
│     ↓ Update Profiles                   │
│  4. Learn & Adapt                       │
│     ↓ Next Time                         │
│  5. AI Uses Learned Approach            │
└─────────────────────────────────────────┘
```

---

## Message Attribution System

### Every Message Tagged by Source

```javascript
interface Message {
  id: string;
  conversation_id: string;
  sender_type: 'client' | 'ai' | 'human_coach';
  sender_id: string; // coach_id if human, 'ai_clone' if AI
  content: string;
  
  // Critical for learning
  metadata: {
    is_handoff: boolean;           // Did human take over from AI?
    is_handback: boolean;          // Did AI resume after human?
    handoff_reason: string;        // Why did human intervene?
    handoff_trigger: string;       // 'crisis', 'low_confidence', 'client_request'
    ai_confidence_before: number;  // 0.0 - 1.0
    ai_confidence_after: number;   // After handback
    human_duration_minutes: number;
    messages_in_intervention: number;
  };
  
  timestamp: Date;
}
```

### Handoff Detection

```javascript
// When should human take over?
function shouldHandoffToHuman(aiResponse, context) {
  let handoffScore = 0;
  
  // Crisis keywords
  if (containsCrisisKeywords(context.lastMessage)) {
    handoffScore += 0.5;
  }
  
  // Low AI confidence
  if (aiResponse.confidence < 0.6) {
    handoffScore += 0.3;
  }
  
  // High emotional intensity
  if (context.sentiment < -0.7) {
    handoffScore += 0.2;
  }
  
  // High-stakes decision
  if (containsHighStakesKeywords(context.lastMessage)) {
    handoffScore += 0.3;
  }
  
  // Client explicitly asked
  if (context.clientRequestedHuman) {
    handoffScore += 0.4;
  }
  
  return handoffScore > 0.7; // Threshold
}
```

### Visual Flow Example

```
[10:00] Client: "I'm having a crisis at work"
[10:01] AI: "I understand. Can you tell me more..."
        [Confidence: 0.75]
        
[10:05] Client: "I want to quit my job right now"
[10:06] AI: [Confidence: 0.4 - BELOW THRESHOLD]
        → HANDOFF TRIGGERED
        → Reason: 'high_stakes_decision'
        → Notify human coach
        
[10:06] SYSTEM: Human Coach notified

[10:08] HUMAN: "Sarah, stop. Take a breath. I can hear the panic. 
               Before we burn bridges, let's get you grounded.
               What's ONE thing you know for sure?"
        ↑
        └── Tagged: 
            sender_type='human_coach'
            metadata.is_handoff=true
            metadata.handoff_reason='high_stakes_decision'
            metadata.ai_confidence_before=0.4

[10:15] Client: "You're right, I should think this through"
[10:16] HUMAN: "Exactly. What's the core issue here?"
...
[10:25] HUMAN: HANDBACK → "Ready for AI to resume"
[10:26] AI: Resumes with learned style
```

---

## Human Intervention Analysis Engine

### What Gets Extracted

```javascript
interface HumanInterventionAnalysis {
  message_id: string;
  coach_id: string;
  client_id: string;
  
  // What the human did
  intervention: {
    style: {
      tone: 'empathetic' | 'direct' | 'analytical' | 'playful' | 'challenging';
      pace: 'fast' | 'measured' | 'slow' | 'deliberate';
      formality: 'casual' | 'professional' | 'intimate';
      energy: 'calm' | 'energetic' | 'urgent';
    };
    
    technique: {
      type: 'question' | 'reflection' | 'reframe' | 'challenge' | 
            'story' | 'silence' | 'direct_advice' | 'normalization';
      framework: string; // e.g., "GROW", "Socratic", "Motivational Interviewing"
      approach: string;  // e.g., "Asked powerful question", "Shared personal story"
      structure: string[]; // Step-by-step approach
    };
    
    content_analysis: {
      key_phrases: string[];      // Signature phrases coach used
      opening_pattern: string;    // How they started
      closing_pattern: string;    // How they ended
      emotional_intelligence: string[]; // How they handled emotions
      specific_advice: string;    // What they actually said (anonymized)
      metaphor_used: string;      // e.g., "burn bridges", "sports analogy"
    };
    
    timing: {
      response_time_seconds: number; // How fast human responded
      intervention_duration_minutes: number;
      messages_exchanged: number;
    };
  };
  
  // Why it worked (or didn't)
  outcome: {
    client_reaction: 'positive' | 'neutral' | 'negative' | 'transformative';
    conversation_shift: string; // How did conversation change?
    resolution_achieved: boolean;
    client_engagement: 'high' | 'medium' | 'low';
    follow_through_likely: number; // 0.0 - 1.0
  };
  
  // Learning metadata
  learning: {
    applicable_to_similar_situations: boolean;
    transferable_to_other_clients: boolean;
    confidence_score: number; // How sure are we this was effective
    replication_value: number; // How valuable to teach AI
  };
  
  created_at: Date;
}
```

### Analysis Prompt

```javascript
async function analyzeHumanIntervention(messageId, conversationContext) {
  const message = await getMessage(messageId);
  const beforeContext = await getMessagesBefore(messageId, 5);
  const afterContext = await getMessagesAfter(messageId, 5);
  
  const analysisPrompt = `
Analyze this human coach intervention:

SITUATION BEFORE (Last 5 messages):
${formatMessages(beforeContext)}

HUMAN COACH RESPONSE:
"${message.content}"

SITUATION AFTER (Next 5 messages):
${formatMessages(afterContext)}

Extract in JSON:
{
  "style": {
    "tone": "empathetic|direct|analytical|playful|challenging",
    "pace": "fast|measured|slow",
    "formality": "casual|professional|intimate",
    "energy": "calm|energetic|urgent"
  },
  "technique": {
    "type": "question|reflection|reframe|challenge|story|silence|direct_advice",
    "framework": "coaching framework evident",
    "approach": "detailed description of approach",
    "structure": ["step 1", "step 2", "step 3"]
  },
  "content_analysis": {
    "key_phrases": ["signature phrases used"],
    "opening_pattern": "how they opened",
    "closing_pattern": "how they closed",
    "emotional_intelligence": ["how they handled emotions"],
    "metaphor_used": "any metaphors or analogies"
  },
  "outcome": {
    "client_reaction": "positive|neutral|negative|transformative",
    "conversation_shift": "how conversation changed",
    "resolution_achieved": true|false
  },
  "learning": {
    "applicable_to_similar_situations": true|false,
    "transferable_to_other_clients": true|false,
    "why_effective": "reasoning"
  }
}

Be specific and actionable. Identify what made this intervention effective.
`;
  
  const analysis = await ai.generate(analysisPrompt, { format: 'json' });
  
  await db.insert('human_intervention_analyses', {
    message_id: messageId,
    ...analysis
  });
  
  return analysis;
}
```

---

## Three Levels of Learning

### Level 1: Client-Specific Learning

**What:** "What works for THIS specific client"

**How it works:**
```javascript
async function updateClientLearning(clientId, interventionAnalysis) {
  const situationType = classifySituation(interventionAnalysis.situation);
  
  // Add to client's "what works for me" profile
  await db.query(`
    INSERT INTO client_effective_approaches 
    (client_id, situation_type, approach, effectiveness, source, examples)
    VALUES ($1, $2, $3, $4, 'human_intervention', $5)
    ON CONFLICT (client_id, situation_type) 
    DO UPDATE SET 
      approach = EXCLUDED.approach,
      effectiveness = GREATEST(client_effective_approaches.effectiveness, EXCLUDED.effectiveness),
      times_validated = client_effective_approaches.times_validated + 1,
      updated_at = NOW()
  `, [
    clientId,
    situationType,           // e.g., "crisis_moment"
    interventionAnalysis.intervention,  // The approach used
    interventionAnalysis.outcome.client_reaction === 'positive' ? 0.9 : 0.5,
    [interventionAnalysis.content_analysis.key_phrases[0]] // Example phrase
  ]);
}
```

**Example:**
```
Client: Sarah
Situation: "Crisis moment - wants to quit job"
Human approach: "Pause + grounding + powerful question"
Effectiveness: 0.95 (client calmed down, gained clarity)
Key phrase: "Before we burn bridges"

→ Next time Sarah is in crisis
→ AI uses: "Sarah, stop. Take a breath. Before we burn bridges, 
             what's ONE thing you know for sure?"
```

---

### Level 2: Coach Style Learning

**What:** "THIS coach's unique style and techniques"

**How it works:**
```javascript
interface CoachStyleProfile {
  coach_id: string;
  
  // Evolving style profile
  style_signature: {
    tone: string;                    // "empathetic"
    typical_opening: string;         // "I hear you..."
    question_style: string;          // "powerful_reflective"
    energy_level: string;            // "calm_measured"
    metaphor_frequency: number;      // 0.25 (uses metaphors in 25% of responses)
  };
  
  // Preferred techniques (ranked by frequency & effectiveness)
  preferred_techniques: [{
    technique: string;               // "powerful_question"
    frequency: number;               // 0.45 (45% of interventions)
    effectiveness: number;           // 0.88 (88% success rate)
    examples: string[];              // ["What would you advise a friend?", ...]
    typical_contexts: string[];      // ["stuck_client", "decision_making"]
  }];
  
  // Frameworks used
  frameworks: [{
    name: string;                    // "GROW"
    usage_frequency: number;         // 0.6 (60% of sessions)
    adaptations: string;             // "Adds emotional check-ins between steps"
  }];
  
  // Situation-response mapping
  situation_responses: [{
    situation_type: string;          // "crisis"
    typical_response: string;        // "pause_and_ground"
    effectiveness: number;           // 0.91
    response_template: string;       // "[Name], stop. Take a breath..."
  }];
  
  // Signature phrases (auto-extracted)
  signature_phrases: [{
    phrase: string;                  // "Before we burn bridges"
    frequency: number;               // 0.15
    contexts: string[];              // ["crisis", "high_stakes"]
  }];
  
  // Stats
  total_interventions_analyzed: number;
  last_updated: Date;
}
```

**Example after 50 interventions:**
```
Coach: Akmal
Analysis of 50 human interventions:

Style Signature:
- Tone: Direct but warm
- Opens with: "Let's get real..." or "Here's what I see..."
- Uses analogy frequently (sports, nature) - 35% of interventions
- Challenges gently: "What if the opposite is true?"
- Energy: Calm but firm

Preferred Techniques:
1. Reframe (40% of time, 90% effective)
   Examples: "What if this is an opportunity?"
   Contexts: Negative thinking, victim mindset

2. Analogy (35% of time, 85% effective)
   Examples: "It's like training for a marathon..."
   Contexts: Long-term goals, patience needed

3. Direct advice (20% of time, 75% effective)
   Contexts: Client explicitly asks "what should I do?"

Frameworks:
- GROW model (60% of sessions, adds emotional check-ins)
- OKR framework (for goal-setting sessions)

Situation Responses:
- Crisis: "Pause + grounding" (91% effective)
- Resistance: "Gentle confrontation" (85% effective)
- Breakthrough: "Deepen the insight" (92% effective)

Signature Phrases:
- "Before we burn bridges" (crisis situations)
- "Let's get real for a moment" (cutting through BS)
- "What would [famous athlete] do?" (sports analogies)

→ AI learns to use sports analogies when coaching with Akmal's clone
→ Opens with "Let's get real..."
→ Uses reframing in similar situations
```

---

### Level 3: Universal Best Practices

**What:** "What works across ALL coaches and clients"

**How it works:**
```javascript
interface UniversalCoachingPattern {
  pattern_id: string;
  
  // The situation
  situation: {
    description: string;           // "Client in crisis wanting to quit"
    keywords: string[];            // ["quit", "leave", "can't take it", "done"]
    sentiment_range: [number, number]; // [-1, -0.5] (negative)
    urgency_indicators: string[];  // ["right now", "immediately", "today"]
    context_clues: string[];       // ["job", "relationship", "career"]
  };
  
  // Effective response
  effective_response: {
    technique: string;             // "pause_and_ground"
    approach: string;              // "Stop, breathe, question"
    opening_phrase_pattern: string; // "[Name], stop. Take a breath."
    structure: string[];           // ["acknowledge", "pause", "reframe", "question"]
    timing: string;                // "immediate"
  };
  
  // Evidence
  source_interventions: string[];  // IDs of human interventions
  coach_count: number;             // How many coaches used this
  client_count: number;            // How many clients this worked for
  success_rate: number;            // % of times this worked
  
  // Statistical validation
  statistical_significance: boolean;
  confidence_score: number;        // 0.0 - 1.0
  sample_size: number;
  
  // Usage tracking
  times_used_by_ai: number;
  ai_success_rate: number;
  
  created_at: Date;
  updated_at: Date;
}
```

**Example:**
```
Pattern: "High-stakes decision + emotional overwhelm"

Situation:
- Client wants to make major life change immediately
- High emotional intensity (sentiment < -0.6)
- Keywords: "quit", "leave", "done", "can't take it"
- Urgency: "right now", "immediately"

Effective Response (learned from 12 human coaches):
1. ACKNOWLEDGE: "I hear you're at your limit"
2. PAUSE: "Before we decide, let's breathe"
3. REFRAME: "This feeling is information, not instruction"
4. QUESTION: "What do you know for sure vs what are you assuming?"
5. COMMITMENT: "Let's decide tomorrow, not today"

Evidence:
- Source interventions: 47
- Coach count: 12 (different coaches)
- Client count: 35
- Success rate: 87%
- Statistical significance: Yes (p < 0.01)

AI Usage:
- Times used by AI: 45
- AI success rate: 82%
- Confidence score: 0.91

→ AI uses this pattern for ANY client in similar situation
→ Adapts opening to coach's style (Akmal: "Let's get real..." vs 
   Sarah's coach: "I hear you...")
```

---

## Real-Time Style Adaptation

### When AI Generates Response

```javascript
async function generateAIResponse(clientId, coachId, clientMessage, context) {
  
  // 1. Get all learning layers
  const clientLearning = await getClientEffectiveApproaches(clientId);
  const coachStyle = await getCoachStyleProfile(coachId);
  const universalPatterns = await findMatchingPatterns(clientMessage);
  const situation = analyzeSituation(clientMessage);
  
  // 2. Build style instructions
  const styleInstructions = buildStyleInstructions({
    coachStyle,
    clientLearning,
    universalPatterns,
    situation
  });
  
  // 3. Generate with learned style
  const prompt = `
You are coaching as ${coachStyle.name}.

YOUR COACHING STYLE (learned from ${coachStyle.total_interventions_analyzed} interventions):
- Tone: ${coachStyle.style_signature.tone}
- Typical opening: ${coachStyle.style_signature.typical_opening}
- Energy: ${coachStyle.style_signature.energy_level}
- You use analogies ${Math.round(coachStyle.style_signature.metaphor_frequency * 100)}% of the time

TECHNIQUES THAT WORK FOR YOU:
${coachStyle.preferred_techniques
  .slice(0, 3)
  .map(t => `- ${t.technique}: ${Math.round(t.effectiveness * 100)}% effective, used ${Math.round(t.frequency * 100)}% of the time`)
  .join('\n')}

${coachStyle.signature_phrases.length > 0 
  ? `SIGNATURE PHRASES (use naturally):\n${coachStyle.signature_phrases.slice(0, 3).map(p => `- "${p.phrase}"`).join('\n')}`
  : ''}

WHAT WORKS FOR THIS CLIENT:
${clientLearning.map(l => `- When ${l.situation_type}: ${l.approach} (${Math.round(l.effectiveness * 100)}% effective)`).join('\n')}

CURRENT SITUATION:
${universalPatterns 
  ? `This is a "${universalPatterns.situation.description}" situation. 
     Proven approach: ${universalPatterns.effective_response.technique}
     Structure: ${universalPatterns.effective_response.structure.join(' → ')}`
  : 'No specific pattern match - use your judgment'}

CLIENT MESSAGE: "${clientMessage}"

Respond in YOUR authentic coaching style, using techniques proven to work.
${coachStyle.frameworks.length > 0 ? `Consider using: ${coachStyle.frameworks[0].name} framework` : ''}
`;
  
  const response = await ai.generate(prompt);
  
  // 4. Log for future learning
  await logAIResponse(clientId, coachId, response, styleInstructions);
  
  return response;
}
```

---

## Complete Example: Learning Flow

### Scenario: Client "Sarah" in Crisis

**T+0: Initial AI Response (Generic)**
```
AI: "I'm sorry to hear you're struggling. Can you tell me more 
     about what's happening at work?"
[Confidence: 0.6]
```

**T+1: Human Coach Takes Over**
```
Human: "Sarah, stop. Take a breath. I can hear the panic in your words.
        Before we burn bridges, let's get you grounded. 
        What's ONE thing you know for sure about this situation?"

Tagged: sender_type='human_coach', is_handoff=true
```

**T+2: System Analyzes Intervention**
```javascript
Analysis:
- Style: Direct, grounding, urgent but calm
- Technique: Pause + grounding + powerful question
- Key phrases: "Before we burn bridges", "stop. take a breath"
- Structure: ["interrupt", "ground", "reframe", "question"]
- Effectiveness: Sarah calmed down, started thinking clearly
```

**T+3: Updates Made**

```javascript
// Client Learning (Sarah-specific)
"crisis_moment" → {
  approach: "Pause + grounding + one powerful question",
  effectiveness: 0.95,
  phrase: "Before we burn bridges",
  key_elements: ["interrupt pattern", "physical grounding", "singular focus"]
}

// Coach Style Learning (Akmal's style)
Updates profile:
- Adds "Before we burn bridges" to signature phrases
- Crisis response: "Direct grounding with metaphor"
- Technique effectiveness: Pause (90%), Grounding (95%), Powerful Q (88%)
- Increases "metaphor_frequency" (uses bridge metaphor)

// Universal Pattern
Adds evidence to "crisis_intervention" pattern:
- New source intervention added
- Success rate: 85% → 87%
- Coach diversity: 11 → 12 coaches
```

**T+4: Next Time Sarah is in Crisis**
```
AI: "Sarah, stop. Take a breath. I can hear this is intense.
     Before we burn bridges, let's get you grounded.
     What's ONE thing you know for sure right now?"
     
[Uses Akmal's direct style]
[Uses proven technique for Sarah]
[Uses phrase that worked before]
[Confidence: 0.85]
```

**T+5: Next Time DIFFERENT Client is in Crisis**
```
AI: "[Client], I can hear you're at your limit. 
     Before making any big moves, let's pause.
     What's one thing you know for sure about this situation?"
     
[Uses universal pattern learned from 12 coaches]
[Adapted to new client's communication style]
[Uses proven structure: acknowledge → pause → question]
[Confidence: 0.82]
```

---

## Dashboard: Learning Visualization

### For Coaches

```
┌─────────────────────────────────────────┐
│  YOUR AI IS LEARNING FROM YOU           │
├─────────────────────────────────────────┤
│                                         │
│  Interventions Analyzed: 47             │
│  Learning Since: Jan 15, 2026           │
│                                         │
│  YOUR SIGNATURE STYLE:                  │
│  • Direct & grounding (85%)             │
│  • Uses metaphors frequently            │
│  • Opens with: "Let's get real..."      │
│                                         │
│  TOP TECHNIQUES (by effectiveness):     │
│  1. Reframe (90% effective)             │
│  2. Pause technique (88%)               │
│  3. Analogy (85%)                       │
│                                         │
│  SIGNATURE PHRASES:                     │
│  • "Before we burn bridges"             │
│  • "Let's get real for a moment"        │
│  • "What would [athlete] do?"           │
│                                         │
│  AI CONFIDENCE TREND:                   │
│  3 months ago: ████████░░ 62%           │
│  Today:       ██████████ 84% 📈         │
│                                         │
│  RECENT LEARNINGS:                      │
│  • Clients respond to sports analogies  │
│  • "Stop. Breathe." effective in crisis │
│                                         │
└─────────────────────────────────────────┘
```

---

## Database Schema

```sql
-- Human interventions with full analysis
CREATE TABLE human_intervention_analyses (
  id UUID PRIMARY KEY,
  message_id UUID REFERENCES messages(id),
  coach_id UUID REFERENCES coaches(id),
  client_id UUID REFERENCES clients(id),
  
  -- Style extraction
  style_tone VARCHAR(50),
  style_pace VARCHAR(20),
  style_formality VARCHAR(20),
  style_energy VARCHAR(20),
  
  -- Technique
  technique_type VARCHAR(50),
  framework_used VARCHAR(50),
  approach_description TEXT,
  key_phrases TEXT[],
  
  -- Outcome
  client_reaction VARCHAR(20),
  resolution_achieved BOOLEAN,
  conversation_shift TEXT,
  
  -- Learning flags
  applicable_to_similar BOOLEAN,
  transferable_to_others BOOLEAN,
  confidence_score FLOAT,
  replication_value FLOAT,
  
  created_at TIMESTAMP DEFAULT NOW()
);

-- Client-specific learning
CREATE TABLE client_effective_approaches (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  situation_type VARCHAR(100),
  approach JSONB,
  effectiveness_score FLOAT,
  times_validated INT DEFAULT 1,
  source_intervention_id UUID,
  examples TEXT[],
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  
  UNIQUE(client_id, situation_type)
);

-- Coach style profiles (continuously updated)
CREATE TABLE coach_style_profiles (
  id UUID PRIMARY KEY,
  coach_id UUID REFERENCES coaches(id),
  
  style_signature JSONB,
  preferred_techniques JSONB,
  frameworks JSONB,
  situation_responses JSONB,
  signature_phrases JSONB,
  
  total_interventions_analyzed INT DEFAULT 0,
  ai_confidence_trend JSONB, -- [{date, score}, ...]
  last_updated TIMESTAMP DEFAULT NOW()
);

-- Universal patterns (across all coaches)
CREATE TABLE universal_coaching_patterns (
  id UUID PRIMARY KEY,
  
  situation_description TEXT,
  situation_keywords TEXT[],
  sentiment_range_min FLOAT,
  sentiment_range_max FLOAT,
  urgency_indicators TEXT[],
  
  effective_response JSONB,
  
  source_interventions UUID[],
  coach_count INT,
  client_count INT,
  success_rate FLOAT,
  
  statistical_significance BOOLEAN,
  confidence_score FLOAT,
  sample_size INT,
  
  times_used_by_ai INT DEFAULT 0,
  ai_success_rate FLOAT,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Summary

| Level | What | Updates When | Impact |
|-------|------|--------------|--------|
| **Client-specific** | "What works for Sarah" | Every human intervention | Immediate relevance |
| **Coach-specific** | "Akmal's style" | Every intervention | Authentic voice |
| **Universal** | "What works for everyone" | Pattern detected across coaches | Best practices |

**The Flywheel:**
```
Every human intervention → Analysis → 3-level learning → 
Better AI responses → Less human intervention needed → Scale
```

After 6 months:
- AI knows each client's triggers and effective responses
- AI coaches in the authentic style of the human coach
- AI applies proven techniques across similar situations
- Human intervention needed only for true edge cases

**This is the moat:** Not just AI coaching, but AI that **learns and improves** from every human touch.

---

*See also: [Conversation Memory](conversation-memory.md) - 4-layer memory system*
