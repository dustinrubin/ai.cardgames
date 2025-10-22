---
layout: post
title: "Building a RL Environment for The Fox in the Forest"
date: 2025-10-20
---
# The Fox in the Forest 
The Fox in the Forest is a well rated 2 player trick taking card game. In technical terms, it is a 2 player zero sum Partially Observable Stochastic Game (POSG) and the solution concept is to find a nash equalibrim. It has elements of bluffing and cards with special effects similar to Trading Card Games (TCG). It is much simpler than TCG with a small fixed card pool. This will be a good game to test ai assisted coding and Deep Renforcemnt Leaning (DRL) methods before moving on to more complex TCGs. This ariticle will talk about the coding and creation of the environment and consider how this might be expanded to more complex games.  


## The Environment Problem in TCG AI

One of the most difficult parts of AI for TCGs is the environment. For TCGs with hundreds or even tens of thousands of cards that have unique effects but can share many common elements, manually coding every card is prohibitively expensive. Even for a skilled developer manually creating an environment for all cards is a time consuming process. 


## AI-Assisted Development + TDD + Rust

Recent advances in AI-assisted coding make it possible to automate the coding of complex game environments piece by piece. 

The core idea of automating software development is extremely powerful, and this problem has many characteristics that make it more achievable:

- **Structured rules** - Card effects follow patterns Large Langage Model (LLM) based AI agnets can understand or search the internet for clarificaiton
- **Verifiable correctness** - AI generated tests can be verified by (LLM) and checked by humans in isolation
- **Incremental complexity** - Cards can be implemented one at a time and implmenations verified in isolation 

Test-Driven Development (TDD) is crucial for this approach. LLM agents can verify their implementations and compiler correctness by running tests, making errors easily detectable for both the agent and the programmer. The workflow becomes:

For each new mechnica or card 
1. LLM agent generates test 
2. LLM agent generates implementation
3. Compiler + tests verify correctness
4. Iterate until all tests pass

For coding this game claude code with sonnet 4.5 was used along with a spec describing the goals of the enviroment. A brief review after each step was done to check direction and for each of the cards with unique rules the process above was used. It took only a few hours of iteration with calude code with limited supervision and revision requests. It seems that currently TCG egnines lend themselves well to ai asssited coding requiring limited human intervention. This is a valuable insight that may be mean that for even complex TCG like Magic the Gathering (MTG) AI assisted coding may be able to automate large parts of the engine creation. 

## Why Rust?

Rust may be an initially suprizing programming language but I think strongly that it is the best choise for a few reasons. 

- **Performance of a systems language** - Comparable to C/C++, targeting 10,000+ games/second
- **Strict compiler feedback** - Type errors and explictiness give the LLM clear error messages during TDD iteration and make it harder to have incorrect code
- **Simpler than C++** - Doesn't have years of history and udpates and thus is generally simpler and more modern 

The traditional disadvantage of Rust is its complexity, but AI-assisted coding makes it far simpler to create.  

## Why The Fox in the Forest?

This environment is quite simple and is a proof of concept to demonstrate the core ideas that will expand to other card games.

### Game Mechanics

The game involves racing to a score of 21 points, players try to win tricks to score points, but winning too many tricks results in zero points due to a "greediness penalty." This non-monotonic scoring creates  strategic depth and bluffing potential. A "trick" involved one player playing a any card from their hand then the other player must follow suit if able. 

| Tricks Won | Points | Description        |
|------------|--------|--------------------|
| 0-3        | 6      | Humble             |
| 4-5        | 1-2    | Defeated           |
| 6          | 3      |                    |
| 7-9        | 6      | Victorious         |
| 10-13      | 0      | Greedy (too many!) |

Special cards have unique abilities:
- **Swan (1)** - If you lose the trick, you lead next
- **Fox (3)** - Exchange the decree card to change trump suit
- **Woodcutter (5)** - Draw a card, then discard one
- **Witch (9)** - Becomes trump if it's the only 9 in the trick
- **Monarch (11)** - Opponent must play 1 or their highest card in the suit

These special abilities are similar to TCG where some cards will have unique effects and agents must learn these effects to play as best as possible

See the offical rules for more details. 

The Fox in the Forest also has advantages for RL training:

- **Small enough to train quickly** - 33 cards vs 1,000+ in games like Lorcana or Magic the Gathering
- **Complex in ways simmilar to TCG** - Special abilities, imperfect information
- **Tests the same patterns needed for full TCGs** - Hidden information, game strucutre, action masking

## The Core API Design

The high-level interface is designed for reinforcement learning and an additoanl gym enviroment wrapper can be added aroudn this. At every decision point, the agent receives a list of legal actions. The engine exposes a python api for use with RL libraries. 

