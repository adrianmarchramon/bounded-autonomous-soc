# Phase 7 — The Agentic Loop: Integration

> So far, you have built five pieces: perception (sees the network), investigation (understands the incident), decision (plans the response), action (executes safely), and trust (explains and audits). In this phase, you unite them into the **agentic loop**, which is where the system transitions from five separate components to a *live agent*: one that receives an incident and handles it on its own, from start to finish. The guiding concept of this phase, and the key to making the loop work, is **re-evaluation**: an incident is not resolved with a single, one-off response because it *evolves* (the attacker reacts to your defenses, your defenses take effect or do not). Thus, the agent acts with what it currently knows, perceives again, and *re-evaluates* in light of new information. Furthermore, there is something that distinguishes this loop from any other: **trust runs entirely through it**. It is not simply a "perceive-decide-act" loop, but one where, in each cycle, the agent's actions are bounded, severe issues are escalated, and everything is explained and logged. It is a closed loop *worthy of trust*, and it represents the synthesis of your entire approach.

**Phase objective:** Unify the layers into the closed-loop agent.
**Duration:** Weeks 9-11.
**Upon completion, you will have:** A closed-loop agent that handles an incident from end to end in a trustworthy manner.

---

## The Big Picture

The flow of this phase is the loop that brings the agent to life:

```
   Network Incident
        │
        ▼
   ┌──────────────────────────────────────────────────────────────────────┐
   │  REPEAT until incident is resolved (or escalated / steps exhausted): │
   │                                                                       │
   │   1. PERCEIVE (Phase 2)     ──►  what is happening now on the network  │
   │   2. INVESTIGATE (Phase 3)  ──►  what occurred (grounded)             │
   │   3. DECIDE (Phase 4)       ──►  the response (RL proposes)           │
   │   4. ACT (Phase 5)          ──►  execute BOUNDED, human-in-the-loop   │
   │   5. EXPLAIN + AUDIT (Phase 6) ──►  justify and log everything        │
   │   6. (perceive again, and RE-EVALUATE: the attacker has reacted)      │
   └──────────────────────────────────────────────────────────────────────┘
        │
        ▼
   [ Incident handled in a trustworthy manner ]
```

---

## Step 1 — The Loop That Brings the Agent to Life

It is worth understanding what changes in this phase, as it represents a qualitative leap. You have five excellent layers, but in isolation, they defend nothing: perception sees but does not decide; investigation understands but does not act; decision proposes but does not execute; and so on. What turns them into an *agent* is uniting them in a cycle where they feed into one another: what perception sees feeds the investigation, what investigation understands feeds the decision, what decision proposes passes through safe action, everything is explained and logged, and the outcome alters the network, which perception then reads again. This cycle is the agentic loop, and it is the heart of any autonomous agent (it is, in fact, the "incident response loop" you saw in the fundamentals: sense, reason, act, learn).

Moreover, this is the culmination of your approach. Each layer does what it knows best (perception grounds, investigation reasons, decision plans, action protects, trust provides transparency), and the loop orchestrates them so that the agent as a whole handles an incident. This is the same step you took with your robot when closing its perception-reasoning-action loop, now applied to cyberdefense. Closing the loop is what gives meaning to everything you have built as a cohesive whole.

---

## Step 2 — The Loop, Step by Step

The core of this phase is the loop that integrates the five layers. Create `src/agent/agent.py`:

```python
def manejar_incidente(controller, modelo_rl, inteligencia, detector, max_pasos=100):
    hash_log = ""
    for paso in range(max_pasos):                      # safeguard: do not loop forever
        # 1. PERCEIVE (Phase 2)
        obs = controller.observe()
        estado = comprender_estado(obs, percibir_amenazas(obs, detector))
        if not estado["comprometidos"] and not estado["escaneados"]:
            return {"estado": "resuelto", "pasos": paso, "log": hash_log}   # clean network

        # 2. INVESTIGATE (Phase 3): grounded reasoning
        investigacion = investigar(estado, evidencia(obs), inteligencia)

        # 3. DECIDE (Phase 4): RL proposes the response
        accion = decidir(estado, modelo_rl)

        # 4. ACT (Phase 5): BOUNDED execution, escalating severe issues to a human
        resultado = ejecutar_accion(accion.tipo, accion.objetivo)

        # 5. EXPLAIN + AUDIT (Phase 6): transparency and accountability
        explicacion = explicar(estado, obs, investigacion, accion)
        hash_log = registrar("audit.log", {"evento": resultado, "por_que": explicacion}, hash_log)

        # 6. perceive again and RE-EVALUATE (the network has changed: the attacker reacted)
    return {"estado": "sin_resolver_en_el_limite", "pasos": max_pasos, "log": hash_log}
```

Notice how each step reuses a layer you have already built: `comprender_estado` (Phase 2), `investigar` (Phase 3), `decidir` (Phase 4), `ejecutar_accion` (Phase 5), `explicar`, and `registrar` (Phase 6). The loop does not add new intelligence; it *orchestrates* what you already have. Observe the complete flow: perceive, investigate, decide, act safely, explain and log, and start again. The key lies in that final turn: upon perceiving again, the agent sees a network that has changed (its action took effect, and the attacker reacted), and it re-evaluates using this new information.

---

## Step 3 — Re-evaluation Closes the Loop

Here is the most important concept of this phase, and it is worth understanding why the loop is a *loop* and not a linear sequence. It might be tempting to think that the agent should decide on the entire response at once and execute it all. However, that would fail because an incident *evolves*: the attacker reacts to each defense (if you block one path, they try another), your actions may or may not take full effect (a "Remove" action might not completely eliminate the attacker), and the situation changes step by step. A fixed plan, drawn up at the beginning, would become obsolete at the adversary's first reaction.

