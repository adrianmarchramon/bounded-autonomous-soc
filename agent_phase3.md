# Phase 3 — Investigation: Reasoning over Evidence, Grounded

> In Phase 2, perception detected that *something is wrong* (which hosts are compromised, where). In this phase, the agent makes the leap from *detecting* to *understanding*: it investigates what happened, reconstructing the history of the incident (how the attacker got in, what they did, in what order), reasoning over the evidence with an LLM. This is the first time the agent *reasons*, and it reunites two of your projects at once: the grounded reasoning of your RAG and the timeline reconstruction of your forensics work. But there is one discipline at the heart of this phase that is non-negotiable—not optional—in security: **grounding to reduce hallucinations**. Because here, a hallucination is not an anecdotal detail: if the agent *invents* what happened, it will make decisions and act based on a fiction, and in a system that interacts with real machines, that is dangerous. Therefore, the agent will reason *only* over the evidence and retrieved intelligence, and will say "I don't know" rather than making things up. It is the step that turns detection into a reliable understanding of the incident.

**Phase Objective:** For the agent to investigate an incident by reasoning over the evidence, without hallucinating.
**Duration:** Weeks 3-5.
**Upon completion, you will have:** An investigation layer that, when faced with an incident, reconstructs and explains what happened, grounded in the evidence.

---

## The Big Picture

The flow of this phase turns detection into a grounded understanding of the incident:

```
   Incident detected (Phase 2) + evidence (telemetry, events)
        │
        ▼
   [ RAG: retrieve intelligence ]  ──►  attack patterns, runbooks, past incidents
        │
        ▼
   [ LLM: GROUNDED reasoning ]  ──►  reconstruct what happened (without inventing)
        │
        ▼
   [ Incident timeline ]  ──►  the attack sequence, with its evidence and uncertainty
```

Add the dependencies (the same ones as your RAG):

```bash
uv add llama-index-llms-ollama llama-index-embeddings-ollama
```

---

## Step 1 — From Detecting to Understanding

It is worth understanding the leap the agent makes in this phase, because this is where it begins to reason. Phase 2's perception answers "which hosts are affected?"; investigation answers a much richer question: "what happened here?". That is, reconstructing the history of the incident: the attacker scanned the user subnet, exploited host X, escalated privileges, and from there laterally moved toward server Y. This understanding is what allows for deciding on a good response (it is not the same to respond to an initial scan as to respond to an attacker already entrenched on a critical server).

And here you reuse two of your projects at once, in a very natural way. From your **RAG**, you bring the reasoning of an LLM grounded in retrieved information (here, threat intelligence, runbooks, and past incidents). From your **forensics**, you bring the idea of reconstructing what happened as a *timeline* from the evidence. The investigation is, in essence, an automated security analyst reasoning over clues to understand the attack—precisely the confluence of those two skills of yours.

---

## Step 2 — Grounded Reasoning with RAG: Reducing Hallucinations

The core of this phase is getting the LLM to reason over the evidence, but in a *grounded* manner, to reduce hallucinations. First, you retrieve relevant intelligence using RAG (reusing your project): in response to the incident, you search for known attack patterns, response runbooks, and similar past incidents. Then, you provide the LLM with the evidence and that intelligence, asking it to reconstruct what happened under the discipline of grounding. Create `src/investigation/investigate.py`:

```python
from llama_index.llms.ollama import Ollama

llm = Ollama(model="llama3", request_timeout=120.0)   # reused from your RAG

INVESTIGATE_PROMPT = """You are a security analyst investigating an incident. Reconstruct what \
has occurred based ONLY on the evidence and retrieved intelligence. DO NOT make things up.

Network state (from perception): {estado}
Evidence (telemetry, events): {evidencia}
Retrieved threat intelligence (RAG): {contexto}

Strict instructions:
- Reason ONLY based on the evidence and intelligence provided above. DO NOT invent steps, hosts, or facts.
- If the evidence does not allow you to assert something, say so ("there is insufficient evidence to determine X").

Respond in JSON with:
- "linea_de_tiempo": the reconstructed sequence of the incident (each step with the supporting evidence).
- "evaluacion": what kind of attack it looks like and its scope.
- "incertidumbre": what cannot be determined with the available evidence.
"""


def investigar(estado, evidencia, indice_inteligencia):
    contexto = indice_inteligencia.as_query_engine().query(str(estado))   # RAG, reused
    prompt = INVESTIGATE_PROMPT.format(estado=estado, evidencia=evidencia, contexto=contexto)
    return parse_json_safe(llm.complete(prompt).text)
```

Notice how the prompt embodies the grounding from your RAG, and why it is so critical here. The instruction is strict: "reason ONLY based on the evidence and intelligence; DO NOT make things up; if you cannot assert something, say so." This is exactly what cutting-edge research in 2026 focuses on (work on LLM-based incident response planning insists on *reducing hallucinations* by grounding on reliable data using techniques like RAG). The RAG provides the expert knowledge (known attack patterns) that the agent must not invent, and the prompt forces it to base everything on evidence. This is the same discipline that prevented the RAG from making up answers; here, it prevents making up the story of an attack.

---

## Step 3 — Reconstructing the Incident Timeline

The key product of the investigation is an incident **timeline**, and here you directly reuse your forensics work. Instead of a vague verdict ("there is an attack"), the investigation reconstructs the *sequence*: what happened first, what happened next, and what evidence supports each step. This turns scattered clues (compromised hosts, scans, anomalies) into a coherent story of the attack:

