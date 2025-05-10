

# MathGPT Take home assignment: Agentic Tutoring Flow Design

## Executive Summary

Designed an agent-based personalized tutoring system that adapts dynamically to each student's learning needs. It uses a Receptionist Agent to identify the student’s intent, a Subject Planner Agent to craft individualized learning plans using prior progress, preference of student and knowledge graphs, and a Negotiation Loop that adapts these plans based on feedback. This modular architecture supports adaptive learning flows while maintaining pedagogical integrity. The following is inspired from [this paper](https://arxiv.org/abs/2410.10650)

---

## 1. Key Assumptions and Problem Formulation

Modern educational tools often fail to personalize learning in a meaningful, adaptive way. Static content delivery does not address individual learner struggles, progress, or evolving goals.

The goal was to build a multi-agent tutoring system that tailors instructional content in real-time based on a student’s:
- Prior performance  
- Current mastery levels   
- Feedback on the learning plan  

**Key assumptions:**
- Each student has a structured knowledge graph per subject (e.g., Algebra → quadratic_equations). This is stored in the form of a json (dict). 
- Each subject has a progress % (content covered) and mastery score (accuracy-based). There is a content_access list of different subtopics in a subject. 
- Student intent can be inferred from queries (e.g., "I want to work more on Algebra").
- The proposed plan will only have modules from content_access. Each module in content_access can have multiple activity types.
- Student will only input a topic/subject which is part of the knowledge graph.   
- The negotiations respect the defined pedagogy and scaffolded learning flow.  

A knowledge graph and content_access list looks for a student like this:
```
profile = StudentProfile(
    name="Alice",
    content_access=[
        "Algebra_Basics", "Algebra_Quadratics", "Algebra_Polynomials",
        "Geometry_Basics", "Geometry_Triangles", "Geometry_Circles",
        "Calculus_Limits", "Calculus_Derivatives"
    ],
    subject_stats={
        "Algebra": {"progress": 75, "mastery": 0.68},
        "Geometry": {"progress": 85, "mastery": 0.80},
        "Calculus": {"progress": 40, "mastery": 0.55}
    },
    knowledge_graph={
        "Algebra": {
            "linear_equations": {"status": "mastered", "last_seen": "2025-04-10"},
            "quadratic_equations": {"status": "partial", "last_seen": "2025-04-25"},
            "polynomials": {"status": "unseen", "last_seen": None}
        },
        "Geometry": {
            "angles": {"status": "mastered", "last_seen": "2025-04-15"},
            "triangles": {"status": "partial", "last_seen": "2025-04-22"},
            "circles": {"status": "unseen", "last_seen": None}
        },
        "Calculus": {
            "limits": {"status": "partial", "last_seen": "2025-04-28"},
            "derivatives": {"status": "unseen", "last_seen": None},
            "integrals": {"status": "unseen", "last_seen": None}
        }
    },
    session_history=[
        {"date": "2025-04-28", "activity": "quiz", "topic": "limits", "score": "0.5"},
        {"date": "2025-04-27", "activity": "interactive_simulation", "topic": "limits"},
        {"date": "2025-04-26", "activity": "video", "topic": "triangles", "watched": "True"},
        {"date": "2025-04-25", "activity": "quiz", "topic": "quadratic_equations", "score": "0.6"},
        {"date": "2025-04-24", "activity": "video", "topic": "quadratic_equations", "watched": "True"},
        {"date": "2025-04-23", "activity": "group_discussion", "topic": "angles"},
        {"date": "2025-04-20", "activity": "quiz", "topic": "linear_equations", "score": "0.85"}
    ],
    preference={
        "video": False,
        "interactive": True,
        "quiz_difficulty": "medium",
        "time_limit": 10,
        "preferred_subject": "Algebra",
        "max_session_duration": 30,  # minutes
        "preferred_learning_style": "inquiry-based",
        "avoid_activities": ["lecture_notes", "long videos"]
    }
)

```
---

## 2. Proposed Approach

### 2.1 Architecture Overview

Implemented a modular architecture with 2 cooperating agents:

#### (i) Receptionist Agent  
Classifies the student’s query and selects intent and a suitable subject using subject stats. The output is a dict which is shown below:

> **Student input**:  
> `What would you like to study today (Algebra, Geometry, Calculus)? > revise geometry`

