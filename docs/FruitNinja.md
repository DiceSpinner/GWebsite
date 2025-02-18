# [Fruit Ninja](https://github.com/DiceSpinner/FruitNinja)
As the title says, this is a clone for the infamous [Fruit Ninja](https://en.wikipedia.org/wiki/Fruit_Ninja) game, developed in C++ with OpenGL for graphics and OpenAL for audios. This project mainly serves as a mean of self education on how things work at lower level. Some of the tasks include configuring the build system using visual studio, CMake with vcpkg package manager, setting up OpenGL to communicate with the GPU, rendering meshes, UI images, fonts and particles in an efficient manner and finally implementing gameplay logics using optimized approaches like object pooling, safe memory management with smart pointers and providing easy ways of extending behaviors using object oriented design and templates. This project is planned to be continued for at least a while to satisfy my ever expanding learning goal.

## What's Implemented
### Fundamentals
- Texture/Model/Audio Loading
- Component based gameplay architecture
- Model rendering
    - Phong shading
- UI rendering
    - Image rendering
    - Batched font rendering
- Instanced particle rendering
    - Billboard
- Outline rendering by stencil test
- Object Pooling
- Rigidbody physics
- Slow/Pause/Speed up time
- OpenAL Integration
- Cursor picking
### Game
- The original classic game mode
    - 3 lives maximum, lose 1 life for every fruit miss, recover 1 life on every 50 fruit slices
    - Spawns random fruits in random periods and amounts
    - The game keeps on until all lives are lost
    - Bomb
        - Explosion audio
        - Outline rendering (Stencil Test)
        - Game over if sliced
- Fruit slice in half
- Fruit Slice particle effect
- Fruit slice audio
- Start Menu music
- Fruit deploy audio
- Life loss/recover audio
- Game Start/Over Audio
- Mouse swipe trailing vfx

## Current State
<video width="900" height="600" controls>
  <source src="../mp4s/fruit_ninja.mp4" type="video/mp4">
</video>

## Will Be Implemented
### Game
- Bonus score for hitting multiple fruits
- Critical strike
- Fruit Slice texture animation on the background
- Bomb
    - Explosion effect
  
## Under Consideration
### Game
- Multiplayer, similar to the original local player where 2 players compete to survive the longest and can throw bombs at each other.
    - Idea: Slicing fruit fills energy bar, both player can spend energy to throw bombs at each other or spawn super fruit that provides special effects.