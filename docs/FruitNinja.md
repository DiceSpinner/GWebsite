# Fruit Ninja 
As the title says, this is a clone for the infamous [Fruit Ninja](https://en.wikipedia.org/wiki/Fruit_Ninja) game, developed in C++ with OpenGL for graphics and OpenAL for audios. This project mainly serves to educate myself on how things work at lower level. This is my first fully owned C++ project. The first thing I learned is to setup the build system using cmake with vcpkg package manager to manage the libraries needed for various tasks. The libraries themselves are relatively easy to find with vcpkg except for OpenAL where I have to fiddle around with the version and build settings to make sure the utilities are installed and the debug extension is enabled. I was following this [tutorial](https://learnopengl.com/Getting-started/OpenGL) and managed to implement rendering which handles meshes, UI images, fonts and particles in an relatively efficient manner. The foundation of the gameplay is similar to Unity. I managed to implement 3D transform and component object architecture as well as some utilities using templates and object oriented design. Finally the gameplay logics are implemented on top and I learned to use various C++ utilities to do various optimizations like object pooling to handle memory management. I have to limited to the scope of this project to be the list of features down below, but there's a good chance that I may want to come back and expand the scope. The current idea for expansion I have in mind is mulitplayer which could be a good chance for me to be proficient at multithreaded and socket programming.

The source code can be found on the [github](https://github.com/DiceSpinner/FruitNinja).

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
- Coroutine
    - Game State Transition
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
        - Explosion visual effect and audio
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
  
## Under Consideration
### Game
- Multiplayer, similar to the original local player where 2 players compete to survive the longest and can throw bombs at each other.
    - Idea: Slicing fruit fills energy bar, both player can spend energy to throw bombs at each other or spawn super fruit that provides special effects.