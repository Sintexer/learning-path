## Core Pillars of Team Management

1. **Leadership styles**: Understand different leadership styles—such as autocratic, democratic, transformational, and servant leadership—and when each applies (e.g., directive during a Sev-1 incident vs servant/coaching during normal product delivery).
2. **Communication skills**: Effective communication is crucial in team management. Practice clear and concise communication, active listening, and delivering timely, constructive feedback (see also [[Code review best practices and pitfals]]).
3. **Conflict resolution**: Manage and resolve disagreements constructively using negotiation, mediation, and aligning on shared goals/data rather than personal preferences.
4. **Team building**: Foster team cohesion, morale, and **psychological safety**—including **blameless postmortems** that fix systemic failure modes instead of blaming individuals.
5. **Delegation**: Delegate tasks and ownership based on strengths and growth goals—delegate *outcomes and guardrails*, not step-by-step instructions—while remaining available for support.
6. **Performance management**: Set clear goals and expectations, give regular (no-surprise) feedback, and address underperformance or blockers early.
7. **Time management**: Prioritize high-leverage work, protect deep-focus time, set realistic deadlines, and manage scope explicitly.
8. **Emotional intelligence**: Build self-awareness, self-regulation, empathy, and social skills to read team dynamics and defuse tension.
9. **Motivation**: Inspire and empower engineers through autonomy, mastery, purpose, and visible recognition.
10. **Continuous learning**: Embrace a growth mindset and invest in ongoing technical and leadership development.

## Staff+ Engineering Leadership Lens

- **Influence without authority**: Staff engineers drive alignment across teams through crisp written RFCs/ADRs, prototypes, and shared trade-off analysis rather than positional authority.
- **Conway's Law**: Organizations design systems that mirror their communication structures—deliberately align team ownership boundaries with service boundaries ([[Microservices decomposition]], [[DDD]]).

> [!tip] Production reality
> **Micromanagement vs abdication**: Telling senior engineers *how* to implement every detail kills ownership and makes you a bottleneck; conversely, handing off a vague goal with zero context ("abdication") leads to rework. Align clearly on the **why**, **success criteria**, and **constraints/guardrails**, then let the owner drive the **how** with lightweight checkpoints.

Effective delegation transfers ==outcomes, constraints, and decision ownership== rather than step-by-step instructions—avoiding both micromanagement and vague abdication.

What is the goal of a blameless postmortem?::To surface systemic, process, and architectural root causes behind an incident—assuming engineers acted with good intent given the information they had—so the organization fixes systems rather than hiding mistakes out of fear.

## How does Conway's Law affect engineering management and system architecture? #card

Conway's Law states that organizations design systems whose structure mirrors their internal communication structures. If two teams share ownership of one database or require constant cross-team syncs to ship a feature, the architecture becomes tightly coupled—so aligning team boundaries with bounded contexts ([[DDD]], [[Microservices decomposition]]) is both an organizational and architectural lever.

## Related

- [[Code review best practices and pitfals]]
- [[Microservices decomposition]]
- [[DDD]]
- [[Reliability, Scalability, Maintainability]]
