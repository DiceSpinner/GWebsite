## Clash of Steels

**Clash of Steels** is a top-down 2D action game developed in **Unity (C#)**. Designed to emulate the precision and tactical intensity of *Sekiro*, it implements a physics-driven combat system with posture mechanics and weighted interactions. The project is also a testbed for **Lobster Framework**, a modular toolkit I created to streamline gameplay architecture, editor workflows, and runtime pooling in Unity. Currently I'm using Unity built-in source control so only the framework has the source code on [github](https://github.com/DiceSpinner/LobsterFramework). The documentation of the framework currently undergoing development can be found [here](https://dicespinner.github.io/LobsterFramework/).

This combination allowed me to explore a different style of gameplay engineering compared to my low-level C++ projects — this time leveraging Unity's ecosystem while still aiming for control and reusability in core systems.

## Current Development Status

The game is still under active development. Major focus so far has been on building the core combat mechanics, physics interactions, and tooling infrastructure. While the foundational systems are functional — including the combat engine, ability system, and AI state machine architecture — aspects such as enemy logic, level content, and full game progression are still being prototyped and iterated.

The project is currently in a pre-alpha state with a working testbed for real-time physics-based combat. The next milestone is to build a compelling boss fight and then construct a level around it to test and validate all core gameplay systems. Once these systems are proven in context, future effort will shift toward game design, visuals, and audio integration.

---

## Demo

<video width="900" height="600" controls>
  <source src="../mp4s/Poise__Character_State_Manager.mp4" type="video/mp4">
</video>

<video width="900" height="600" controls>
  <source src="../mp4s/Offhand_Grab.mp4" type="video/mp4">
</video>

---

## Design Goals

- Deliver the feel of weighty, reactive combat through real-time physics.
- Support clear feedback loops: posture break, slow-down effects, impact stun.
- Avoid animation lock-ins — use dynamic systems for movement and response.
- Establish reusable Unity systems for future projects via Lobster Framework.

---

## Game World & Progression

The game is currently structured as a single-player, linear experience. Each level is hand-designed and features its own unique set of enemies and resources for the player to gather. To complete the game, the player must progress through all levels in sequence.

Boss encounters serve as the primary focus in each level. Every stage will contain one or more boss fights that challenge the player's mastery of the combat system. These encounters are intended to be the culmination of that level's mechanics and enemy design.

Level design, enemy layouts, and exploration structure are still under active development, as the majority of effort so far has been devoted to implementing and refining the core combat mechanics.

---

## Combat Mechanics

The game employs a layered combat resource system composed of three interrelated components:

- **Health**: The primary life resource. If health is depleted, the character immediately dies.

- **Posture**: A visible stamina-like value that determines a character's defensive stability. When posture is broken:

    - The character becomes immobilized for about 2–3 seconds.
    - Posture is restored to 70% afterward.
    - It gradually regenerates over time if the character avoids taking hits and is not attacking.

- **Poise**: A hidden internal value used to interrupt actions.

    - Damaged whenever health damage is taken.
    - Causes brief vulnerability but recovers fully after a couple of seconds without receiving additional health damage.

This system encourages a balance between aggression and defense, and introduces a reactive flow to combat through layered stamina-like mechanics.

This introduces a rhythm of offensive and defensive maneuvers similar to *Sekiro* but implemented through continuous physics rather than scripted animations.

The character is able to perform the following actions:

- Weapon Switch (Left): Switch the offhand weapon
- Weapon Switch (Right): Switch the mainhand weapon
- Block: Raise weapon to enter defense stance
- Light Attack Combo: Perform a sequence of light attacks with the mainhand weapon. The specific moveset is determined by the weapon's assigned animation.
- Charged Attack: Perform a charged attack with the mainhand weapon
    - Can hold to charge and increase damage
    - Has max charge time, will attack automatically when fully charged
- Weapon Art (Weapon dependant)
    - Perform the equipped weapon art of the current mainhand weapon
- 8 Directional Move
- 8 Directional Dash
- Rotate view, used in conjuection with move and dash for omni-directional movement

The effect of actions:

- Attacks will swing the weapon equipped in main(right) hand and will reduce the enemy posture and health on hit
    - Health damage is determined by **weapon sharpness**
    - Posture damage is determined by **weapon weight**
- Blocking mitigates health loss and reduces posture damage based on the weight dynamics between attacker and defender. The heavier the enemy's weapon, the more posture damage it inflicts. Conversely, the greater the combined weight of the character and their blocking weapon, the less posture damage they receive.
- Knockback force is computed from the actual posture damage taken. As a result, if an attack is blocked and less posture damage is incurred, the resulting knockback will also be reduced.
- If the character hits terrain (walls) at high velocity, posture damage will be taken
- **Character weight** acts as both a knockback resistance and a movement speed penalty.

Rather than relying on baked animations to simulate hit reactions, all impact responses are computed using Unity’s 2D physics engine. This makes weapon swings, knockbacks, and stuns feel more grounded and dynamically reactive.

---

## Evolution into Lobster Framework

**Clash of Steels** originally began as a standalone Unity project. As development progressed, I realized that many core gameplay systems — including an ability system for managing character actions and casting, AI state machines, interaction handlers, and custom initialization utilities — required significant time and effort to build. Both the ability system and AI state machine include built-in serialization support and come with custom Unity editor interfaces for easier authoring and inspection. These systems also introduced custom engine events and were built on top of Unity to provide more flexibility and modularity. To improve efficiency and reduce duplication in future Unity projects, I began extracting these features into a reusable module, which evolved into **Lobster Framework**.

This framework was born directly out of the needs of *Clash of Steels*, and many of its systems were first tested and validated in this game before being generalized into standalone tools.

🔗 [Explore Lobster Framework](https://dicespinner.github.io/LobsterFramework/)
