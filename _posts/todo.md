### The Solution: Determinization

Determinization solves this by randomly sampling possible opponent hands that are consistent with what we've observed so far:

1. Sample a random valid opponent hand (cards they could plausibly have, given what's been played)
2. Search assuming that's their actual hand
3. Average results across multiple sampled determinizations

This approach ensures the agent can only use information it should actually know, while still being able to plan ahead.

Future enhancement: Instead of uniform random sampling, we could use the agent's belief distribution about what the opponent likely holds, based on their play patterns. This would make the search more focused on realistic scenarios.