```python
# Example of the output: a reconstructed timeline grounded in evidence
{
  "linea_de_tiempo": [
    {"paso": 1, "evento": "User subnet scan", "evidencia": "Host3, Host4: Scan"},
    {"paso": 2, "evento": "Exploitation of Host4", "evidencia": "Host4: Exploit, User"},
    {"paso": 3, "evento": "Privilege escalation on Host4", "evidencia": "Host4: Privileged"},
  ],
  "evaluacion": "Phishing intrusion with lateral movement in progress; scope: user subnet.",
  "incertidumbre": "There is no evidence that the attacker has reached the server subnet yet."
}
```

Just as the timeline (the "super timeline") in your forensics work ordered events to tell the story of what happened, here it orders the incident evidence to reconstruct the attack, grounded in each case by the supporting evidence. Notice the *uncertainty* field: the investigation explicitly states what it *cannot* assert, which is the other side of grounding (not only avoiding inventing what you do not know, but also acknowledging what you do not know). This grounded and honest timeline is what Phase 4's decision-making will use to choose the appropriate response.

---

## Step 4 — Why Grounding is Non-Negotiable in Security

It is worth pausing here to understand why the discipline of this phase matters so much, because it demonstrates that you understand what is at stake. In your RAG, a hallucination yielded an incorrect answer (bad, but recoverable). Here, a hallucination carries a much higher cost: if the agent *invents* what happened (a non-existent attack or an exaggerated scope), Phase 4's decision will be based on that fiction, and Phase 5's action will execute on real machines. A hallucination could lead the agent to ignore a real attack, or to "respond" to a non-existent one, disrupting the network for no reason. This is why grounding here is not just a desirable enhancement: it is *non-negotiable*.

This has two implications that align with your approach. First, the agent must **say "I don't know"** rather than making things up, reusing that same discipline from your RAG: when faced with insufficient evidence, the correct action is to recognize the uncertainty (and, as you will see, escalate to a human), not to fabricate a conclusion. Second, this grounded investigation is what makes the agent's decisions *trustworthy* and *explainable*: because understanding the incident is backed by evidence (not invention) and presented as a timeline that a human can review, decisions based on it will rest on a solid, auditable foundation. In this way, this phase builds two properties of trustworthy autonomy at once: being **grounded** (reasoning about reality) and being **explainable** (its understanding can be reviewed). Grounding in security is what separates an agent you can trust from one that is a hazard.

---

## Step 5 — Testing

Verify the logic that does not require the model (that the prompt enforces grounding, and the parsing). Complete `tests/test_investigation.py`:

```python
from src.investigation.investigate import INVESTIGATE_PROMPT, parse_json_safe


def test_prompt_enforces_grounding_and_honesty():
    prompt = INVESTIGATE_PROMPT.format(estado="-", evidencia="-", contexto="-")
    assert "ONLY" in prompt and "DO NOT" in prompt          # reasons only on evidence
    assert "insufficient evidence" in prompt                # say what you do not know


def test_evidence_and_intelligence_enter_the_prompt():
    prompt = INVESTIGATE_PROMPT.format(estado="Host4 compromised", evidencia="Host4: Exploit", contexto="phishing pattern")
    assert "Host4" in prompt and "phishing" in prompt            # reasons about retrieved data


def test_parsing_extracts_timeline():
    r = parse_json_safe('{"linea_de_tiempo": [{"paso": 1, "evento": "Scan"}], "evaluacion": "intrusion", "incertidumbre": "-"}')
    assert r["linea_de_tiempo"][0]["evento"] == "Scan"
```

The tests verify that the prompt enforces grounding and honesty about what is unknown, that evidence and intelligence are incorporated into the reasoning, and that parsing successfully extracts the timeline. Run them with `make test`.

---

## Verification: Definition of Done

The phase is complete when the following is met:

- [ ] You have built the investigation layer with an LLM + RAG using threat intelligence, runbooks, and past incidents (reusing both RAG and forensics).
- [ ] You reconstruct what happened as a timeline, grounded in the evidence.
- [ ] You apply the discipline of grounding to reduce hallucinations, and the agent says "I don't know" rather than making things up.
- [ ] You understand why grounding is non-negotiable in security (a hallucination leads to incorrect action).
- [ ] **The key test:** the agent investigates an incident by reasoning over actual evidence, not over hallucinations.

The key test is that of grounded reasoning: if the agent reconstructs what happened based on actual evidence (and acknowledges what it cannot determine) instead of making up a plausible story, you have built a trustworthy investigation. The key word is *grounded*: in security, brilliant but fabricated reasoning is worse than useless—it is dangerous, because it leads to actions on real systems. Having the understanding of the incident grounded in evidence, and honest about its uncertainty, is what makes it worthy of the trust on which the response will be decided.

---

## Deliverables and Next Steps

By wrapping up Phase 3, you have the layer that enables the agent to *understand*, not just detect: an investigation that, in the face of an incident, reconstructs what happened as a timeline grounded in the evidence, reusing both your RAG and forensics work. You have done so with the non-negotiable discipline of grounding, reducing hallucinations and saying "I don't know" rather than making things up—which simultaneously builds both the grounded and explainable properties of your approach. You now have a reliable and auditable understanding of the incident, ready to lay the groundwork for the response.

The next step, **Phase 4**, is where the agent decides what to do: **Decision**. You will build the layer that, based on the understanding of the incident, plans the response as a *sequence* of defensive actions under uncertainty against the adversary, reusing your RL-based defense project (on the same CybORG environment). This is the piece that provides intelligent and adaptive decision-making, backed by the grounded investigation you just built. You have gone from "the agent understands what happened" to being on the verge of "the agent decides, step-by-step, how to respond."
