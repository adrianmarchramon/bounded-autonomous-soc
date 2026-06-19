# Phase 4 — Decision: Planning the Response

> In Phase 3, the agent understood *what occurred* (the anchored investigation). In this phase, it decides *what to do about it*: it plans the response. And here is an idea you already demonstrated in your RL defense project, which is the reason for using RL again: incident response is not *a single* action, but a *sequence* of decisions unfolding over time, under uncertainty, and against a reacting adversary. This is precisely the problem for which reinforcement learning is the right tool (not a classifier that makes a decision all at once). The guiding concept of this phase is exactly that: making sound decisions for a defense *sequence* in an uncertain and adversarial environment. And, true to the layered architecture of your approach, remember that this layer *proposes* the response; executing it safely will be the job of the next one. It is the piece that provides the adaptive intelligence of the defense.

**Phase objective:** For the agent to decide on a response sequence under uncertainty.
**Duration:** Weeks 5-7.
**Upon completion, you will have:** A decision layer that, based on the understanding of the incident, plans the response as a sequence of defense actions.

---

## The Big Picture

The workflow of this phase converts incident understanding into a planned response:

```
   Incident understanding (Phase 3) + network state (Phase 2)
        │
        ▼
   [ Decision policy (RL / PPO) ]  ──►  what is the best defense action NOW?
        │  (learned against the adversary, reusing your RL)
        ▼
   [ The next action in the sequence ]  ──►  Analyse / Remove / Restore / Decoy...
        │
        ▼
   (the action in Phase 5 will execute it safely; the decision PROPOSES)
```

Add the RL dependency (the same as in your RL defense project):

```bash
uv add stable-baselines3
```

---

## Step 1 — Why RL: The Response is a Sequence

It is worth starting by understanding *why* RL, as it reaffirms the thesis you already demonstrated in your RL defense project. It might be tempting to treat the response as a *classification* problem: "given this incident, what action should be taken?", and choosing one all at once. But that misunderstands the problem. Defending is not a single action, but a *sequence* that unfolds over time: you monitor, analyze a suspicious host, remove an attacker session, deploy a decoy, restore a compromised host... and each decision alters the situation and triggers a reaction from the adversary. What matters is not getting an isolated action right, but choosing the *sequence* that best defends the network throughout the course of the incident.

This is exactly the problem of **reinforcement learning**: optimizing a sequence of decisions for a long-term goal within a reacting environment. This is the same central thesis of your RL defense project ("RL is for the *sequential* problem of defending, not for classifying intrusions"), and here you reuse it directly, on the same CybORG environment. The agent's decision is not a classifier: it is a *policy* that knows what to do at each point of an evolving incident. Recognizing that this is an RL problem, and not a classification one, is precisely the "right tool for the job" criterion that your portfolio demonstrates.

---

## Step 2 — The Decision Policy with RL

The core of this phase is training the decision policy with RL, reusing your expertise from your RL defense project (Stable-Baselines3 and PPO, on CybORG). The agent learns, through trial and error against the scripted attacker, which sequence of actions best defends the network. Create `src/decision/decide.py`:

```python
from stable_baselines3 import PPO

def train_policy(env, steps=1_000_000):
    """Trains the decision policy with RL (PPO), reusing your RL defense."""
    model = PPO("MlpPolicy", env, verbose=1)
    model.learn(total_timesteps=steps)     # learns against the attacker (B_line, Meander)
    return model


def decide(state, model):
    """In operation: the policy decides the next defense action, given the state."""
    action, _ = model.predict(state, deterministic=True)
    return action       # e.g., Analyse Host4, Remove, Restore, or deploy a Decoy
```

What is important, and where it becomes clear that the policy is *intelligent* rather than naive, is what it learns to balance. Recall from Phase 1 that actions have a *cost*: the reward function penalizes both the attacker gaining access and the agent's own disruptive actions (such as Restore disrupting a host). Therefore, by maximizing long-term reward, the policy learns to defend effectively *without* disrupting the entire network: using drastic actions only when they are worth it, deploying decoys to detect the attacker, and acting in a timely manner. This is the adaptive decision-making that RL provides and that a classifier could not offer.

---

## Step 3 — Deciding Under Uncertainty and Against an Adversary

Here lies what makes this decision challenging, demonstrating your understanding of the problem. The agent makes decisions under conditions that RL is well-equipped to handle, but which are important to keep in mind. First, under **uncertainty**: as you saw in Phases 1 and 2, perception is partial and noisy (the fog of war), so the agent makes decisions based on an imperfect view of the state. RL learns a policy that is *robust* to this uncertainty because it is trained specifically within this noisy environment. Second, against a reacting **adversary**: it is not a static environment, but one where the attacker responds to defenses, meaning the policy must be *adaptive*. By training against attackers (B_line, Meander), the RL agent learns to counteract their strategies.

