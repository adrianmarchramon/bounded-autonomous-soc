# Phase 5 — Action: Bounded Response and Human-in-the-Loop

> **This is the phase that serves as the core of security, and perhaps the most important in the entire project.** In Phase 4, the decision *proposed* a response; in this phase, the agent *acts*, but in a way that represents the heart of trustworthy autonomy: **bounded** (limited and, where possible, reversible actions) and with a **human in the loop** (high-consequence actions require human approval). The governing concept of this phase, which materializes the "bounded" property of your approach, is that security lives *here*, in this reliable and deterministic layer, *not* in the decision (which is RL-based and can make mistakes) nor in any part that can fail. In this project, it is the exact equivalent of your robot's "safety-guaranteed navigation": no matter what the agent decides, this layer guarantees that it will never execute anything harmful or irreversible on its own. It is what separates a trustworthy agent from one that is a hazard.

**Objective of the phase:** For the agent to act, but with bounded and supervised autonomy.  
**Duration:** Weeks 7-8.  
**By the end, you will have:** An action layer that executes responses within safe limits, escalating high-consequence actions to a human.

---

## The Big Picture

The flow of this phase executes the response, but puts safety first:

```
   Action proposed by the decision (Phase 4)
        │
        ▼
   [ Is it low-risk and reversible? ]
        │  yes                             │  no (high-risk / irreversible)
        ▼                                  ▼
   [ The agent executes it (autonomous) ] [ HUMAN IN THE LOOP: proposes and waits for approval ]
        │                                  │  (approved)        │  (rejected)
        ▼                                  ▼                    ▼
   [ Execute in the environment ]         [ Execute ]          [ Do not execute ]
        │
        ▼
   [ GUARANTEE: never anything harmful or irreversible autonomously ]
```

---

## Step 1 — The Action Layer Owns Security

It is best to start by understanding the role of this layer, as this is where the maturity of the project is concentrated. The decision in Phase 4 is intelligent, but it relies on RL, and like any model, it can make mistakes—especially under the fog of war encountered in Phases 1 and 2. If you allowed every decision to turn directly, without filtering, into an action on real machines, a single error (or manipulation of the agent) could bring down a critical service, block legitimate users, or cause irreversible damage. That is why the responsibility to *act safely* cannot fall on the decision: it falls on this layer, which is classical, deterministic, and supervised, and whose sole mission is to ensure that no dangerous action is executed without oversight.

This is the underlying reason for the layered architecture of your approach, and the exact equivalent of the principle you already applied to your robot ("navigation guarantees safety, not the LLM"). There, the navigation prevented the robot from colliding even if the LLM directed it toward a wall; here, the action layer prevents the agent from causing harm even if the decision proposes it. Safety is not *expected* from a component that can fail; it is *guaranteed* in a layer designed for that very purpose. Building this layer correctly is, quite literally, what makes the entire system trustworthy.

---

## Step 2 — Bounded Actions: Limited and Reversible

The first safety mechanism is *bounding* what the agent can do on its own, which requires recognizing that not all actions are equal. Some are low-risk and reversible (monitoring a host, analyzing processes, deploying a decoy): if the agent makes a mistake, it is not a major issue, and it can be undone. Others are high-risk, disruptive, or irreversible (restoring a critical server, isolating a network segment, blocking traffic): an error there has severe consequences. The action layer classifies actions by this criterion and *bounds* the agent's autonomy accordingly. Create `src/action/execute.py`:

```python
# Classify actions by risk and reversibility
AUTONOMOUS_ACTIONS = {"Monitor", "Analyse", "Decoy"}              # low-risk, reversible
HIGH_RISK_ACTIONS = {"Restore", "Isolate", "BlockTraffic"}        # disruptive / critical


def execute_action(action, target, human_approval=False):
    if action in AUTONOMOUS_ACTIONS:
        return _execute_in_environment(action, target)            # the agent can perform it alone
    if action in HIGH_RISK_ACTIONS:
        if not human_approval:
            return _escalate_to_human(action, target)             # proposes; does NOT execute (see Step 3)
        return _execute_in_environment(action, target)            # only with approval
    # FAIL-SAFE: unclassified actions are treated as high-risk (when in doubt, escalate)
    return _escalate_to_human(action, target)
```

Notice two details that demonstrate sound judgment. First, the *risk classification*: the agent acts alone only on what is safe and reversible, and stops itself on critical tasks. Second, and highly important, the *fail-safe* at the end: any action that is not explicitly classified as autonomous is treated as high-risk and escalated. This is the secure default posture (when in doubt, do not act alone), preventing any new or unexpected action from slipping through without supervision. Bounding autonomy in this manner is the direct materialization of the "bounded" property of your approach.

---

## Step 3 — Human in the Loop

For high-consequence actions, the second safety mechanism is the *human in the loop*: instead of executing them, the agent *proposes* them to a human, who then decides. The agent does the heavy lifting (perceiving, investigating, deciding on the best response) and presents the high-risk action alongside its justification; the human either approves or rejects it. Create the escalation logic:

```python
def _escalate_to_human(action, target):
    """The agent PROPOSES the high-risk action, along with its justification; the human decides."""
    return {
        "status": "pending_approval",
        "action": action, 
        "target": target,
        "justification": _justify(action, target),   # from the investigation + the decision
    }   # the agent does NOT execute it on its own; it waits for the human decision
```

