---
title: Building a working Action AI in Unreal
date: 2025-07-29
tags: [gamedev, software, profiling, indie]
---

I---
title: Template Action AI in Unreal
description: A guide to create a functional action AI template in Unreal
date: 2025-07-29
tags: [unreal, ai, gamedev]
---

During my adventure learning Unreal and shipping my first game, I had more problems than I care to admit building reliable AI in Unreal; even when what I wanted was a very simple and common behavior: patrol some path, go investigate when a noise is heard and chase when a target is acquired.

I believe partly it’s due to the **Unreal AI Perception system** being a bit cryptic and counterintuitive. And partly, because I didn’t fully understand the tools and was trying to create transitions the wrong way.

Eventually, I figured out a decent way to go about it. Let me share a few ideas here.

## What I Want to Build
At its core, I want an agent that can switch between 3 states:

- **Default**: when the agent has no task
- **Investigate**: move to last heard location
- **Chase**: follow player — or attack, or anything you define

---

### Transition Rules

Let’s break it down.

- `DEFAULT → INVESTIGATE`: We have no target, but we heard a sound.
- `INVESTIGATE → CHASE`: We now have a valid target.
- `CHASE → INVESTIGATE`: We lost the target for more than X seconds.
- `INVESTIGATE → DEFAULT`: Investigation completed without reacquiring the target.

All transitions are based on **Perception** updates and need to happen at the right time. I find it easier to model this using State Trees because they are built around the concept of a state.
You can achieve the same exact thing with Behavior Trees (after all, you can use BTs as State Machines) but that is the path you take if you really want to complicate your life. Because Behavior Trees are reactive it is harder to define when and how a transition should take place.

State Trees allow you to define authoritative transition logic: when this happens, go to this state.

{% callout "info", "Learn the tools" %}
The tricky part is not implementing the states themselves, but **when and how** to transition between them.
{% endcallout %}
---

## SIGHT: Gaining and Forgetting a Target

The system we defined is basically revolving around answering these 2 questions, as the agent:

1. **Did we acquire a new target?**
2. **Have we lost the target long enough to forget it?**

**Acquiring** is simple: if we see someone and we’re not already tracking them — boom, we have a new target.

**Forgetting**, however, is trickier. Unreal **does not** send an event when a sense times out.

{% callout "warning", "Unreal won’t tell you when it forgets a target" %}
You can set MaxAge all you want: it is a catch-all property and will only trigger an event when the perception component has no more valid sense data (that is, for example: if you ha no sight info but still hearing info cached for the same actor, the event will not trigger. ).

{% endcallout %}

### The Fix: A Forget Timer

We solve this by creating our own aging system:

1. When sight is lost, we start a **timer**.
2. If the timer expires without reacquiring the target, we forget them.
3. If the target is seen again, we **reset the timer**.

---

## HEARING: Simpler, but Still Useful

Every time a sound is heard, we store its location.

In this setup, we only care about the **last** sound, so a single variable will do.

---

## Implementation

So how do we actually build this?

The core of the system lives in a **State Tree Evaluator**, which does the following:

- Registers for **perception updates**
- Tracks the current **Target** and **LastHeardLocation**
- Sends **custom events** to trigger state transitions
- Starts and resets the **forget timer**

---

### Sight Stimulus Flow

Here’s how we handle sight updates:

{% callout "tip", "Sight logic: what happens when perception updates?" %}
1. If we *had a target* but lost it: start forget timer.
2. If we *didn’t have a target* and now we see one:
   - If it's the same as the one we just lost: stop the forget timer.
   - If it's new: send event, update the target.
{% endcallout %}

---

### Hearing Stimulus Flow

Hearing is much simpler:

- We just update `LastHeardLocation` with each new sound.

---

## Showcase

Here’s what the final result looks like in Unreal:

<img src="./ai_state_tree_showcase.jpg" alt="Screenshot of AI State Tree and transitions in Unreal.">

---

## Final Thoughts

This setup isn’t complex, but clarity in transitions is key.

If you're struggling with Unreal’s Perception system — you’re not alone. But with a custom evaluator and a simple forget timer, you can build surprisingly flexible AI logic.

{% callout "note", "Want more like this?" %}
I’m planning to do a follow-up on adding **attack and retreat** logic next. If you’re interested, follow me on [YouTube](https://youtube.com/) or [Twitter](https://twitter.com/).
{% endcallout %}