```python
import foxintheforest as fox
import random

state = fox.GameState.new_game()
engine = fox.GameEngine()

while not state.is_game_over():
    legal_actions = engine.get_legal_actions(state)

    action = random.choice(legal_actions)
    engine.apply_action(state, action)

print(f"Winner: Player {state.winner()}")
```

### Key Design Principles

1. **Stateless engine** - GameEngine contains no state, only rules for a clean seperation of duties.
2. **Mutable game state** - GameState is a pure data structure that can be cheaply cloned for search algorithms.
3. **Action masking** - Only legal actions are provided, preventing the agent from trying illegal moves. This will be critical for more complex TCG's. 
4. **Staged actions for complex effects** - For TCGs where a single action requires multiple choices (e.g., "Draw 3 cards, then discard 2"), staged actions prevent combinatorial explosion of the action space and will hopefull allow agents to generalize and understand actions better. 

## Player Perspective & Observations

The design explicitly handles the two-player perspective problem though limited observations.

```python
import foxintheforest as fox

state = fox.GameState.new_game()
engine = fox.GameEngine()

agents = [MyPPOAgent(), MyPPOAgent()]

while not state.is_game_over():
    current_player = state.current_player

    obs = state.get_observable_state(current_player)
    encoding = obs.encode()  # shape: (111,), dtype: float32

]   legal_actions = engine.get_legal_actions(state)
    action = agents[current_player].get_action(encoding, legal_actions)

    print(f"Player {current_player} plays {action}")
    engine.apply_action(state, action)

print(f"Winner: Player {state.winner()}")
```

get_observable_state(player_index) Returns only information visible to that player

The agent is not told what card abilities do. For example, when the opponent plays a Fox (rank 3) and changes the trump suit, the agent must learn this pattern by observing:

- `obs[t-1].decree_card = Bells` (trump before action)
- Action: opponent plays a 3
- `obs[t].decree_card = Keys` (trump after action)

A neural network must learn that, "When a 3 is played, the trump suit often changes." This forces the agent to discover card abilities through experience.


## What the Agent Sees: The 111-Feature Encoding

The observation encoding is a flat array of 111 float32 values:

- **Player HAND [0-32]** - Binary one-hot encoding of which cards player holds
- **OPPONENT HAND SIZE [33]** - Normalized count
- **TRUMP SUIT [34-36]** - One-hot encoding (Bells/Keys/Moons)
- **CURRENT TRICK - Player CARD [37-69]** - Binary encoding of card player played (if any)
- **CURRENT TRICK - OPPONENT CARD [70-102]** - Binary encoding of card opponent played (if any)
- **TRICKS WON [103-104]** - Normalized counts for both players
- **SCORES [105-106]** - Normalized by 30 (Note ties above 21 can cause higher scores)
- **DECK SIZE [107]** - Normalized count (May be removed)
- **ROUND NUMBER [108]** - Normalized by 10
- **Is Players Turn [109]** - Binary flag
- **Is Player Dealer [110]** - Binary flag

In more compelx TCG a one hot encoding of cards will not make sense and an embedding space will be required. 


## Determinization for Search Algorithms

This design is fast enough for search-based methods like MCTS or AlphaZero (targeting 10,000+ games/second). However, search in imperfect information games requires special handling.

### The Problem

When running MCTS, we need to simulate future game states, but we don't know the opponent's hidden cards. If we just use the "true" hidden information during search, the agent would be cheating.

## Validation & Performance

The test suite that can be checked by a human from TDD includes:

- **Core mechanics tests** - Trick-taking, following suit, trump resolution, scoring
- **Ability tests** - All 6 special card abilities (Fox, Woodcutter, Witch, Swan, Treasure, Monarch)
- **History tests** - Action sequence tracking for temporal learning
- **Observability tests** - Verifying hidden information is properly filtered
- **Encoding tests** - Validating the 111-feature representation

For perforamnce the Rust implmenation is blazeingly fast. I ran this on my M1 2020 macbook air for 10 million games with agents that always choose a random action. 

```
⏱️  Time & Throughput:
   Wall time:          138.31s
   Games completed:    10000000
   Games/second:       72301.1

📊 Game Statistics (averages):
   Actions per game:   117.1
   Rounds per game:    3.86
   Duration per game:  0.000s

🏆 Win Distribution:
   Player 0 wins:      4997506 (50.0%)
   Player 1 wins:      5002494 (50.0%)

⚡ Performance Metrics:
   Actions/second:     8466160
   μs per action:      0.12
   μs per game:        13.83
```

Even with multiple rounds each game is almost instant and it is achieving over 70,000 games per second.


## Next Steps: Training RL Agents

With the basic of the environment complete, the next phase is starting to train reinforcement learning agents

---

*Next post: Training PPO agents and analyzing what strategies they discover*