This shows a mature understanding of how such an agent should operate, and reflects how the industry functions in reality: organizations do not go from zero to full autonomy overnight; rather, they progress through stages (from *observing* to *governing* to *enforcing*), almost always keeping a human in the loop for consequential actions. There is an important idea behind this: the agent *augments* the analyst, it does not replace them. It relieves them of overwhelming tasks (monitoring, triaging, investigating, proposing responses) and leaves the higher-stakes decisions to them, which is precisely where human judgment and accountability matter. An agent that escalates critical matters to a human is not less valuable because of it; it is *more* trustworthy, and fits naturally into how real organizations adopt this technology.

---

## Step 4 — The Guarantee: Never Perform Harmful or Irreversible Actions Autonomously

Here lies the heart of this phase, the guarantee that governs everything: the agent **never executes a harmful or irreversible action autonomously**. What is important is *how* this is guaranteed: not by trusting that the decision is correct (which can fail), but *structurally*, within this layer. Every action passes through the filter: if it is low-risk and reversible, it is executed; if it is high-risk, it is escalated to a human; and if it is unclassified, it is escalated by default. There is no path through which a dangerous action can reach the environment without passing through this control. Safety is a property of the *architecture*, not a hope regarding the model's behavior.

This is what distinguishes a mature agent from a reckless one, and it is the culmination of the "bounded" property of your approach. Remember why this matters so much: the agent has the power to interact with real systems, and the decision-making components (the RL and the LLMs from previous phases) can make mistakes or be manipulated. Entrusting safety to a fallible component would be like trusting that a robot will not crash because its LLM "surely" will not direct it into a wall: reckless. That is why safety lives in this reliable, bounded, and supervised layer, which *can* guarantee it, regardless of what happens upstream. Imagine if the decision, due to an error under the fog of war, proposed restoring (and disrupting) all critical servers at once: this layer simply would not execute it without a human approving it. That is the difference between an agent you can trust and one that is a hazard.

---

## Step 5 — Tests

The safety logic in this phase is the most important part of the entire project that you can (and must) test thoroughly. Complete `tests/test_action.py`:

```python
from src.action.execute import execute_action


def test_low_risk_actions_are_executed_autonomously():
    r = execute_action("Monitor", "Host4")
    assert r["status"] != "pending_approval"      # reversible: the agent can perform it alone


def test_high_risk_actions_are_escalated_and_not_executed():
    r = execute_action("Restore", "CriticalServer")    # without approval
    assert r["status"] == "pending_approval"      # KEY: proposes, does not execute
    assert "justification" in r                    # includes justification for the human


def test_without_approval_critical_actions_never_reach_environment():
    r = execute_action("Isolate", "ServerSubnet", human_approval=False)
    assert r["status"] == "pending_approval"      # never executed autonomously


def test_fail_safe_unclassified_actions_are_escalated():
    r = execute_action("UnknownNewAction", "Host1")
    assert r["status"] == "pending_approval"      # when in doubt, escalate (secure posture)
```

These tests verify the most critical aspects of the project: that reversible actions run on their own, that critical actions *escalate and do not execute* without approval, that nothing dangerous reaches the environment autonomously, and that unknown actions escalate by default (fail-safe). Passing these tests proves that the safety guarantee is real. Run them with `make test`.

---

## Verification: The "Definition of Done"

The phase is complete when the following are met:

- [ ] You have built the action executor, with actions classified by risk and reversibility.
- [ ] The agent autonomously executes only low-risk, reversible actions.
- [ ] High-consequence actions require human approval (the agent proposes them, it does not execute them).
- [ ] A fail-safe is in place: unclassified actions escalate by default.
- [ ] **The key test:** The agent acts within safe boundaries, escalating high-consequence actions to a human.

The key test is bounded and supervised autonomy: if the agent acts on its own only when it is safe, and escalates serious actions to a human (never executing them without approval), you have built the core of trust. What is essential is that this is guaranteed *by the architecture* (via the filter, escalation, and fail-safe), rather than relying on the correctness of the decision. Having safety be structural, with a human controlling high-stakes actions, is what turns a powerful agent into a trustworthy one. It is what will set your work apart: anyone can build an agent that acts; the difficult and mature part is building one that acts *with guaranteed safety*.

---

## Deliverables and Next Steps

By closing Phase 5, you have established the core of trust: an action layer that executes responses within safe limits—acting alone only on reversible actions, escalating high-consequence ones to a human, and using a fail-safe that escalates when in doubt. You have materialized the "bounded" property of your approach, put safety where it belongs (in a reliable layer, rather than a fallible decision component), and ensured the agent augments the analyst rather than replacing them. The agent now acts, but with trustworthy autonomy.

The next step, **Phase 6**, completes trust from another angle: **explainability and auditability**. You will make the agent *explain* every decision (why it detected, investigated, or responded in a certain way, reusing your XAI and the verbalized reasoning of the LLMs) and ensure that every action is recorded in an *immutable* log (reusing the traceability of your forensics) so that a human can understand and audit everything the agent does. If this phase guaranteed that the agent acts *safely*, the next one ensures that it acts *transcreetly and accountably*. You have progressed from "the agent acts safely" to being on the verge of "the agent acts in an explainable and accountable manner."
