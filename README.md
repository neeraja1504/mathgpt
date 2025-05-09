# mathgpt

# Option B – Agentic Tutoring Flow Design

## Executive Summary

We designed an agent-based personalized tutoring system that adapts dynamically to each student's learning needs. It uses a Receptionist Agent to identify the student’s intent, a Subject Planner Agent to craft individualized learning plans using prior progress and knowledge graphs, and a Negotiation Loop that adapts these plans based on feedback. This modular architecture supports adaptive learning flows while maintaining pedagogical integrity.

---

## 1. Key Assumptions

Modern educational tools often fail to personalize learning in a meaningful, adaptive way. Static content delivery does not address individual learner struggles, progress, or evolving goals.

Our goal was to build a multi-agent tutoring system that tailors instructional content in real-time based on a student’s:
- Prior performance  
- Current mastery levels  
- Self-declared interests  
- Feedback on the learning plan  

**Key assumptions:**
- Students have a structured knowledge graph per subject (e.g., Algebra → quadratic_equations).  
- Each subject has a progress % (content covered) and mastery score (accuracy-based).  
- Student intent can be inferred from queries (e.g., "I want to work more on Algebra").  
- Plans can be negotiated using conversational feedback loops.  
- The negotiations respect the defined pedagogy and scaffolded learning flow.  

---

## 2. Proposed Approach

### 2.1 Architecture Overview

We implemented a modular architecture with 2 cooperating agents:

#### (i) Receptionist Agent  
Classifies the student’s query and selects intent and a suitable subject using subject stats.

> **Student input**:  
> `What would you like to study today? > revise geometry`

> **Receptionist Output**:
```json
{
  "intent": "review_request",
  "target_subject": "Geometry",
  "context": {
    "progress": 90,
    "mastery": 0.88
  }
}
```

## (ii) Subject Planner Agent
Creates a learning plan based on the student’s:
- Mastery level,
- Prior session history, and
- Knowledge graph for the selected subject.


Sample Output:
```json
{
  "learning_objective": "Review and consolidate understanding of triangles and introduce circles in Geometry.",
  "pedagogy": "Adaptive review through targeted quizzes and interactive learning.",
  "activity_sequence": [
    {
      "activity": "quiz",
      "topic": "triangles",
      "difficulty": "medium",
      "time_limit": 10
    },
    {
      "activity": "interactive_game",
      "topic": "triangles"
    },
    {
      "activity": "quiz",
      "topic": "circles",
      "difficulty": "medium",
      "time_limit": 10
    },
    {
      "activity": "interactive_simulation",
      "topic": "circles"
    }
  ]
}
```
## (iii) Negotiation Loop
 Handles student objections or requests and preserves pedagogical intent while modifying the plan. If a student wants to "skip the instructions," we might convert a video into notes but retain the concept. The negotiation is sent back to the subjectplanner agent and new plan is suggested. 
Any feedback on this plan? (Leave blank to accept) > no interactive game
🔁 Updated Plan after Negotiation:
```json
{
  "learning_objective": "Review and consolidate understanding of triangles and introduce circles in Geometry.",
  "pedagogy": "Adaptive review through targeted quizzes and interactive learning.",
  "activity_sequence": [
    {
      "activity": "quiz",
      "topic": "triangles",
      "difficulty": "medium",
      "time_limit": 10
    },{
      "activity": "group_discussion",
      "topic": "triangles"
    },{
      "activity": "quiz",
      "topic": "circles",
      "difficulty": "medium",
      "time_limit": 10
    },{
      "activity": "interactive_simulation",
      "topic": "circles"
    }  ] }
```

2.2 Data Flow and Protocol
Student Query --> Receptionist --> Subject Planner --> Plan --> Student Feedback --> Negotiation Loop --> Updated Plan

All components interact via structured JSON messages.
2.3 Code Overview
call_openai(): Handles LLM interaction


receptionist_agent(): Detects intent and recommends a subject


subject_planner_agent(): Designs pedagogical plan


We used GPT-4o-mini as the backend model due to its support for low-latency and instruction-following behavior.

## tradeoffs
| **Decision**                | **Trade-off**                          | **Justification**                                                                 |
|----------------------------|----------------------------------------|-----------------------------------------------------------------------------------|
| Agent-based modularity     | Slight overhead due to message passing | Allows clean responsibility separation, easy scaling, and debugging              |
| LLM-powered planning       | Cost of API calls                      | Rich domain-specific generation and adaptation                                   |
| Structured JSON I/O        | Less flexible than freeform text       | Ensures safe plan parsing and validation                                         |
| Fallbacks in Negotiation   | Might oversimplify student objections  | We preserve scaffolding by substituting rather than removing core activities     |


## 4. Next Steps
If we had more time, we would:
- Expand content coverage: Add more subjects and granular topic breakdowns.
- Incorporate retrieval-based grounding: Fetch relevant curriculum examples, not rely entirely on generation.
- Log plan effectiveness: Track which activities result in improved mastery.
- Stick to certain plans only
- Add a UI: For students to interactively accept/decline or modify each activity.
- Introduce memory: Track long-term learning patterns across sessions.



