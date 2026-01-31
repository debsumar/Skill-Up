---
title: Agile & Scrum
date: 2026-01-31
tags:
  - agile
  - scrum
  - interview
---

# Agile & Scrum

> [!important] JD Requirement
> "Work effectively in an Agile/Scrum environment, participating in sprint planning, estimations, standups, and retrospectives."

## Scrum Framework

```
┌─────────────────────────────────────────────────────────────┐
│                      SPRINT (2-4 weeks)                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │ Planning │──▶│  Daily   │──▶│  Review  │──▶│  Retro   │ │
│  └──────────┘   │ Standups │   └──────────┘   └──────────┘ │
│                 └──────────┘                                │
│                                                             │
│  Product    Sprint                          Increment       │
│  Backlog ──▶ Backlog ──────────────────────▶ (Shippable)   │
└─────────────────────────────────────────────────────────────┘
```

## Scrum Roles

| Role | Responsibility |
|------|----------------|
| **Product Owner** | Defines features, prioritizes backlog, accepts work |
| **Scrum Master** | Facilitates process, removes blockers |
| **Development Team** | Self-organizing, delivers increment |

## Scrum Ceremonies

### Sprint Planning

```
Duration: 2-4 hours for 2-week sprint

Inputs:
├── Product Backlog (prioritized)
├── Team velocity
└── Sprint goal

Outputs:
├── Sprint Backlog
├── Sprint Goal
└── Commitment
```

**What happens:**
1. PO presents top backlog items
2. Team discusses and clarifies requirements
3. Team estimates and commits to items
4. Break items into tasks

### Daily Standup

```
Duration: 15 minutes MAX
Time: Same time daily
Format: Standing up (keeps it short)

Each person answers:
┌─────────────────────────────────┐
│ 1. What did I do yesterday?     │
│ 2. What will I do today?        │
│ 3. Any blockers?                │
└─────────────────────────────────┘
```

> [!tip] Best Practices
> - Start on time, don't wait
> - Keep it brief - details offline
> - Focus on sprint goal progress
> - Raise blockers, don't solve them here

### Sprint Review (Demo)

```
Duration: 1-2 hours
Attendees: Team + Stakeholders

Agenda:
1. Demo completed work
2. Gather feedback
3. Discuss what's next
4. Update product backlog
```

### Sprint Retrospective

```
Duration: 1-1.5 hours
Attendees: Team only (safe space)

Common Format:
┌─────────────────────────────────────────┐
│  What went well?  │  What to improve?   │
├───────────────────┼─────────────────────┤
│  - CI/CD setup    │  - Code reviews slow│
│  - Team collab    │  - Unclear specs    │
└───────────────────┴─────────────────────┘
           │
           ▼
    Action Items (pick 1-2)
```

## Estimation Techniques

### Story Points

```
Fibonacci: 1, 2, 3, 5, 8, 13, 21

1 point  = Trivial (change a label)
2 points = Simple (add a button)
3 points = Small feature
5 points = Medium feature
8 points = Large feature
13 points = Very large (consider splitting)
21 points = Epic (must split)
```

### Planning Poker

1. PO describes story
2. Team discusses
3. Everyone reveals estimate simultaneously
4. Discuss outliers
5. Re-vote if needed
6. Consensus reached

### T-Shirt Sizing

| Size | Effort |
|------|--------|
| XS | Hours |
| S | 1-2 days |
| M | 3-5 days |
| L | 1-2 weeks |
| XL | Too big, split it |

## Velocity

```
Sprint 1: 20 points completed
Sprint 2: 25 points completed
Sprint 3: 22 points completed
─────────────────────────────
Average Velocity: ~22 points/sprint

Use for:
- Sprint planning capacity
- Release forecasting
- Identifying trends
```

## User Stories

### Format

```
As a [user type]
I want [feature]
So that [benefit]

Example:
As a mobile user
I want to login with biometrics
So that I can access the app quickly
```

### Acceptance Criteria

