## Clash of Steels

**Clash of Steels** is a top-down 2D action game developed in **Unity (C#)**. Designed to emulate the precision and tactical intensity of *Sekiro*, it implements a physics-driven combat system with posture mechanics and weighted interactions. The project is also a testbed for **Lobster Framework**, a modular toolkit I created to streamline gameplay architecture, editor workflows, and runtime pooling in Unity.

This combination allowed me to explore a different style of gameplay engineering compared to my low-level C++ projects — this time leveraging Unity's ecosystem while still aiming for control and reusability in core systems.

## Demo
<video width="900" height="600" controls>
  <source src="../mp4s/Poise__Character_State_Manager.mp4" type="video/mp4">
</video>

<video width="900" height="600" controls>
  <source src="../mp4s/Offhand_Grab.mp4" type="video/mp4">
</video>

---

## Combat Mechanics

The game features a dual-resource combat system — **Health** and **Posture**. Both reduced on taking hits. 

If health is depleted, the character immediately dies.

If posture is broken, the character becomes immobilized, creating an opportunity for follow-up strikes. This introduces a rhythm of offensive and defensive maneuvers similar to *Sekiro* but implemented through continuous physics rather than scripted animations.

The character is able to perform the following actions:

- Weapon Switch (Left): Switch the offhand weapon
- Weapon Switch (Right): Switch the mainhand weapon
- Block: Raise weapon to enter defense stance
- Light Attack: Perform a quick attack with the mainhand weapon
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
- Blocking mitigates health loss and reduces posture damage based on the difference between the sum of self **character weight** plus self **weapon weight** and enemy **weapon weight**. The heavier the
- Attacks apply knockback based on **weapon weight**.
- If the character hits terrain (walls) at high velocity, posture damage will be taken
- **Character weight** acts as both a knockback resistance and a movement speed penalty.

Rather than relying on baked animations to simulate hit reactions, all impact responses are computed using Unity’s 2D physics engine. This makes weapon swings, knockbacks, and stuns feel more grounded and dynamically reactive.

---

## Evolution into Lobster Framework

**Clash of Steels** originally began as a standalone Unity project. As development progressed, I realized that many core gameplay systems — such as object pooling, runtime state management, and debugging tools — required significant time and effort to build. To improve efficiency and reduce duplication in future Unity projects, I began extracting these systems into a reusable module, which evolved into **Lobster Framework**.

This framework was born directly out of the needs of *Clash of Steels*, and many of its systems were first tested and validated in this game before being generalized into standalone tools.

🔗 [Explore Lobster Framework](https://dicespinner.github.io/LobsterFramework/)

---

### Design Goals

- Deliver the feel of weighty, reactive combat through real-time physics.
- Support clear feedback loops: posture break, slow-down effects, impact stun.
- Avoid animation lock-ins — use dynamic systems for movement and response.
- Establish reusable Unity systems for future projects via Lobster Framework.
