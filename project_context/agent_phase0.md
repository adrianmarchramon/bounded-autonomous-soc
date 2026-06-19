# Phase 0 — Setup, Vision, and Focus

> This phase lays the foundation, and in this project—the most consequential of your entire portfolio—that means, above all, one thing: **establishing the focus, safety, and threat model before writing a single line of active code**. There is a temptation that would be a grave mistake: rushing to build "an AI that defends on its own," without limits, without auditing, without considering how it might fail or be manipulated. The most important decision of the project is to avoid that trap, committing yourself in writing to *trustworthy* autonomy: bounded, auditable, explainable, grounded, and secure. And since the agent here will have the power to act on systems, there is a piece that was not as central in your other projects but is indispensable here from day one: the *threat model*, which includes the agent itself as a potential target. If you do not establish this now, you could build something powerful but dangerous. That is why this phase is not just about "setting up the environment": it is about getting the approach, safety, and risks right, which is where a project proves itself mature rather than reckless.

**Phase Objective:** environment ready, understanding agentic safety, and establishing the focus and trust.
**Duration:** Week 1.
**By the end, you will have:** a well-structured repository, a solid understanding of agentic safety, and, most importantly, the documented focus (trustworthy autonomy), security dimension, and an initial threat model.

---

## The Big Picture

This phase is about getting the foundations right across four fronts, and note that nothing is being defended yet (that begins in Phase 1): here, you prepare and, above all, decide.

```
   [ 1. Repository and environment ] ──► the engineering foundation
   [ 2. Understanding the problem ]  ──► agentic safety, agentic SOC, the response loop
   [ 3. Setting the focus ]          ──► trustworthy autonomy (bounded, auditable,
                                         explainable, grounded, secure) and why
   [ 4. Safety and threat model ]    ──► bounded autonomy, human-in-the-loop, dual-use,
                                         and the agent as an attack surface
```

---

## Step 1 — The Repository and the Environment

You start by setting up the engineering foundation you already master and reuse: the repository with the fundamental structure, the environment using `uv`, code quality with `ruff`, and a `Makefile`.

```bash
uv init centinela           # or the name of your choice
cd centinela
uv add --dev pytest ruff
mkdir -p src/{environment,perception,investigation,decision,action,trust,agent,hardening,evaluation} \
         deployment results docs notebooks tests
```

Configure the `.gitignore` so that the `results/` folder is not pushed to the repository. Regarding project-specific dependencies: you will install the simulation environment (CybORG/CSLE) in Phase 1, and the rest (detection, LLM, RL) in their respective phases, reusing what you already have. In this phase, a lightweight base is sufficient. As you can see, the structure is broader than in your other projects because the system has more layers; having it clear from the beginning will keep you from getting lost in a large project.

---

## Step 2 — Understanding the Problem: Agentic Safety and the Agentic SOC

Before touching anything, spend some time thoroughly understanding the domain. Here, there is a key difference from your previous projects: although you bring all the pieces with you, *agentic safety* (AI that does not just assist, but *operates*) is a new synthesis, with its own character and its own risks. Document what you establish in `docs/context.md`, which will also serve as material for the README.

Make sure you are clear on the fundamental concepts. What **agentic safety** and an **agentic SOC** are (a security operations center where AI agents observe, detect, investigate, decide, and respond with minimal manual intervention). Why cyberdefense aligns so well with agentic AI: because it is, by nature, continuous monitoring, sequential decision-making, tool coordination, and adaptation to an adversary. And, above all, the **closed incident response loop** (sense, reason, act, learn) and why the real challenge is *trust*.

Gaining a deep understanding of this domain now is what will allow you, in subsequent phases, to build each layer with sound judgment and, more importantly, to understand why autonomy needs to be bounded and trustworthy. It is the investment that makes everything else possible (and safe).

---

## Step 3 — Setting the Focus

This is one of the most important steps of this phase, and the intellectual heart of the project. This is where you commit, in writing, to the right approach, to avoid falling into the temptation of "an AI that defends on its own." Document the core idea and its reasoning in `docs/context.md`, because knowing how to explain it is what demonstrates maturity.

The idea is that **the challenge is not autonomy itself, but trustworthy autonomy**, and that trust is *designed* through five properties, each inherited from one of your projects. **Bounded:** the agent's actions are limited and reversible, and high-consequence actions are passed to a human (similar to the navigation that guarantees your robot's safety). **Auditable:** every action is immutably recorded (the traceability of your forensics). **Explainable:** the agent justifies its decisions (your XAI). **Grounded:** it reasons based on real data, not hallucinations, because in security, a hallucination is dangerous (the grounding of your RAG). **Secure:** the agent itself is attackable and must be protected (the new dimension). The reasoning that binds everything together: an autonomous defender without these properties is not an asset, but a liability—worse than having no agent at all, because a power that acts without being trustworthy is, itself, a threat.

