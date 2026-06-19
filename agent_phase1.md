# Phase 1 — The Environment: A Network to Defend

> In this phase, you set up the "world" where your agent will live: a simulated computer network with an active attacker that the agent must defend. It is the equivalent, in this project, to your robot simulator, and the good news is that you already know it: it is CybORG, the environment you used in your RL defense project. The guiding concept of this phase is to thoroughly understand this world before putting the agent in charge of defending it: what exists on the network, what the attacker does, what the agent *perceives* (the telemetry), and what it can *do* (the defensive actions). And, just like with the robot, you will discover two realities that will shape the entire project: that the agent's perception is *partial and noisy* (the "fog of war" of cybersecurity), and that its actions have *consequences and costs* (it cannot simply tear everything down to defend itself). Understanding this well is what will allow you, in the subsequent phases, to build an agent that acts judiciously in an uncertain world.

**Phase Objective:** Set up the cyber operations environment.  
**Duration:** Weeks 1-2.  
**By the end, you will have:** The environment running, including the attacker and defensive actions, along with a solid understanding of what the agent perceives and what it can do.

---

## The Overall Idea

The workflow of this phase places the agent in its world and establishes an understanding of its interface with it:

```
   [ 1. Start the environment ]  ──►  a simulated network (CybORG/CAGE 2): blue, red, green
        │
        ▼
   [ 2. Perception: telemetry ]  ──►  state per host (partial and noisy): the fog of war
        │
        ▼
   [ 3. Action: defensive actions ]  ──►  Monitor, Analyse, Remove, Restore, Decoy (with cost)
        │
        ▼
   [ 4. Interaction loop ]  ──►  observe, act, against a scripted attacker
```

Add the environment dependency (the same one from your RL project):

```bash
uv add cyborg    # CybORG / CybORG++ (the cleaned-up version for CAGE 2)
```

---

## Step 1 — Setting Up the Environment

You begin by getting CybORG up and running, reusing what you already built in your RL project. CybORG is a gym (compatible with Gymnasium) for developing cyber defense agents and hosts various scenarios; the most suitable one to start with is **CAGE 2**, which involves defending a small enterprise network with a single defending agent (it is recommended to use CybORG++, a cleaned-up version that fixes several bugs from the original CAGE 2).

The key is understanding what exists in this world. The network consists of about thirteen hosts (workstations, servers, defense nodes) and accommodates three types of agents: **blue** (the defender, which is the one you are building), **red** (the attacker, which follows a predefined *script*, such as the B_line—direct—or Meander—exploratory—strategies, so you do not need to build it), and **green** (neutral users generating normal traffic). The attacker tries to compromise the network (scanning, exploiting, escalating privileges), and the defender tries to prevent it. Having this clear is the first step: the agent will live within this network, perceiving what happens and acting to defend it.

---

## Step 2 — Perception: Telemetry

What the agent *perceives* of the network is its telemetry, and understanding it well is key because it is what the perception layer in Phase 2 will process. In CAGE 2, the observation is a vector that, for each host, indicates two things: its **activity state** (None if nothing has happened, Scan if the attacker has scanned it, Exploit if it has been exploited) and its **compromise state** (no compromise, user access, or privileged access). In other words, a snapshot of what the attacker is doing and where they have gained access.

```python
# The Gymnasium loop (reused from your RL): the agent observes the network state
obs, info = env.reset()
# obs: for each host, its Activity state (None/Scan/Exploit) and Compromise state (No/User/Privileged)
```

And here is the first reality that will define the project, directly connecting to the approach you established: this perception is **partial and noisy**. Partial, because the agent does not see the full, true state of the network, but only a filtered view (similar to the partial observability of your robot and your own RL project). And noisy, because detection is not perfect: there are *false positives* (believing it sees an attack that does not exist) and *false negatives* (failing to detect a real attack). This "fog of war" is exactly why *anchoring* (grounding) matters so much in this project: the agent will have to reason and make decisions based on an incomplete and imperfect image of the world, without over-relying on it—exactly the grounding challenge you saw in your RAG and your robot. Imperfect perception is not a flaw in the environment; it is the realism of cybersecurity.

---

## Step 3 — Action: Defensive Actions with Consequences

What the agent can *do* are defensive actions, and understanding them (especially their cost) is the second key to this phase. In CAGE 2, the defender has actions such as **Monitor** (watch a host), **Analyse** (analyze a host's processes to detect compromises), **Remove** (remove an attacker's session from a host), **Restore** (restore a host to a known good state), and **Decoy** (deploy decoys to deceive and detect the attacker).

