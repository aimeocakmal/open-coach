# ⚠️ Risks & Mitigations

## Overview

Every startup faces risks. The key is identifying them early and having mitigation plans. This document tracks the major risks for OpenCoach and our strategies to address them.

---

## Risk Matrix

| Risk | Likelihood | Impact | Priority |
|------|------------|--------|----------|
| WhatsApp API access denied | Medium | Critical | 🔴 High |
| AI quality doesn't meet expectations | Medium | High | 🔴 High |
| Low coach adoption | Medium | Critical | 🔴 High |
| Competition from well-funded players | High | Medium | 🟡 Medium |
| AI API cost volatility | Low | Medium | 🟡 Medium |
| Data privacy regulations | Medium | High | 🟡 Medium |
| Platform dependence on Supabase/Vercel | Low | Medium | 🟢 Low |
| Technical debt from rapid building | High | Medium | 🟢 Low |

---

## Critical Risks (🔴)

### 1. WhatsApp Business API Access Denied

**Risk:** Meta rejects our application for WhatsApp Business API access, blocking a key channel.

**Likelihood:** Medium  
**Impact:** Critical — WhatsApp is core to adoption strategy

**Root Causes:**
- Meta's approval process is opaque
- "Coaching" may not be an approved use case
- New/unproven company = higher scrutiny

**Mitigation Strategies:**

1. **Apply Early**
   - Submit application in Month 2 (before built)
   - Use established company (OKR Institute) if possible
   - Prepare thorough business case

2. **Alternative Channel First**
   - Build Telegram integration first (approved immediately)
   - Prove product-market fit without WhatsApp
   - Use traction to strengthen WhatsApp application

3. **B2B Angle**
   - Position as "employee coaching platform" not "consumer coaching"
   - Enterprise use case = higher approval chance
   - Partner with established enterprise software company

4. **Fallback Position**
   - Web + Telegram is still viable product
   - WhatsApp is enhancer, not core dependency
   - Can succeed without it (just slower growth)

**Contingency Plan:**
> If WhatsApp denied after 6 months, pivot to "enterprise messaging platform" positioning and reapply.

---

### 2. AI Quality Doesn't Meet Expectations

**Risk:** The AI coaching responses feel robotic, generic, or unhelpful. Coaches abandon platform.

**Likelihood:** Medium  
**Impact:** High — core value proposition fails

**Root Causes:**
- Prompt engineering not sophisticated enough
- Context retrieval (RAG) not working well
- Hallucinations or incorrect advice
- Tone doesn't match coach's style

**Mitigation Strategies:**

1. **Invest Heavily in Prompt Engineering**
   - Hire specialized prompt engineer
   - A/B test prompt variations
   - Build prompt versioning system
   - Collect feedback loops

2. **Start with Narrow Use Cases**
   - Don't try to replace all coaching
   - Focus on specific workflows (goal setting, check-ins)
   - Expand scope as AI improves

3. **Human-in-the-Loop Feedback**
   - Coaches rate AI responses
   - Use ratings to fine-tune
   - Continuous improvement cycle

4. **Conservative Defaults**
   - Lower confidence threshold for handoff
   - Better to escalate to human than give bad advice
   - "I'd rather involve [Coach] on this"

5. **Coach Training**
   - Teach coaches how to train their AI
   - Best practices for knowledge base
   - Expectation setting (AI is assistant, not replacement)

**Success Metrics:**
- Coach satisfaction with AI responses > 4.0/5.0
- < 20% of AI responses require correction
- Client NPS for AI-coached sessions > 30

---

### 3. Low Coach Adoption

**Risk:** Coaches don't sign up or churn quickly. Can't reach critical mass.

**Likelihood:** Medium  
**Impact:** Critical — business fails without users

**Root Causes:**
- Coaches fear AI will replace them
- Product too complex to set up
- Price too high for value delivered
- Don't understand the value proposition

**Mitigation Strategies:**

1. **Position as "Amplifier" Not "Replacement"**
   - Marketing: "Scale yourself, not replace yourself"
   - Emphasize human handoff features
   - Case studies: "Coach X increased revenue 3x with AI"

2. **Frictionless Onboarding**
   - 5-minute setup promise
   - Pre-built templates for common coaching styles
   - AI-assisted setup (AI helps configure itself)
   - White-glove onboarding for first 50 coaches

3. **Lead with Free Tier**
   - Free forever for 1 client
   - Coaches experience value before paying
   - Natural upgrade path

4. **Leverage OKR Institute Network**
   - Beta test with existing coach network
   - Testimonials from trusted coaches
   - Referral incentives

