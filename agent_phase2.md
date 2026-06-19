# Phase 2 — Perception: Monitor and Detect

> In Phase 1, you saw that the agent receives raw telemetry from the environment (the noisy, partial, per-host state); in this phase, you build the layer that converts this telemetry into a useful *understanding* of what is happening on the network: what threats exist, where they are, and how severe they are. The guiding idea of this phase is that perception is the **foundation of grounding**, precisely the "grounded" property you established in your approach: it connects the agent's reasoning and decisions with what is actually happening on the network. And, true to the reality discovered in Phase 1, you will do so with honesty regarding the "fog of war": detection will be *probabilistic*, not certain, and the system will have to reason about this imperfect picture without over-relying on it. The good news is you already know how to do this: you will reuse the anomaly detection from your honeypot. Perception is the first pillar for the agent to act on reality, rather than on an illusion.

**Phase Objective:** Enable the agent to perceive what is happening on the network.
**Duration:** Weeks 2-3.
**Upon completion, you will have:** A perception layer that, using telemetry, detects threats and understands the network state, grounded in real data.

---

## The Big Picture

The workflow of this phase converts telemetry into a grounded understanding:

```
   Raw telemetry (Phase 1: noisy, partial, per-host state)
        │
        ▼
   [ Monitor and detect ]  ──►  anomalies and threats (reusing the honeypot)
        │
        ▼
   [ State understanding ]  ──►  "host X compromised (privileged); Y, Z scanned; spreading"
        │
        ▼
   [ The foundation of grounding ]  (honest about the fog of war: probabilistic detection)
```

Add the detection dependency (the same one from your honeypot):

```bash
uv add scikit-learn
```

---

## Step 1 — Perception as the Foundation of Grounding

Before diving into code, it is worth understanding why this layer is so important: it represents one of the five properties of trustworthy autonomy you established as your focus—being **grounded**. In the upcoming phases, the agent will investigate, decide, and act; however, all of that is only valuable if it relies on a faithful perception of what is actually happening on the network. If the agent reasons and decides based on an incorrect picture of the state, its actions, no matter how intelligent, will be wrong. In security, this is dangerous: it could ignore a real attack or "respond" to a non-existent one, disrupting the network. Perception is precisely what prevents this: it is the bridge between real telemetry and the agent's reasoning, *grounding* it in reality.

This is why this layer is the foundation of grounding, a concept that spans your entire portfolio (from your RAG to your robot). Just as the robot had to reason about what it actually perceived, and the RAG had to ground its answers in documents, here the agent must ground everything in what perception reports about the network state. And, like in those projects, the quality of perception determines the quality of everything that follows. Building this layer well means building the foundation of trust.

---

## Step 2 — Monitor and Detect

The core of this phase is the layer that monitors telemetry and detects threats, and here you directly reuse what you mastered in your honeypot. Phase 1 telemetry (the activity and compromise state per host) already highlights part of what is happening, but a rich perception goes further: it detects *anomalies*, unusual patterns that betray an attack, using the unsupervised methods from your honeypot (such as IsolationForest). Create `src/perception/detect.py`:

```python
from sklearn.ensemble import IsolationForest

def detect_anomalies(historical_telemetry):
    """Detects anomalous patterns in telemetry (reused from your honeypot)."""
    model = IsolationForest(contamination=0.05)
    model.fit(historical_telemetry)
    return model            # will flag anything deviating from normal behavior as anomalous


def perceive_threats(observation, detector):
    """Combines direct telemetry (host states) with anomaly detection."""
    threats = []
    for host, (activity, compromise) in observation.items():
        if activity in ("Scan", "Exploit") or compromise != "No":
            threats.append({"host": host, "activity": activity, "compromise": compromise})
    # ...and add the anomalies flagged by the detector on top of the telemetry
    return threats
```

It is the same principle as in your honeypot: converting a raw stream of data (logs there, telemetry here) into threat signals. In fact, it is the same theme you encountered there and in your forensics project (converting a flood of data into intelligence), now applied to real-time network telemetry.

---

## Step 3 — From Telemetry to State Understanding

Detecting isolated threats is not enough; the subsequent layers (investigation, decision) need an *understanding* of the network state. Therefore, perception synthesizes individual signals into a situational picture. Create `src/perception/state.py`:

```python
def understand_state(observation, threats) -> dict:
    """Converts telemetry and threats into a useful understanding of the network state."""
    compromised = [{"host": h, "level": c} for h, (a, c) in observation.items() if c != "No"]
    scanned = [h for h, (a, c) in observation.items() if a == "Scan"]
    return {
        "compromised": compromised,                  # which hosts are compromised, and at what level
        "scanned": scanned,                          # which hosts have been touched by the attacker
        "severity": _evaluate_severity(compromised),  # are critical systems at risk?
        "propagation": _evaluate_propagation(observation), # does the attack seem to be spreading?
    }
```