And there is an integration detail that demonstrates maturity: the decision does not rely solely on raw telemetry, but on the **anchored understanding** produced by the Phase 3 investigation. The agent decides based on what it *understands* to have occurred (the incident timeline, anchored in evidence), rather than isolated bits of data. This makes the decision more informed and robust: reacting to "Host 4 reports Exploit" is not the same as deciding with the knowledge that "there is an active phishing intrusion with lateral movement originating from the user subnet." The grounding you established in Phase 3 *flows* into this step: an anchored decision rooted in an anchored understanding.

---

## Step 4 — The Decision Proposes, the Action Disposes

This is where the layered architecture of your approach materializes, and it is important to keep this clear for the next phase. This layer *decides* and *proposes* the response (the next defense action, the sequence), but *does not execute* it autonomously and without limits. Executing the response (and doing so in a bounded and secure manner, with a human-in-the-loop for high-consequence actions) will be the work of Phase 5.

This separation is deliberate and fundamental, serving as the equivalent here to the "LLM reasons, navigation drives" concept of your robot. The RL policy is excellent at deciding *which* response is best, but the policy's decision should not simply translate directly into actions on real machines: because the policy, like any model, can make mistakes (especially under uncertainty), and because some actions (such as restoring a critical server or isolating a segment) have severe consequences that require oversight. For this reason, the decision *proposes*, and the action layer (reliable, bounded, supervised) *disposes*. Keeping this boundary clear now is what will allow you, in Phase 5, to build the bounded autonomy that makes the agent trustworthy. This layer is the brain of the response; the next is the safe hand.

---

## Step 5 — Verification

RL is not verified with traditional unit tests (as the policy is *learned*), so this is a sanity check, much like in your RL defense project. Confirm that: the policy trains without errors; it produces *valid* actions given a state; and, as the key indicator, that a trained policy defends *significantly better* than a random one (higher reward, meaning less penalty). A sanity check:

```python
def verify_policy(model, env, episodes=50):
    def mean_reward(policy):
        total = 0
        for _ in range(episodes):
            obs, _ = env.reset(); done = False
            while not done:
                action = policy(obs)
                obs, r, term, trunc, _ = env.step(action); total += r; done = term or trunc
        return total / episodes
    trained = mean_reward(lambda o: model.predict(o, deterministic=True)[0])
    random_policy = mean_reward(lambda o: env.action_space.sample())
    assert trained > random_policy, "The trained policy does not outperform the random policy"
    print(f"Policy OK: trained={trained:.1f} vs random={random_policy:.1f} (less penalty)")
```

This check confirms the essentials: that the learned decision-making indeed defends better than acting randomly, which is the signal that the policy has learned something useful. (The rigorous evaluation of how well it defends will come in Phase 9.)

---

## Verification: The "Definition of Done"

The phase is complete when the following is met:

- [ ] You have built the decision layer with RL (Stable-Baselines3, PPO), reusing your RL defense.
- [ ] The policy plans the response as a *sequence* of actions, not as an isolated decision.
- [ ] The decision handles uncertainty (the fog of war) and the adversarial nature of the environment.
- [ ] The decision relies on the anchored understanding of the investigation and *proposes* the response (it does not execute it yet).
- [ ] **The key test:** The agent decides what to do, step-by-step, during an incident.

The key test is that of sequential decision-making: if the agent decides the next defense action at each moment of an evolving incident (rather than a single response all at once), you have built the adaptive brain of the response. What makes it appropriate is having recognized that defending is a *sequential* and *adversarial* problem (requiring RL), not a classification one, and having let the decision *propose* while the secure action layer *disposes*. That decision, adaptive and anchored, is the intelligence of the defense.

---

## Deliverables and What Comes Next

At the close of Phase 4, you have the brain of the response: a decision layer that, based on the anchored understanding of the incident, plans the response as a sequence of defense actions under uncertainty and against the adversary, reusing your RL defense. You have reaffirmed the "right tool for the job" criterion (RL for the sequential problem, not classification), ensured that the decision is backed by the anchored investigation, and maintained the boundary of the layered architecture: the decision *proposes*. You have the adaptive intelligence of the defense, ready to be executed safely.

The next step, **Phase 5**, is the core of trust: **bounded action with a human-in-the-loop**. You will build the layer that *executes* the responses proposed by the decision layer, but within safe boundaries (bounded and, where possible, reversible actions) and with a human approving high-consequence ones, ensuring the agent never autonomously executes anything harmful or irreversible. This is the realization of the "bounded" property of your approach, and the equivalent, here, of the navigation system that ensures your robot's safety. You have moved from "the agent decides, step-by-step, how to respond" to being on the verge of "the agent acts, but with a trustworthy level of autonomy."
