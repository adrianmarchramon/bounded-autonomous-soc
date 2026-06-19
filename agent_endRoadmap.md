# Roadmap Closure — Autonomous Cyberdefense Agent

> This document compiles what comes after the phases: common mistakes to avoid, stretch goals to go further, learning resources, a milestone summary, and a practical conclusion. The major reflection on what this project means (the synthesis of your portfolio, the culmination of its responsibility arc, and the lesson that what sets one apart is judgment) is already in Phase 9; here, the goal is to provide you with a reference guide to build it well.

---

## 1. Common mistakes and how to avoid them

Throughout the project, there are recurring pitfalls. Interestingly, almost all of them share the same root, which you will see at the end.

**Allowing limitless autonomy (Phase 5).** This is the most serious and tempting mistake: letting the agent execute any action it decides on its own. The problem is that the decision can be wrong (especially under the fog of war) or manipulated, and an autonomous action on real machines can cause irreversible damage. The antidote is found in Phase 5: bound the actions (making only reversible actions autonomous) and place a human in the loop for high-consequence decisions. An agent acting without boundaries is not more powerful; it is more dangerous.

**Reasoning without grounding (Phase 3).** Letting the LLM "investigate" by inventing what happened, instead of reasoning based on evidence. In security, this is especially serious because a hallucination translates into a wrong decision and a real action (ignoring an attack, or disrupting the network in response to a non-existent one). The antidote is found in Phase 3: ground with RAG, and have the agent say "I don't know" rather than inventing.

**Inability to explain or audit decisions (Phase 6).** Building an agent that performs well but is a black box: no one can understand why it did something, nor review what it did afterward. Such an agent, no matter how secure, cannot be trusted because it cannot be validated or held accountable. The antidote is found in Phase 6: explainability (to validate in real-time) and immutable auditing (to review afterward).

**Forgetting that the agent itself is vulnerable to attack (Phase 8).** Focusing so much on the agent's defensive capabilities that you forget to protect the agent itself. However, the agent reads content it does not control and has the power to act, making it a target: an attacker could poison its inputs to turn it against its own systems. The antidote is found in Phase 8: treat inputs as untrusted, apply least privilege, and rely on trust defenses that, fortunately, already protect the agent.

**Entrusting security to a fallible component (Phase 5).** Assuming that because the decision-making (RL) is good, it can be allowed to act without a filter. But no model is infallible, and security cannot depend on it being right. The antidote, again from Phase 5: security must reside in a reliable, deterministic layer, not in the decision-making component.

**Trying to cover everything at once (Phases 0 and 7).** Attempting to build the entire system perfectly from the start with all its layers, the multi-agent architecture, and all stretch goals. The result is an unmanageable and superficial project. The antidote spans the project, from the honest framing of Phase 0 to the note on multi-agent systems in Phase 7: build a deep and coherent core, and leave the rest as future evolution. The judgment to scope down is part of what you demonstrate.

**Ignoring dual-use (Phase 0).** Unintentionally building capabilities that can be used for both defense and attack. The antidote is found in Phase 0: explicitly frame the project from a defensive standpoint, with bounded autonomy and supervision, and acknowledge the risk in the threat model.

**The common thread.** If you look closely, almost all of these mistakes are the *same* error in different disguises: treating *autonomy* (or capability) as the goal, instead of *trustworthy autonomy*. Every error is a way for autonomy to stop being trustworthy: by being boundless, hallucinating, opaque, attackable, or built without judgment. That is why the antidote is always the same, and it is the central idea of the entire project: the goal is not for the agent to be autonomous, but to be autonomous *in a way that can be trusted*. Anyone who internalizes this avoids nearly all the pitfalls in one fell swoop.

---

## 2. Stretch goals (if you want to go further)

If you want to take the project further, here are some natural extensions, ordered from most established to least.

**Multi-agent architecture with an orchestrator.** *What it is:* dividing the agent into several specialized agents (one for threat intelligence synthesis, another for investigation, another for decision-making...), coordinated by an orchestrator, just as leading agentic SOCs do. *Why it is worth it:* each specialized agent can be more capable at its task, the system is more modular and scalable, and it demonstrates that you are operating at an industry standard. *How it connects:* it is the natural evolution of the loop in Phase 7 (your layers are already, almost, specialized agents, and the loop acts as the orchestrator). *Difficulty:* high.

**Continuous learning.** *What it is:* allowing the agent to improve with each incident it handles, rather than remaining static after training. *Why it is worth it:* threats evolve, and a learning defender adapts to new tactics. *How it connects:* it extends the decision-making of Phase 4 (RL), with the caution that online learning, in turn, introduces poisoning risks (Phase 8) that would need to be managed. *Difficulty:* high.

