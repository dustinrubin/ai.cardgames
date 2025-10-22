---
layout: post
title: "Building AI Agents for Trading Card Games"
date: 2025-10-19
---

I'm starting a blog to document my work building AI engines that can play trading card games.

Trading card games like Magic: The Gathering (MTG) and Hearthstone present one of the hardest challenges in game AI. They combine imperfect information (hidden hands), massive action spaces (hundreds of possible plays each turn), and ever-growing complexity (each new card can introduce unique mechanics). Current approaches to evaluating cards and strategies rely on population statistics from online trackers or expensive multi-day expert playtesting sessions. An AI that could play at expert level would give designers and players a powerful new tool for understanding these games.

The technical challenges of this problem are:

- **Imperfect information** - Every action reveals information about your hand and strategy, while you're simultaneously trying to infer your opponent's plans
- **Combinatorial explosion** - The state space and action space grow massively with each card added to the pool
- **Complex rules engines** - Most TCGs don't have public rules engines built for the kind of fast simulation AI training requires

## What I'll be exploring on this blog

I'm particularly interested in three areas:

1. **AI-assisted coding for engine development** - Using autonomous agents and ai assisted coding to rapidly build game engines for complex TCGs, rather than hand-coding every card to allow building for games with 10 of thousands of cards like MTG.
2. **Deep reinforcement learning** - Experimenting with DRL and search based approches like AlphaZero other DRL methods to create strong agents that can learn strategy from self-play. This is Multi Agent Reinforcement Leanring (MARL)
3. **Advanced search methods** - Testing different algorithmic approaches for handling imperfect information and huge action spaces

My plan is to start with simpler card games and developer engines and progressively tackle more complex ones, documenting what works, what doesn't, and what I learn along the way.