5. **Show, Don't Tell**
   - Interactive demo (try AI coaching yourself)
   - ROI calculator
   - Time savings tracker

6. **Community Building**
   - "AI-Enabled Coaches" community
   - Share best practices
   - Peer support
   - Certification program

**Success Metrics:**
- Activation rate (complete setup): > 60%
- 30-day retention: > 50%
- Organic referrals: > 20% of new signups

---

## Medium Risks (🟡)

### 4. Competition from Well-Funded Players

**Risk:** BetterUp, CoachHub, or new unicorn startup launches competing product.

**Likelihood:** High  
**Impact:** Medium — splits market, drives up CAC

**Mitigation:**
- **Speed to market:** Launch fast, establish brand
- **Niche focus:** Own "individual coaches" segment
- **Network effects:** More coaches = more value
- **Data moat:** Memory/insights become switching cost
- **Community:** Hard to replicate relationships

**Contingency:**
- If BetterUp goes downmarket, consider partnership/acquisition
- Focus on features they can't easily copy (BYOK, WhatsApp)

---

### 5. AI API Cost Volatility

**Risk:** OpenAI/Anthropic raise prices significantly. BYOK model becomes unattractive.

**Likelihood:** Low  
**Impact:** Medium — affects unit economics

**Mitigation:**
- **Multi-provider support:** Not locked to one AI provider
- **Optimization:** Efficient prompts, caching, context compression
- **Pricing flexibility:** Can adjust if needed (value-based, not cost-plus)
- **Self-hosting option:** Eventually offer fine-tuned open-source models

---

### 6. Data Privacy Regulations

**Risk:** GDPR, PDPA, or new regulations require expensive compliance changes.

**Likelihood:** Medium  
**Impact:** High — legal/compliance burden

**Mitigation:**
- **Privacy by design:** Build compliant from start
- **Data minimization:** Only collect what's necessary
- **Regional hosting:** EU data in EU, etc.
- **DPO consultant:** Engage privacy expert early
- **Clear policies:** Transparent data usage terms

**Key Requirements:**
- Right to deletion
- Data portability
- Consent management
- Breach notification procedures

---

## Low Risks (🟢)

### 7. Platform Dependence (Supabase/Vercel)

**Risk:** Vendor lock-in with Supabase or Vercel. Price hikes or service issues.

**Likelihood:** Low  
**Impact:** Medium

**Mitigation:**
- Supabase is open source (can self-host)
- Standard PostgreSQL (easy migration)
- Next.js works anywhere (Vercel optional)
- Multi-cloud strategy by Phase 3

---

### 8. Technical Debt

**Risk:** Rapid building creates unmaintainable code. Slows future development.

**Likelihood:** High  
**Impact:** Medium

**Mitigation:**
- **Refactoring sprints:** Every 4th week dedicated to cleanup
- **Code reviews:** Even for solo founder, use PRs
- **Documentation:** This project is a good start!
- **Testing:** Unit tests for critical paths (AI, handoff)
- **Monitoring:** Error tracking from day one

---

## Risk Monitoring

### Monthly Risk Review

| Question | Action if "Yes" |
|----------|-----------------|
| Any critical risks materialized? | Emergency response plan |
| Any medium risks elevated? | Develop mitigation plan |
| Any new risks identified? | Add to register |
| Are mitigations working? | Adjust strategies |

### Key Risk Indicators (KRIs)

| KRI | Threshold | Response |
|-----|-----------|----------|
| Coach churn rate | > 10%/month | Investigate, interview churned coaches |
| AI confidence scores | < 0.7 avg | Review prompts, escalate more |
| Support ticket volume | > 5/coach/month | Product usability issues |
| WhatsApp API status | Rejected | Activate contingency plan |

---

## Contingency Plans Summary

| If This Happens | Do This |
|-----------------|---------|
| WhatsApp denied | Double down on Telegram + Web; reposition as B2B |
| AI quality poor | Reduce scope; more human handoff; hire prompt engineer |
| Low adoption | Free tier expansion; OKR Institute push; referral bonuses |
| BetterUp competition | Niche down further; emphasize BYOK/price advantage |
| AI costs spike | Add efficiency features; multi-provider; optimize prompts |
| Regulation changes | Engage legal; build compliance features; regional strategy |

---

## Final Thought

> "The biggest risk is not taking any risk. In a world that's changing quickly, the only strategy that is guaranteed to fail is not taking risks." — Mark Zuckerberg

We've identified the risks. Now let's build.

---

*Next: [Roadmap](roadmap.md)*