**Red-Blue simulation.** *What it is:* training and evaluating the defender against an *attacking* agent (also AI-driven, for defensive purposes), rather than just scripted attackers. *Why it is worth it:* it tests the agent's robustness against an adversary that also adapts, which is closest to reality. *How it connects:* it extends the environment (Phase 1) and the evaluation (Phase 9). *Difficulty:* high (and this is precisely the area where dual-use requires the most caution: the attacking agent is a defensive research tool, not an end in itself).

**Confidence calibration.** *What it is:* having the agent model its own uncertainty and know *when* it is unsure, in order to escalate to a human based on that uncertainty rather than fixed rules. *Why it is worth it:* it makes human supervision smarter (the human steps in right when the agent doubts itself) and deepens the honesty about uncertainty that you have already cultivated. *How it connects:* it refines Phases 5 and 6 (action and trust/confidence). *Difficulty:* medium-high.

**Sim-to-real in cyberdefense.** *What it is:* bringing the agent closer to real telemetry, beyond the simulated environment, with all necessary precautions. *Why it is worth it:* it is the bridge to a real-world deployment and the ultimate challenge of the field. *How it connects:* it extends the environment (Phase 1) toward reality, just as "sim-to-real" did in your robotics project. *Difficulty:* very high, and delicate: an agent acting on real systems requires much stronger security guarantees, making it more of a research horizon than an immediate next step.

---

## 3. Learning resources

- **Your nine previous projects.** They are, literally, the resources for this one: each one provided you with one of its layers. Reviewing them is the best preparation.
- **Literature on agentic security and agentic SOCs.** The 2026 surveys on agentic AI and cybersecurity, and research papers on incident response with LLMs and hallucination mitigation (Hammar et al. and others): they provide you with the state of the art, closed-loop architecture, and grounding as a key component.
- **CybORG / CSLE and the CAGE challenges.** The cyber operations environment where you build and demonstrate the agent; you already know it from your RL project.
- **Field benchmarks.** CTIBench (threat intelligence), ExCyTIn-Bench (incident investigation), and OpenSec (calibration under adversarial evidence): they provide the mindset of *what* and *how* to evaluate.
- **The OWASP Top 10 for Agentic Applications.** The reference framework to secure the agent itself (goal hijacking, tool misuse, memory poisoning) and its mitigations.
- **Concepts to master.** The incident response loop; bounded autonomy and trust; grounding to reduce hallucinations in security; the dual-use dilemma; the agent as an attack surface; and the integration of perception, reasoning, decision-making, and action with responsibility.

---

## 4. Milestone Summary

| Phase | Milestone | Reuses |
|------|------|-----------|
| 0 | Setup, focus (trustworthy autonomy), security, and threat model | — |
| 1 | The environment: a network to defend | Defense with RL |
| 2 | Perception: monitoring and detection | Honeypot |
| 3 | Investigation: grounded reasoning on evidence | RAG + forensics |
| 4 | Decision: planning the response (RL) | Defense with RL |
| 5 | Action: bounded and with human-in-the-loop (the core of security) | Robotics (the idea) |
| 6 | Trust: explainability and auditing | XAI + forensics |
| 7 | The agentic loop: integration | (synthesis) |
| 8 | Hardening the system and securing the agent | Fuzzer + agentic OWASP |
| 9 | Rigorous evaluation, production, and presentation | MLOps + (everyone's rigor) |

The five properties of trustworthy autonomy, and where they are built: **grounded** (Phases 2-3), **bounded** (Phase 5), **explainable** (Phase 6), **auditable** (Phase 6), **secure** (Phase 8).

---

## 5. A final reflection (practical)

The major reflection on the meaning of this project is already in Phase 9, so here is just a practical piece of advice for building it, which is, in itself, true to its approach.

Build it *layer by layer and in order*, and resist the temptation to cover everything at once. The most sensible path is: set up the environment (Phase 1), and then add one layer at a time (perception, investigation, decision-making), verifying each one before moving forward. Once you have action (Phase 5) and trust (Phase 6), close the loop (Phase 7): at that point, you will already have a complete and trustworthy agent, even if it is simple. That is your *deep and coherent core*, and it is an excellent project on its own. Hardening (Phase 8), rigorous evaluation (Phase 9), and stretch goals elevate it, but the core is what truly matters.

In other words: the success of this project is not measured by how much you cover, but by the depth and coherence of what you build, and by ensuring that trust runs through every single layer. An agent that handles an incident from start to finish in a trustworthy manner is worth infinitely more than a massive, half-finished system. If you build that core carefully, you will have realized the idea that gives meaning to everything: autonomy you can truly trust. And that, as you already know, is the hard part, the rare part, and what matters most.
