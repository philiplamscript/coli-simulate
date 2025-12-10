# Ranking Simulation Model (Guardian Tales Colosseum System Simulation)

![image](./GT_coli.jpg)

This repository contains Python code for simulating a competitive ranking system, specifically modeled after the **Guardian Tales Colosseum** feature. The system is designed to simulate how an entity's observable **score** is updated based on simulated matches against an opponent, capturing the key dynamics of the GT Colosseum, particularly the strategic element of opponent selection. The core of the simulation models the interaction between the observable score and a hidden **true score** (representing the entity's intrinsic skill level, or team power/composition).

The simulation runs over multiple rounds, during which entities are matched against opponents, and their scores are adjusted based on the match outcome and the underlying skill levels.

## Key Files

* `coli starting run.ipynb`: Contains the core utility functions for data creation, score manipulation, and the match (fight) logic.
* `coli s5.1.ipynb`: Contains the main simulation logic, data initialization, the primary 20-round simulation loop, and analysis/plotting.

## Simulation Mechanics

### 1. Core Match Logic (`coli starting run.ipynb`)

#### `coli_fight2(score1, score2, true_score1, true_score2)`

This function simulates a match (a "fight") between two entities and updates their scores:

1.  **Win Probability Calculation:** The probability ($\text{prob}$) of the attacker (entity 1) winning is calculated primarily based on the difference between their **true scores**.
    $$\text{prob} = \text{clip}\left(\frac{\text{true\_score}_1 - \text{true\_score}_2}{200} + 0.6, 0, 1\right)$$
    The result is clipped between 0 and 1, ensuring a valid probability. The formula gives the attacker a slight advantage (the $\mathbf{+0.6}$ offset) even if the true scores are equal.

2.  **Score Update (K-factor equivalent):**
    * **Attacker Wins:** Attacker's score increases by **+20**, Defender's score decreases by **-10**.
    * **Defender Wins:** Attacker's score decreases by **-10**, Defender's score increases by **+20**.
    *The absolute change in score is larger for the winner ($\mathbf{20}$ points) than for the loser ($\mathbf{10}$ points), meaning the overall score pool increases with every match.*

### 2. Simulation Logic and Data (`coli s5.1.ipynb`)

#### Data Initialization

The simulation uses a Pandas DataFrame named `score_borad`, initialized with **5040 entities**. Each entity has several attributes:

| Column | Description |
| :--- | :--- |
| `score` | The entity's current, observable ranking score (Colosseum Rank Point). |
| `true score` | The entity's intrinsic, hidden skill level (e.g., actual team power/strength). |
| `int_score` | The initial starting score of the entity. |
| `type` | A category for the entity (e.g., 0, 1, 2, 3), potentially representing different meta strategies or player behaviors. |
| `play_time` | The maximum number of matches the entity is allowed to attack in. |
| `att count` | The number of times the entity has attacked/played. |
| `def count` | The number of times the entity has defended/been played against. |
| `att win` | Count of attack wins. |
| `def win` | Count of defense wins. |
| `skiped time` | Number of rounds the entity was skipped due to type conditions. |
| `scorediff` | The final change in score (`score` - `int_score`). |

#### Simulation Loop (20 Rounds)

The simulation runs for **20 iterations (rounds)**. In each round:

1.  The entities are shuffled to randomize the attacking order.
2.  Each entity that has not reached its `play_time` limit is considered as an attacker (`id_1`).

3.  **Opponent Selection (Matchmaking Bias - GT Strategy):**
    * The attacker first finds opponents whose current `score` is **closest** to their own score (similar to the pool of opponents displayed in Colosseum).
    * From the top two closest opponents, the attacker selects the one with the **minimum `true score`**.
    *This mechanism simulates the common Colosseum strategy of **score-matching but strength-exploiting**, where players target opponents who appear to be within their rank bracket but have demonstrably lower actual team strength (`true score`).*

4.  **Type-Specific Rule:**
    * **Type 2 Entities** are specifically excluded from participation (their attack attempt is skipped) during the first 5 rounds of the simulation (`i < 5`). This simulates a late entry or period of inactivity.

#### Output and Analysis

After 20 rounds, the final score change (`scorediff`) is calculated for all entities, and the notebook generates pivot tables and plots to analyze how the `type` and `true score` influence the final `score` and overall performance.