And here, it is also highly valuable to establish in writing an **honest framing of ambition**: that what will make this project exceptional is not trying to cover everything, but solving what is truly difficult (trust) with maturity. For this reason, you will build it as a *coherent and deep core*, in phases, without aiming for an unmanageable scope. Committing to a sensible scope now is, in itself, a demonstration of the judgment that the project seeks to showcase. Documenting all of this is what will guide every subsequent decision and, when well explained, will set you apart.

---

## Step 4 — Setting Safety and the Threat Model

The other decisive step, and one that takes on an importance in this project that it did not have in previous ones, is documenting safety and the threat model. This is where the mature is separated from the reckless. You will capture this in two documents.

In `docs/safety.md`, you will document the responsibility dimension. **Bounded autonomy** and the **human-in-the-loop**: the agent acts within boundaries, and high-consequence or irreversible actions require human approval; the agent *augments* the analyst rather than replacing them. It is also worth noting that this reflects how the industry is progressing: organizations move in stages (from monitoring, to governing, to enforcing), not from zero to full autonomy, and recognizing this is a sign of solid judgment. And **dual-use**: the same agentic capabilities that defend can also attack (autonomous reconnaissance, exploitation, evasion), so the project is explicitly framed around *defense*, fully aware of this risk.

In `docs/threat-model.md`, you will establish the threat model, and here lies the detail that demonstrates the highest maturity: in addition to threats to the defended system, you model threats to the *agent itself*. Because the agent reads content it does not control (telemetry, logs, threat intelligence), and this content can be *manipulated* to subvert it (such as a prompt injection that makes it, for example, ignore an attack or take a harmful action), a well-documented risk in the industry (the OWASP Top 10 for LLM Applications / Agentic Applications). Thinking from day one about how your defender could be attacked, and documenting the mitigations, is precisely what distinguishes someone who understands agentic safety from someone who builds a naive agent. Most forget to protect the defender; you place it at the center.

---

## Step 5 — Recording Decisions

Just as in your other projects, keep a record of key decisions: the approach (trustworthy autonomy, with its five properties) and its reasoning, the honest framing of the scope, the safety dimension (bounded autonomy, human-in-the-loop, dual-use), the threat model (including the agent as an attack surface), and how you reuse your nine projects across each layer. Having this in writing will serve you well for the README, help you avoid getting lost in a large project, and demonstrate that you have made deliberate decisions in the project where they matter most.

---

## Verification: Definition of "Done"

The phase is complete when the following are met:

- [ ] The repository has the layered structure, the environment set up with `uv`, code quality configurations, the Makefile, and `results/` is added to the `.gitignore`.
- [ ] You understand agentic safety, the agentic SOC, and the incident response loop, and have documented them in `docs/context.md`.
- [ ] The approach is established and documented: trustworthy autonomy (with its five properties) and its reasoning, alongside the honest framing of the scope.
- [ ] Safety is documented in `docs/safety.md` (bounded autonomy, human-in-the-loop, dual-use) and the threat model in `docs/threat-model.md` (including the agent as an attack surface).
- [ ] **The key test:** you understand why the challenge is *trustworthy* autonomy, not autonomy on its own.

The key test has two halves that reflect the dual character of this phase: comprehension (understanding agentic safety) and judgment (having established the approach to trust, safety, and the threat model). What makes the second half so decisive is characteristic of this project more than any other: *getting the approach, safety, and risks right* is what separates a mature agent from a reckless one. A system that acts with power in a high-consequence domain, without being designed to be trustworthy, is dangerous—no matter how well-built its individual components are.

---

## Deliverables and Next Steps

By closing Phase 0, you have laid solid foundations: the engineering base is ready, you have a firm grasp of agentic safety, and, above all, the focus (trustworthy autonomy), the safety dimension, and a threat model are documented. You have started the most demanding project exactly where a competent professional would: ensuring you understand it and approaching it with the maturity, safety, and risk awareness that such a system demands before writing a single line of active code.

The next step, **Phase 1**, will have you work directly on the agent's world: **the environment**. You will spin up the cyber operations environment (CybORG or CSLE, which you already know from your RL project), understand the network, assets, attacker, telemetry, and defensive actions, and validate the basic interaction loop. This is the first touchpoint with the "world" where the agent will operate—the equivalent, in this context, of your robot's simulator. You have moved from "I understand the problem and have established the focus, safety, and risks" to being on the verge of "I have the simulated network where the agent will perceive and act."
