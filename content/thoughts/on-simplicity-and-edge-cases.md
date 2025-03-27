+++
title = 'On Simplicity and Edge Cases'
date = 2025-03-27T12:09:35-07:00
draft = false
toc = false
+++

Questions to ask:
- What’s the dumbest, most naive, most brute-force approach that could possibly work?
- What are the positive qualities of that approach?
- What are the downsides? Is it slow? Expensive? Does it miss important edge cases?

Take these and compare them to our current state. Are the `positive qualities - downsides` of the naive approach better than where we are today? If so, you might have a good first version!

If not, why not? Focus on those problems and consider approaches that address them.

## Explicit > Implicit

Very frequently, *explicit* approaches beat out implicit ones for simplicity.

In an example we talked through today, we considered a system that already has a timer component that sets something up based on user-provided configuration behind the scenes every 10 minutes. In the worst case, that means after a user provides new configuration, it will take up to 10 minutes for the system to respond to that config and set it up.

We considered two options for a user-facing workflow:

1. Add APIs to enable doing that setup on demand and call those from the workflow when the user creates their config.  
2. Add a step to the workflow that sleeps for 10 minutes to ensure the system has completed setup before proceeding to the next step.

Which option is simpler?

Option 2 seems to require less engineering effort upfront. We already have the system timer component. Adding a sleep to the workflow is pretty trivial. It comes at the cost of a slower setup time, but that might be the right tradeoff.

Option 1 takes more upfront work, since we don’t have those APIs yet.

But Option 1 is *explicit*, which has major benefits.

Imagine you're a new engineer working on this system (or just yourself a year from now). You see an "Activate" button in the UI and need to change its behavior somehow. You trace it to the API call, which saves some configuration and then launches a workflow. That workflow contains a series of steps.

Which workflow will you be able to understand and change more easily? The one that has a ten-minute sleep? Or the one that makes two API calls?

## Edge Cases

Brainstorming edge cases, race conditions, and problems is valuable and worth doing upfront! But having them does *not* make the solution bad. Edge cases represent risks, which are tradeoffs we can make.

`edge case risk = probability of it occurring * impact of the occurrence`

A very small possibility of customer data deletion or a security breach, for example, is a high edge case risk—because even though the probability may be minuscule, the impact is astronomical.

Edge cases are barely worth talking about without the context of those two factors: the probability and the impact. The probability may be hard to derive, but we should at least give it a shot. We should also be able to reason about the likely impact.

When we don’t, we wind up with problems like “but there might be inconsistencies!” when no one actually cares. This can snowball into over-engineering and drive us into a place where even small changes become impossible because we’re paralyzed by the possible outcomes.

## Mitigating Edge Cases

When we understand edge cases, we can mitigate them to reduce their risk. This does *not* require fully addressing them.

Mitigation might include:
- Documenting them in a runbook so that an on-call dealing with a live issue can consider them  
- Instrumenting them to track when they actually occur. This can take our `probability of it occurring` value from a guesstimate to a known quantity (and maybe we were wrong and need to deal with it!)
- Defining their behavior — maybe it’s not behavior we *want*, but it’s known (e.g. throwing an error with a meaningful message!). If you can define their behavior in a unit test, that’s fantastic.
