# Phase 6 — Trust: Explainability and Auditing

> Phase 5 ensured the agent acted *safely* (bounded autonomy, human-in-the-loop). But trust has another side, and that is what you build here: having the agent act *transparently and accountably*. Because an agent acting on real machines, even if safely, is not fully trustworthy if you cannot *understand* why it did what it did, nor *review* what it did afterward. The guiding idea of this phase is to complete trust with two properties of your approach: **explainable** (the agent justifies every decision) and **auditable** (every action is logged immutably). And, like almost everything in this project, you reuse what you have built: the explainability of your XAI, and the traceability of your forensics. Together, the safety of Phase 5 and the transparency of this phase make the agent's autonomy, at last, trustworthy.

**Phase objective:** making every decision explainable and every action auditable.
**Duration:** weeks 8-9.
**Upon completion, you will have:** explainability (the agent justifies its decisions) and an immutable audit log of everything it does.

---

## The Big Picture

The flow of this phase wraps every agent decision in transparency and accountability:

```
   Every agent decision (detect, investigate, decide, act)
        │
        ├──► [ Explainability ]  ──►  why (SHAP + LLM verbalized reasoning)
        │       (reused from your XAI)        → a human can UNDERSTAND and VALIDAR
        │
        └──► [ Immutable Audit ] ──►  what it did, when, why (tamper-proof)
                (reused from your forensics)  → a human can REVIEW afterward
```

Add the explainability dependency (the same one from your XAI):

```bash
uv add shap
```

---

## Step 1 — The Other Side of Trust: Transparency and Accountability

It is worth understanding why this phase is as important as the previous one, because it completes trust. Phase 5 guaranteed that the agent does not cause *harm* (safety). But imagine an agent that is safe but remains a black box: it makes decisions you cannot understand, and acts without leaving a trace of what it did or why. Would you trust it to defend your systems? No, because you could not *validate* its decisions or *hold it accountable* for its actions. Trust is not just "it does no harm"; it is also "I understand what it does and I can review it."

That is why this phase builds the other two properties of trustworthy autonomy in your approach: **explainable** (you can understand *why* the agent decided something) and **auditable** (you can *review* afterward *what* it did). And the beauty of it is that you reuse two of your projects for this: your **XAI** taught you to explain a model's decisions (with the idea that "the explanation matters as much as the prediction"), and your **forensics** taught you traceability (that every conclusion must be traceable back to its evidence). Here you unite them to make the agent transparent and accountable. Without this, the safety in Phase 5 would be insufficient: a safe but opaque agent is not trustworthy.

---

## Step 2 — Explainability: Why the Agent Did What It Did

The first step is to make the agent *explain* its decisions, and here you combine two sources, reusing your XAI. The agent makes decisions across multiple layers (detect, investigate, decide response), and each one must be justifiable. Create `src/trust/explain.py`:

```python
def explain(state, perception, investigation, decision):
    """Gathers the justification of the agent's decision, end-to-end."""
    return {
        "why_i_detected": perception["threats"],          # the signals that triggered the alarm
        "what_i_understood": investigation["timeline"],    # the anchored reasoning (the LLM verbalizes it)
        "why_this_response": _decision_factors(state, decision),  # SHAP on the policy
    }
```

There are two complementary sources of explanation. On one hand, the **LLM's verbalized reasoning**: in Phase 3, the investigation already produced a timeline with its reasoning ("the attacker scanned, then exploited host X..."), so that explanation *already exists*—we just have to leverage it. On the other hand, **SHAP** (reused from your XAI): for models that do not speak (the anomaly detector, the RL policy), SHAP reveals which factors weighed on the decision ("Restore was chosen over Host4 because its compromise state was 'privileged' and it is a critical asset"). Together, they provide a clear picture of *why* the agent did what it did, in a language that a human understands. It is the same idea from your XAI: a decision without an explanation, in a serious domain, is not enough.

---

## Step 3 — Auditing: The Immutable Log

The second step is to record *everything* the agent does in an **immutable** audit log, and here you reuse the traceability of your forensics. Every decision, every executed action, every escalation to a human, and every approval is logged, along with its justification, in a way that cannot be altered. Create `src/trust/audit.py`:

```python
import hashlib, json, time

def log_event(log_path, event: dict, previous_hash: str) -> str:
    """Adds an event to the immutable audit log (hash-chained, tamper-proof)."""
    entry = {
        "timestamp": time.time(),
        "event": event,                     # what the agent did (decision, action, escalation...)
        "justification": event.get("why"),
        "previous_hash": previous_hash,     # chains with the previous entry
    }
    entry["hash"] = hashlib.sha256((json.dumps(entry, sort_keys=True)).encode()).hexdigest()
    with open(log_path, "a") as f:          # append-only: only added, never modified
        f.write(json.dumps(entry) + "\n")
    return entry["hash"]
```