The solution is **re-evaluation**: during each iteration of the loop, the agent perceives the state again, reinvestigates if necessary, and decides once more based on *current* information. If its response worked, the incident progresses toward resolution; if the attacker found another way, the agent perceives it and reacts; if an action did not have the expected effect, the agent sees this and adjusts. Re-evaluation is what makes the agent *adaptive* in the face of a live incident and a reacting adversary, and it is, quite literally, what *closes* the loop: the outcome of the action feeds back into perception and decision-making. This is the difference between an agent executing a script (which becomes obsolete when the attacker changes tactics) and one responding to a real incident like an analyst would: acting, observing the effect, and adjusting. It also brings together everything you have seen: the fog of war (the imperfect perception at each turn), the adversary (who reacts), and costly actions (which must be chosen carefully in each cycle).

---

## Step 4 — Trust Runs Through the Entire Loop

This is what makes *this* loop different from any other agent, and it represents the synthesis of your approach. In many systems, the loop would simply consist of "perceive, decide, act." In yours, *trust* is woven into every turn: in each cycle, the agent not only acts, but acts in a *bounded* manner (only taking safe actions autonomously, while escalating severe issues to a human), *explains itself* (why it did what it did), and *logs it* immutably. In other words, the five properties of trustworthy autonomy (grounded, bounded, explainable, auditable, and the upcoming security of the agent itself) are not a separate phase: *they operate during every iteration* of the loop.

This matters because it is what distinguishes your agent from a dangerous automaton. A loop that merely perceives, decides, and acts on real machines would be the exact "unbounded autonomous defender" that your approach rejects from the very beginning. Yours, on the other hand, is a *trustworthy* closed loop: no matter how fast or autonomous it is, at every step it respects boundaries, keeps the human in the loop for consequential decisions, and leaves everything explained and logged. Closing the loop in this project is not just about making the agent run on its own; it is about making it run on its own *in a way that can be trusted*. This is the idea that has guided the entire project, now realized in the loop.

---

## Step 5 — The Multi-Agent Architecture

Once you have the loop working, it is useful to know (and, if you wish, adopt) how leading agentic SOCs handle this: through a **multi-agent** architecture. Instead of a single monolithic agent doing everything, the field tends to use multiple *specialized* agents, each with a specific role (one for threat intelligence synthesis, another for investigation, another for decision-making, etc.), coordinated by an **orchestrator** that decides whom to invoke and when. Notice how your layers naturally fit into this pattern: each one could be a specialized agent, and the loop you have just built is, in essence, the orchestrator.

Adopting this architecture offers advantages (each specialized agent can be more capable in its specific task, and the system becomes more modular and scalable), and demonstrating it aligns your project with current industry practices. However, an honest framing is appropriate here: multi-agent architecture is a *refinement*, not a requirement. A well-built, coherent single-agent loop that integrates the five layers with trust woven in is already a solid, complete core. Consider multi-agent systems as a natural evolution (and an excellent stretch goal), but do not feel obligated to tackle it if doing so would compromise the depth of your core implementation. A single-agent loop that works properly and is trustworthy is far better than a half-baked multi-agent swarm.

---

## Verification: The "Definition of Done"

This phase is complete when the following are met:

- [ ] You have integrated all five layers (perception, investigation, decision, action, trust) into the closed loop.
- [ ] The agent handles an incident from end to end, re-evaluating as it progresses.
- [ ] Trust runs through the loop: each cycle acts in a bounded manner, escalates severe issues, explains itself, and logs the events.
- [ ] You have considered (and, optionally, adopted) a multi-agent architecture with an orchestrator.
- [ ] You have verified the loop: it terminates upon resolving the incident, does not loop infinitely, re-evaluates at each turn, and audits every action.
- [ ] **The key test:** The agent perceives, investigates, decides, acts (bounded), and explains itself in a loop when faced with an incident.

The key test represents the live and trustworthy agent: if the agent manages an incident on its own—perceiving, investigating, deciding, acting in a *bounded* manner, and explaining itself in a loop—you have built a true autonomous cyberdefense agent. What makes it work is not any single layer on its own, but their orchestration within a re-evaluating loop. What makes it *trustworthy* is that trust runs through every turn. Being able to handle a live incident and adapt to an adversary, while remaining within safe boundaries, explaining itself, and logging everything, is what distinguishes your agent from a dangerous automaton and marks the culmination of your approach.

---

## Deliverables and What Comes Next

By closing Phase 7, you have achieved what the entire project set out to accomplish: an autonomous cyberdefense agent that handles an incident from end to end in a closed, re-evaluating loop, remaining trustworthy at every step (bounded, supervised, explainable, and auditable). You have united the five layers into a living system, made re-evaluation adaptive to an evolving incident, and woven trust throughout the cycle. You now have an agent that functions, and does so in a way that can be trusted.

The next step, **Phase 8**, looks both outward and at the agent itself: **hardening the system and securing the defender**. On one hand, you will be proactive: you will harden the defended system by searching for vulnerabilities before attackers do (reusing your fuzzer). On the other hand, and crucially, you will protect the *agent itself*, which constitutes an attack surface (recall the Phase 0 threat model): an attacker might attempt to manipulate it through the content it reads (such as prompt injection). Therefore, you will apply agentic security defenses (such as those from the OWASP Top 10 for LLM Applications). This represents the fifth property of trustworthy autonomy: the *security* of the agent itself. You have moved from "the agent handles incidents in a trustworthy manner" to being on the verge of "the agent is, additionally, proactive and resilient to manipulation."