```
Given [context]
When [action]
Then [expected result]

Example:
Given I have biometrics enabled
When I tap the fingerprint icon
Then I should be logged in within 2 seconds
```

### INVEST Criteria

| Letter | Meaning |
|--------|---------|
| I | Independent |
| N | Negotiable |
| V | Valuable |
| E | Estimable |
| S | Small |
| T | Testable |

## Kanban Board

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ Backlog  │   To Do  │   In     │  Review  │   Done   │
│          │          │ Progress │          │          │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ Story 5  │ Story 3  │ Story 2  │ Story 1  │ Story 0  │
│ Story 6  │ Story 4  │          │          │          │
│ Story 7  │          │          │          │          │
└──────────┴──────────┴──────────┴──────────┴──────────┘
                         WIP Limit: 2
```

## Questions & Answers

> [!question]- Q1: What is the difference between Scrum and Kanban?
> **Answer:**
> | Scrum | Kanban |
> |-------|--------|
> | Fixed sprints | Continuous flow |
> | Defined roles | No prescribed roles |
> | Sprint backlog locked | Can add items anytime |
> | Velocity-based | WIP limits |
> 
> Scrum is better for planned releases, Kanban for continuous delivery.

> [!question]- Q2: How do you handle a story that's not completed in a sprint?
> **Answer:**
> Options:
> 1. Move incomplete story to next sprint
> 2. Split into smaller stories (completed vs remaining)
> 3. Discuss in retrospective why it wasn't completed
> 
> Never mark incomplete work as done. Analyze why - was it too big? Blocked?

> [!question]- Q3: What do you do when blocked?
> **Answer:**
> 1. Raise it immediately in standup
> 2. Notify Scrum Master
> 3. Document the blocker
> 4. Work on other tasks while waiting
> 5. Escalate if not resolved quickly
> 
> Don't wait until standup if it's urgent.

> [!question]- Q4: How do you estimate a story you've never done before?
> **Answer:**
> - Compare to similar past stories
> - Break down into smaller tasks
> - Add buffer for unknowns
> - Discuss with team (collective wisdom)
> - Use spike (time-boxed research) if too uncertain
> 
> It's okay to say "we need more info."

> [!question]- Q5: What makes a good sprint goal?
> **Answer:**
> Good sprint goal:
> - Single, focused objective
> - Provides direction for decisions
> - Measurable
> - Achievable within sprint
> 
> Example: "Users can complete checkout flow" not "Fix bugs and add features"

> [!question]- Q6: How do you handle scope creep during a sprint?
> **Answer:**
> - Sprint backlog is protected once committed
> - New requests go to product backlog
> - PO prioritizes for next sprint
> - Only exception: critical production issues
> 
> If scope must change, something else must be removed.

> [!question]- Q7: What's the purpose of a retrospective?
> **Answer:**
> Continuous improvement:
> - Reflect on what worked/didn't work
> - Identify actionable improvements
> - Build team trust and communication
> - Celebrate successes
> 
> Key: Actually implement the action items!

> [!question]- Q8: How do you handle technical debt in Scrum?
> **Answer:**
> Options:
> - Allocate % of sprint capacity (e.g., 20%)
> - Create technical debt stories in backlog
> - Include refactoring in Definition of Done
> - Dedicated tech debt sprints (occasionally)
> 
> Make it visible to PO with business impact.

> [!question]- Q9: What is Definition of Done (DoD)?
> **Answer:**
> Checklist that defines when work is complete:
> - Code reviewed
> - Unit tests passing
> - Integration tests passing
> - Documentation updated
> - Deployed to staging
> - PO accepted
> 
> Team agrees on DoD, applies to all stories.

> [!question]- Q10: How do you handle disagreements about estimates?
> **Answer:**
> In Planning Poker:
> 1. Highest and lowest explain their reasoning
> 2. Team discusses assumptions
> 3. Re-vote
> 4. If still different, go with higher estimate
> 
> Goal is shared understanding, not exact numbers.