What matters is *why* it must be immutable. An audit log serves three purposes: for *compliance* (being able to prove what the agent did and why), to *investigate incidents* (if something goes wrong, reconstructing exactly what the agent did, just like in a forensic investigation), and to ensure the log itself is *reliable* (so that no one—neither an attacker nor someone wanting to hide an error—can alter it). That is why it is designed as an append-only, hash-chained log, so that any tampering is detected. It is the traceability of your forensics, applied to the agent's actions: everything it does is registered so it can be reviewed and trusted.

---

## Step 4 — Why Trust Demands Explaining and Auditing

Here it is worth pausing to consider why *both* are needed, because it demonstrates that you understand what trust actually is. Explainability and auditing serve different moments. The **explanation** acts *in the moment*: it allows a human to *validate* the agent's decision as it occurs (Is this response justified? Does this reasoning make sense?). The **audit** acts *afterward*: it allows reviewing what the agent did a posteriori (What actions did it take during this incident? Were they correct?), in order to learn, hold it accountable, or investigate if something went wrong. One looks to the present; the other, to the past. Together, they make the agent *accountable* to humans, which is what trust demands: it is not enough for it to act well; we must be able to verify it, both in the moment and after the fact.

And there is a beautiful connection to Phase 5 that demonstrates how everything fits together. Remember how the agent *escalates* high-consequence actions to a human, who then approves or rejects them? Well, that human could not make an informed decision if they did not *understand* why the agent is proposing that action. In other words, the explainability of this phase is precisely what makes the human-in-the-loop from Phase 5 *actually work*: the human sees the proposed action *along with its justification* (the signals, the timeline, the factors), allowing them to give an informed approval rather than a blind one. Explainability is not a decoration: it is what makes human supervision possible. Safety, transparency, and accountability are not loose pieces; they support one another.

---

## Step 5 — Tests

Verify the most important aspects of this phase: the integrity of the audit log (that it is truly immutable) and that the explanation gathers the justifications. Complete `tests/test_trust.py`:

```python
from src.trust.audit import log_event, verify_integrity


def test_audit_log_detects_tampering():
    h0 = log_event("audit.log", {"event": "Restore Host4", "why": "compromised"}, previous_hash="")
    h1 = log_event("audit.log", {"event": "Monitor Host5", "why": "routine"}, previous_hash=h0)
    assert verify_integrity("audit.log") is True       # chain intact
    _tamper_with_entry("audit.log")                    # simulate tampering
    assert verify_integrity("audit.log") is False      # KEY: tampering is detected


def test_every_action_is_logged():
    log_event("audit.log", {"event": "Isolate SubnetX", "why": "lateral movement"}, previous_hash="")
    assert _count_entries("audit.log") == 1              # nothing the agent does is lost


def test_explanation_gathers_the_why():
    exp = explain(state={}, perception={"threats": ["Host4"]}, investigation={"timeline": [...]}, decision="Restore")
    assert "why_i_detected" in exp and "what_i_understood" in exp and "why_this_response" in exp
```

The tests check the essentials: that the audit log detects tampering (it is truly immutable), that every action is logged (nothing is lost), and that the explanation gathers the "why" end-to-end. Run them with `make test`.

---

## Verification: The Definition of "Done"

The phase is complete when the following is met:

- [ ] The agent explains its decisions, combining the LLM's verbalized reasoning and SHAP (reusing your XAI).
- [ ] Every action (as well as decision, escalation, and approval) is recorded in an immutable audit log (reusing your forensics).
- [ ] The log is tamper-proof (append-only, hash-chained).
- [ ] A human can understand why the agent did something (in the moment) and review what it did (afterward).
- [ ] **The key test:** every agent decision is explainable and every action is auditable.

The key test is that of transparency and accountability: if a human can understand *why* the agent decided something and reliably review *what* it did, you have completed the trust that Phase 5 began. Crucially, these two properties (explainable and auditable) are not cosmetic: the explanation is what enables human supervision (the human-in-the-loop in Phase 5), and the audit is what allows for accountability and learning. Together with safety, they finally make the agent's autonomy trustworthy, which is what sets this project apart.

---

## Deliverables and What Comes Next

By closing Phase 6, you have completed trust: the agent not only acts safely (Phase 5), but also explains why it does what it does (reusing your XAI) and records everything it does in an immutable way (reusing your forensics), so that a human can understand and audit it. You have built the "explainable" and "auditable" properties of your approach, and you have seen how they support safety (the explanation is what makes the human-in-the-loop function). With the five properties of trustworthy autonomy now in place (anchored, bounded, explainable, auditable, and the upcoming agent's own safety), you have an agent that can truly be trusted.

The next step, **Phase 7**, unites everything built: **the agentic loop**. You will integrate the layers (perception, investigation, decision, action, trust) into the closed-loop agent, so that it handles an incident end-to-end (perceiving, investigating, deciding, acting in a bounded manner, explaining and logging, and starting over), and you will consider a multi-agent architecture with specialized roles, just as leading agentic SOCs do. This is where the agent comes to life as a whole. You have gone from "the agent is safe, transparent, and accountable" to being on the verge of "the agent handles a complete incident, from start to finish."