And here is the second reality that will shape the project: actions **have consequences and costs**. The environment's reward is negative (penalties), and the agent is penalized not only when the attacker gains access (especially to critical systems) but also when *the agent itself* takes disruptive actions: Restore, for example, cleanses a host but *disrupts* it, carrying an operational cost. This forces the agent to *balance* things: defend effectively, but without destroying the network to do so. Notice how important this is, as it is the seed of the bounded autonomy you set as your approach: in the real world, a defender's actions on systems have real consequences (you can crash a service, block legitimate users), so an agent acting without weighing the cost would be dangerous. From the very beginning, the environment teaches you that defending is not about acting without restraint, but acting with judgment.

---

## Step 4 — Manual Interaction Loop

With perception and action understood, you validate the basic loop: observe the state, act, and see the outcome. This is your first experience of the agent interacting with its world, and the equivalent, in this context, of your robot's perception-action loop:

```python
# The interaction loop (reused from your RL): observe, act, against the attacker
obs, info = env.reset()
for step in range(30):                       # one episode (fixed horizon, e.g., 30-100 steps)
    action = policy(obs)                     # for now, a simple (or random) policy
    obs, reward, terminated, truncated, info = env.step(action)
    # obs: the new state of the network after the action (and after the attacker's move)
    # reward: negative if the attacker gained access, or if the action was disruptive
    if terminated or truncated:
        break
```

Look closely at what you are doing, because it is the embryo of the entire agent: you observe the network state, perform a defensive action, and the environment advances (with the attacker also making a move), returning the new state and a reward. This cycle of observing and acting against a reacting adversary is the incident response loop in its simplest form. Right now, you are driving it with a simple policy; in the following phases, perception will turn into a rich understanding of the state (Phase 2), investigation will reason over the evidence (Phase 3), RL will decide actions (Phase 4), action will be bounded with safety (Phase 5), and everything will come together in the closed-loop agent (Phase 7). However, the essential mechanic (observing and acting against an attacker) is the one you have just touched upon.

---

## Step 5 — Testing

Since this phase is mainly about setting up and understanding the environment, verification consists of checking that everything works. Confirm that: the environment boots up with the CAGE 2 scenario; you obtain the observation (telemetry per host) and understand its fields; the agent can execute defensive actions; and the observe-act loop advances with the attacker moving. A simple test:

```python
def check_environment():
    obs, info = env.reset()
    assert obs is not None, "Observation (telemetry) was not obtained"
    # execute an action and verify that the environment advances and returns a reward
    obs, reward, terminated, truncated, info = env.step(test_action)
    assert reward is not None, "Reward was not obtained"
    print("Environment OK: perception (telemetry) and action (defense) are working")
```

This check confirms that your foundation (perception and action over the network) is functional, which is what you need to build the layers on top. It is the verification that the agent is properly situated in its world.

---

## Verification: The "Done" Criteria

The phase is complete when the following are met:

- [ ] You have set up CybORG with the CAGE 2 scenario (reusing your RL project), including the blue, red, and green agents.
- [ ] You understand perception: telemetry (activity and compromise states per host), and that it is partial and noisy.
- [ ] You understand action: defensive actions (Monitor, Analyse, Remove, Restore, Decoy) and that they carry a cost.
- [ ] You have validated the basic interaction loop (observe, act) with the attacker moving.
- [ ] **The key test:** You have a simulated network where the agent perceives telemetry and can act.

The key test is that of mastering the foundation: if you have the environment running and understand what the agent perceives and what it can do, you have the base on which all other layers will be built. Almost as importantly, you will have internalized the two realities that will shape the project (the fog of war of partial and noisy perception, and the cost of actions), which connect directly to the trustworthy autonomy approach: an agent that reasons over an uncertain picture and acts while weighing the consequences. It is not enough for the environment to simply start up; you must understand the uncertain world and the consequences within which the agent will operate.

---

## Deliverables and Next Steps

By closing Phase 1, you have the foundation of the project: a functioning environment with the attacker and defensive actions, along with a solid understanding of what the agent perceives (the partial and noisy telemetry) and what it can do (the defensive actions, with their costs). You have had your first contact with the agent's world, experienced the observe-act loop, and discovered the fog of war and the cost of action—two realities that connect with trustworthy autonomy. You have the agent well-situated in its world, ready to start equipping it with intelligence.

The next step, **Phase 2**, takes the first step toward that intelligence: **perception**. You will build the layer that converts raw telemetry (those per-host states you just saw) into a useful understanding of what is happening on the network (which threats exist and where), reusing the anomaly detection from your honeypot. It is the layer that *anchors* (grounds) the agent in what is truly happening, translating telemetry into something that can be investigated and acted upon. You have gone from "I have the environment and understand what the agent perceives and can do" to being on the verge of "the agent understands what is happening on the network."