> **Receptionist Output**:
```json
{
  "intent": "review_request",
  "target_subject": "Geometry",
  "context": {
    "progress": 85,
    "mastery": 0.8
  }
}
```

## (ii) Subject Planner Agent
Creates a learning plan based on the student’s:

- Output from the receptionist agent (which has: intent, target_subject, progress, mastery)
- Prior session history
- Knowledge graph for the selected subject (each knowledge graph has submodules and the level of mastery achieved in it).
- Preference for certain activities ([This](https://web.mit.edu/5.95/readings/bloom-two-sigma.pdf) paper suggests that having personal preferences in mind makes the student more productive)
- Only selects modules from content_access


Sample Output:
```json
{
  "learning_objective": "Review and reinforce understanding of triangles and familiarize with circles in geometry.",
  "pedagogy": "Inquiry-based learning through interactive exploration and problem-solving activities.",
  "target_subject": "Geometry",
  "activity_sequence": [
    {
      "topic": "Geometry_Triangles",
      "activity_type": "interactive_simulation",
      "constraints": {
        "time_limit": 10,
        "session_duration": 30
      }
    },
    {
      "topic": "Geometry_Circles",
      "activity_type": "interactive_simulation",
      "constraints": {
        "time_limit": 10,
        "session_duration": 30
      }
    },
    {
      "topic": "Geometry_Triangles",
      "activity_type": "quiz",
      "constraints": {
        "quiz_difficulty": "medium",
        "time_limit": 10
      }
    }
  ]
}
```
## (iii) Negotiation Loop
 Handles student objections or requests and preserves pedagogical intent while modifying the plan. If a student wants to "skip the instructions," we can convert a video into notes but retain the concept. The negotiation is sent back to the subjectplanner agent and new plan is suggested. 
 
Sample output:
`Any feedback on this plan? (Leave blank to accept) > no quiz`
Updated Plan after Negotiation:
```json
{
  "learning_objective": "Review and reinforce understanding of triangles and familiarize with circles in geometry.",
  "pedagogy": "Inquiry-based learning through interactive exploration and problem-solving activities.",
  "activity_sequence": [
    {
      "topic": "Geometry_Triangles",
      "activity_type": "interactive_simulation",
      "constraints": {
        "time_limit": 10,
        "session_duration": 30
      }
    },
    {
      "topic": "Geometry_Circles",
      "activity_type": "interactive_simulation",
      "constraints": {
        "time_limit": 10,
        "session_duration": 30
      }
    },
    {
      "topic": "Geometry_Triangles",
      "activity_type": "group_discussion",
      "constraints": {
        "duration": 10
      }
    }
  ]
}
```

## Data Storage and tradeoffs
The receptionist agent and subjectplanner agent interact via structured json messages. All the agents in this context use the GPT-4o-mini model in the backend. Below is the tradeoff table:


| **Design Decision**               | **Trade-off**                                                             | **Justification**                                                                                  |
|----------------------------------|---------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Stateless agents                 | Requires external state management (profile/context must be passed each time) | Simplifies agent deployment and horizontal scaling                                                  |
| Constraint-based content filtering | May exclude helpful off-path material                                      | Honors curriculum scope, licensing, and ensures targeted content delivery                           |
| Structured JSON protocol         | Reduces expressive flexibility                                             | Enforces reliable agent coordination, validation, and debugging                                     |
| Time-limited activities          | May interrupt flow for slower learners                                     | Keeps sessions manageable and predictable for tutoring environments                                 |


## 4. Next Steps
If I had more time, I would:
- Expand content coverage: Add more subjects and granular topic breakdowns. Also have one more component of different modules to integrate.
- Tackle issues with scaling: Tackle challenges that would occur while scaling it to multiple students.
- Incorporate retrieval-based grounding: Fetch relevant curriculum examples, not rely entirely on generation. This can also reduce hallucinations and have more factually-grounded outputs.
- Log plan effectiveness: Track which activities result in improved mastery. Calculate this over a batch of students to have a better fine-grained analysis.
- Add a UI: For students to interactively accept/decline or modify each activity.
- Introduce memory: Track long-term learning patterns across sessions. We can track student behaviour with this which can help us understand more about potential behaviour of a student.

**Notebook with sample code and system diagram are in this repository**

*Note: Used LLMs to paraphrase the above content in a better way*