The idea is for perception to "tell a story" about the state: not just a list of bits, but "host X is compromised at a privileged level, hosts Y and Z have been scanned, and the attack seems to be spreading from the user subnet." This understanding is what Phase 3's investigation will reason about, and what Phase 4's decision-making will use. It should be highly informative (identifying what is relevant for defense: what is at risk, its severity, and whether it is spreading) and, above all, presented in a clear, structured format. It is, quite literally, the picture of the world the agent will act upon.

---

## Step 4 — Perceiving in the Fog of War

Here comes the dose of reality that demonstrates maturity, representing the core challenge of grounding in this project. As you saw in Phase 1, telemetry is *partial and noisy*: the agent does not see the complete, true state of the network, and detection is prone to false positives (believing there is an attack when there is none) and false negatives (failing to detect a real one). Therefore, the perception you build is *probabilistic*, not certain: it is the best estimate of the state, not absolute truth.

This leads to two consequences that demonstrate your understanding of the problem and align with your approach. First, perception must be *honest about its uncertainty*: instead of asserting "host X is compromised" as an absolute fact, it is better to manage confidence levels so that subsequent layers know how reliable each signal is. Second, and crucially, the rest of the system must reason about this imperfect image *without over-relying on it*: this is precisely the challenge of grounding (basing actions on perception while knowing perception can fail). It justifies decisions in later phases, such as investigating before acting (Phase 3) or refraining from taking drastic, irreversible actions based on a single uncertain signal (the bounded autonomy of Phase 5). Recognizing that perception is imperfect, and designing the system with this in mind, is far more mature (and secure) than pretending to have a perfect view of the network. Grounding in security is not about assuming you see the truth; it is about acting judiciously on an uncertain truth.

---

## Step 5 — Tests

Verify the logic that does not require the trained detector (state understanding). Complete `tests/test_perception.py`:

```python
from src.perception.state import understand_state


def test_identifies_compromised_hosts_and_their_level():
    obs = {"Host1": ("None", "No"), "Host2": ("Exploit", "Privileged"), "Host3": ("Scan", "No")}
    state = understand_state(obs, threats=[])
    assert {"host": "Host2", "level": "Privileged"} in state["compromised"]
    assert "Host3" in state["scanned"]
    assert state["compromised"] != []     # detects compromise


def test_clean_network_does_not_generate_false_threats():
    obs = {"Host1": ("None", "No"), "Host2": ("None", "No")}
    state = understand_state(obs, threats=[])
    assert state["compromised"] == [] and state["scanned"] == []   # does not make up threats
```

The tests verify that perception correctly identifies compromised hosts and their level (telling the correct story), and that it does not generate false threats when the network is clean (no fabrications). Run them using `make test`.

---

## Verification: The Definition of "Done"

The phase is complete when the following are met:

- [ ] You have built a perception layer that monitors telemetry and detects threats (reusing your honeypot).
- [ ] You convert telemetry into a useful understanding of the network state (what is compromised, where, and its severity).
- [ ] Perception is honest about its uncertainty (it is probabilistic, due to the fog of war).
- [ ] You understand that the rest of the system must reason about this imperfect image without over-relying on it.
- [ ] **The key test:** The agent perceives what is happening on the network, grounded in real data.

The key test centers on built grounding: if the agent perceives the network state from real telemetry and understands it in a useful way, you have built the foundation for the "grounded" property of your approach, which will connect reasoning and decisions to reality. Having done so with honesty regarding the fog of war (a probabilistic, rather than certain, perception) is what makes that foundation reliable and truly addresses the challenge. It is not enough to merely read telemetry; it must be *understood*, and this must be done with the awareness that it is imperfect.

---

## Deliverables and What Comes Next

By closing Phase 2, you have the first pillar of the agent's intelligence: a perception layer that converts telemetry into an understanding of the network state (what threats exist, where they are, and their severity), grounded in real data and honest about its uncertainty. You have reused the anomaly detection from your honeypot on a new problem, and you have materialized the "grounded" property of your approach, accounting for the fog of war. You have built the foundation of grounding: the bridge between the network and the agent's reasoning.

The next step, **Phase 3**, is where the agent begins to reason: **investigation**. You will build the layer that, upon detecting an incident, investigates what happened by reasoning about the evidence using an LLM + RAG over threat intelligence, reconstructing the attack story, and reusing your RAG and forensics work. Crucially, it will do so *grounded* in the perception you just built and with the discipline of *reducing hallucinations* (vital in security). You have gone from "the agent perceives what is happening on the network" to being on the verge of "the agent investigates and understands what is occurring, without hallucinating."
