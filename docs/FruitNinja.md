# Introduction

This project is a custom-made Fruit Ninja clone built entirely in **C++ and OpenGL**. It began as a deep dive into the inner workings of real-time rendering, driven by a desire to understand what actually happens beneath the hood of game engines like Unity — which I had previously used extensively.

Rather than relying on pre-built components and black-box systems, I wanted full control over the game loop, the rendering pipeline, memory management, and input handling. Rebuilding familiar gameplay from scratch also provided an ideal playground for experimenting with graphics techniques, system-level architecture, and hands-on performance tuning — all while strengthening my core C++ skills.

What started as a curiosity has grown into a fully playable classic mode, complete with polished visuals, real-time audio, and responsive controls.

The source code with build instructions can be found [here](https://github.com/DiceSpinner/FruitNinja/tree/main)

# Demo
<video width="900" height="600" controls>
  <source src="../mp4s/fruit_ninja.mp4" type="video/mp4">
</video>

# What is Fruit Ninja?

Fruit Ninja is a popular arcade-style game first released in 2010. The gameplay is simple yet addictive: fruits are tossed into the air, and players must slice them by swiping across the screen. 

- Slicing multiple fruits in one swipe yields combo bonuses.
- Accidentally slicing bombs or missing too many fruits results in a game over.

Its intuitive controls, visual feedback, and satisfying pace made it a mobile gaming classic.

Recreating this experience from scratch — without the help of any game engine — posed both a technical challenge and an opportunity to explore core concepts like object transformations, collision detection, resource management, and GPU-accelerated rendering.

# Technical Architecture

## Object Model

The object system in this project draws inspiration from the Unity game engine. Instead of using deep inheritance hierarchies, complex behaviors are composed through aggregation.

Each game object is built from a collection of modular components, with each component representing a distinct behavior or system responsibility (e.g., rendering, physics, audio).

During both the physics update and per-frame update stages of the game loop, each component attached to an object is allowed to run its own logic independently. This design improves flexibility and code reusability, as functionality can be mixed and matched without rigid class structures. It also aligns with data-oriented practices by keeping systems decoupled and modular.

Active game objects are stored as `shared_ptr` instances in a protected list that is traversed during the update loop. Only objects in this list are updated each frame. When an object becomes inactive, it is removed from the update cycle but can be reactivated later. Upon such state transitions, the components of the object receive appropriate activation or deactivation callbacks.

This mechanism is essential to the game state system, which is implemented as a finite state machine. For example, in the in-game state, UI interactions use real 3D objects that persist between states. Avoiding object reconstruction improves efficiency and keeps behavior consistent across game modes.

## Game Loop (Single Player Mode)

The single-player game loop follows a structured sequence to ensure responsive gameplay and consistent state updates. The loop continues until the window is closed, and each frame processes the following stages in order:

1. **Time and Input**: Time is updated and user input is processed through GLFW. Mouse actions are translated into slice gestures.
2. **UI Background Rendering**: Depth testing is disabled, and the back UI layer is rendered first using an orthographic projection matrix.
3. **Object Lifecycle Management**: Newly activated objects are registered before simulation updates begin.
4. **Physics Simulation**: If the fixed timestep has elapsed, the engine performs early and regular fixed updates on all game objects. These updates primarily involve positional adjustments for rigidbodies, particles, and audio sources to ensure smooth motion and synchronized spatial effects.
5. **Per-Frame Updates**: All game objects receive time-based update calls to simulate their behavior and respond to the current game state. This mostly involves checking input-related conditions and updating the status of interactive objects like fruits and bombs. After these updates, the main game logic is executed through a finite state machine. The currently active game state object runs its logic, which may include transitioning to another state.
6. **3D Object Rendering**: Depth testing is re-enabled. The main 3D scene is rendered with lighting, using shaders configured with view/projection matrices and light properties.
7. **Visual Effects**: Depth writes are disabled. Post-process VFX and particle systems are rendered. Particle systems are drawn using instanced rendering based on camera-aligned billboards.
8. **UI Foreground Rendering**: UI is rendered again, this time for overlay elements like score or debug text, using the same shader and projection as the background UI pass.
9. **Frame Presentation**: The front and back buffers are swapped to display the rendered frame.

This loop structure separates UI and gameplay rendering, ensures clean ordering of updates, and preserves performance by limiting physics to fixed intervals while rendering remains as fast as possible.

## Rendering Engine

The rendering system is built on top of modern OpenGL, following the programmable pipeline model. To guide the foundational design, I referenced the [LearnOpenGL](https://learnopengl.com/Getting-started/OpenGL) tutorials, progressively building custom functionality on top of the concepts covered.

At its core, the engine loads 3D models and textures using the **Assimp** library, which parses standard model formats and extracts mesh, material, and texture data. These resources are managed using **RAII**: they are loaded upon initialization and automatically released upon destruction. The parsed data is uploaded to the GPU using **vertex buffer objects (VBOs)** and **vertex array objects (VAOs)**. Shaders written in **GLSL** handle lighting, texture sampling, and screen-space effects.

Before diving into how different types of objects are rendered, it's important to understand the overall structure of the game loop. Each frame, the game loop updates the physics and object positions before proceeding to the rendering phase. The renderer for each object type then uses the updated positional data, typically passed as uniforms, to accurately display the objects.

### Regular Objects

Phong shading is applied for most game objects such as fruits, bombs, and background props. The vertex shader receives transformation matrices via uniform variables and computes the final vertex positions. The fragment shader calculates per-pixel lighting using both light positions and material data, all passed in as uniforms.

To ensure visual consistency on low-poly models, the diffuse value sampled from textures is clamped to be no less than 0.5. This adjustment helps mitigate harsh lighting transitions, which are more noticeable due to the single light source and the flat surfaces of low-poly geometry.

Each object is rendered individually due to differences in geometry and material. An additional option is provided to clear the depth buffer before each draw call. This is particularly useful for gameplay elements like fruits, which are spawned on the same plane and can otherwise cause depth fighting or rendering artifacts when overlapping.

The renderer also supports optional object outlines. If enabled, each object is rendered twice:

- **First pass**: The object is rendered normally, but the stencil buffer is cleared and marked for each rendered pixel.
- **Second pass**: A simpler outline shader program is used. The object's transformation matrix is scaled by 110%, and the vertex shader uses this enlarged geometry. The fragment shader outputs a solid color and discards any pixels marked inside the stencil buffer, resulting in a visible silhouette around the object.

### Particle Systems

Instanced rendering is employed to efficiently render large numbers of particles. Instead of issuing separate draw calls for each particle, per-instance transformation data is uploaded to the GPU. This minimizes CPU overhead and leverages GPU parallelism for rendering effects such as fruit juice splashes and slicing trails.

The particle system plays a crucial role in rendering dynamic effects. Bombs constantly emit fire sparks and smoke before they detonate, while sliced fruits trigger splash effects. These effects involve spawning many similar rendering objects that share geometry but differ in transformation and timing. The particle system manages these efficiently using instancing, allowing for real-time visual richness without compromising performance.

Currently, the particle system updates vertex buffers by mapping them to CPU memory, writing particle positions each frame, and then unmapping before issuing a draw call. While this approach works, it may stall the GPU due to synchronization issues, particularly if the GPU has not yet finished reading from the buffer. This area is identified as a candidate for future optimization, potentially using techniques like persistent mapping or double-buffering to reduce stalls.

### UI Rendering

UI elements such as score displays and debug overlays are rendered using a screen-space approach with a single UI component. This component supports one type of UI element that can include both an image and text. There is no centralized UI system — instead, the component exposes a draw interface that client code invokes at the end of each game loop.

Text rendering relies on a bitmap font atlas and screen-aligned quads. Each character glyph is mapped from the atlas and rendered using a four-vertex quad. Vertex attributes include position and UV coordinates, with character size encoded within the position itself. All characters share a single texture atlas to enable batching in a single draw call. UV coordinates are calculated for each glyph and adjusted to the center of edge pixels to prevent texture bleeding.

UI elements are placed in view space, and the vertex shader does not apply a view matrix. However, it accepts a projection matrix uniform to enable perspective projection, giving the UI a more dynamic, 3D-like appearance. UV coordinates are passed as vertex attributes for texture mapping.

Both text and image rendering use the same UI shader. The fragment shader takes in both the font atlas and image texture and uses an integer flag to determine which one to sample. For images, a quad of four vertices contains the UV coordinates, a relative position offset from the UI anchor, and the texture selection flag. The image is centered by adding or subtracting half its width and height from its anchor position.

To make UI placement more convenient, the component also provides a method that accepts coordinates in Normalized Device Coordinates (NDC). This method converts NDC positions to view space before rendering, simplifying alignment and layout logic for developers.

This approach supports visually informative HUD elements with minimal overhead while ensuring clarity and consistency across frames.

## Audio System

The audio system brings life to the gameplay experience by delivering spatially-aware sound effects and ambient music. Built on top of the OpenAL API, it integrates tightly with the component-based object model to associate sound behavior with individual game objects.

There are two main audio components:

- **AudioSource**: Represents an emitter of sound in the 3D scene. It controls playback state (play, pause, stop), tracks spatial position, and manages sound-specific parameters like pitch and gain.
- **AudioListener**: Represents the player's auditory perspective in the OpenAL context, typically attached to the game camera to simulate how sounds are heard in the world.

All audio data is loaded using the AudioFile library. Like other resources such as models and shaders, sound data is managed via RAII — it is loaded during initialization and automatically released during object teardown. This helps avoid memory leaks and manual cleanup, keeping audio management consistent with the rest of the engine.

During the physics update, each active `AudioSource` and `AudioListener` updates its spatial location to simulate realistic attenuation and directional cues. 

Because OpenAL imposes a hardware- or driver-dependent limit on the number of simultaneous sound sources, the game implements a pooling mechanism to reuse `AudioSource` instances efficiently. The pool owns a stack of reusable `AudioSource` pointers. When an audio effect is triggered, a `std::unique_ptr` with a custom deleter is issued. This deleter, instead of freeing memory, pushes the instance back into the pool’s stack once playback is complete. This approach reduces allocation overhead and ensures that frequent, short-lived sound effects (like slicing or explosions) do not cause unnecessary churn. When sound playback is needed — for example, when a fruit is sliced — an available `AudioSource` instance is requested from the pool. The system sets its position, assigns the proper sound clip, and plays it immediately.

### Typical Uses of Audio

- **Fruit slicing**: splash and swish sound effects
- **Bomb activity**: ticking warnings and explosion sounds
- **Environmental ambiance**: subtle background cues
- **Menu transitions**: background music on entering the game menu

The `AudioSource` component exposes simple controls to modify playback in real time — including volume, pitch, and looping behavior — allowing dynamic sound customization during gameplay.

### ⚠️ Notes on AudioSource Pooling

While the custom `AudioSource` pooling mechanism provides efficient reuse of OpenAL sources, it introduces a caveat related to object lifetime. The pool returns `std::unique_ptr` instances with custom deleters that push returned pointers back into the stack. If the pool is destroyed before any active `AudioSource` instances are returned, the custom deleter will attempt to access a dangling pointer, resulting in undefined behavior.

The current game implementation ensures this situation does not occur by controlling the lifetime of the pool relative to all active sources. However, if pooling logic is reused in more dynamic or long-lived systems, additional safeguards or architectural adjustments may be required to prevent premature deallocation.


Although relatively lightweight, the system provides all essential functionality needed for real-time feedback and integrates cleanly with the game loop. Each frame, the audio components are updated in sync with the object lifecycle, ensuring all sound-emitting behaviors remain consistent with game state and timing.

## Coroutine System

This system allows the game to express sequential logic such as time-based transitions or delays across multiple frames without cluttering the main game loop or relying on manual state machines.

For example, coroutines are used to animate UI transitions between game states: when a player selects a game mode, the current UI text gradually fades out, and the text for the next game state fades in. This effect is achieved by progressively changing UI transparency values over time within a coroutine. 

I've had experience with C# corotuine for Unity developement, however, C++ does not provide such functionality out of box. Only the core mechnisms to implement such feature is provided since C++20. I found this [series of blogs](https://lewissbaker.github.io/2017/09/25/coroutine-theory) by Lewis Baker and this [cppcon video](https://www.youtube.com/watch?v=8sEe-4tig_A&t=1427s) to be incredibly helpful. C++20 provides the foundational language support for implementing coroutines by introducing `co_yield`, `co_return`, and `co_await`. In this project, a simple coroutine system was developed using `co_yield` and `co_return`. The `co_yield` keyword pauses execution and returns an `optional<YieldOption>` to the coroutine manager, which uses the yielded value to determine how and when the coroutine should be resumed.

The `YieldOption` struct provides static methods such as `YieldOption::Wait(float timeToWait)` and `YieldOption::WaitUnscaled(float timeToWait)` to express delay behavior. This mechanism is similar in spirit to `std::this_thread::sleep_for`, but adapted to a frame-based asynchronous system suitable for real-time applications. The `co_return` statement ends the coroutine immediately.

To use a coroutine, a `CoroutineManager` must first be instantiated. Coroutines are defined by functions that return a `Coroutine` type and use either `co_yield` or `co_return`. After the manager is created, a coroutine is launched by calling the coroutine function like a regular function and adding the returned object to the manager. The manager is then responsible for resuming all active coroutines every frame.

### Example: Fading Out UI with a Coroutine

Here is an example of how the coroutine system is used to gradually fade out a UI button over a given duration:

```cpp
Coroutine FadeOutUI(UI* button, float duration, float updateInterval) {
    float time = 0;

    while (time < duration) {
        time += updateInterval;
        float opacity = 1 - time / duration;

        button->textColor.a = opacity;

        co_yield YieldOption::Wait(updateInterval);
    }

    button->textColor.a = 0;
    co_return;
}
```

To run this coroutine, a `CoroutineManager` is used as shown below:

```cpp
void main() {
    UI ui;
    CoroutineManager manager;
    manager.AddCoroutine(FadeOutUI(&ui, 2, 0.1));
    while (!manager.empty()) manager.Run();
}
```

This demonstrates a simple use case where the coroutine manager handles the timing of UI updates by executing the coroutine step-by-step across frames.

This coroutine gradually decreases the alpha value of the button’s text color. Each time it yields, it tells the coroutine manager to pause execution for a fixed interval before resuming, creating a smooth fade-out effect over time.

# Game Mechanics

This section outlines how the core gameplay mechanics are implemented to simulate the classic Fruit Ninja experience.

## Slicing Detection

Slice detection is implemented inside the per-frame update of components attached to sliceable objects. If the left mouse button is pressed in both the current and previous frames, the system forms a slicing line from the mouse positions of those two frames. This line is then tested against each object's sliceable region, which is approximated as a sphere with a predetermined radius. If an intersection is found, the object is marked as sliced.

To simulate the feel of a real slice, sliced objects trigger visual and auditory feedback — including mesh break-up, particle effects (e.g., juice splash), and sound cues. When a fruit is sliced, the original object is immediately deactivated and replaced with two preconstructed mesh models representing the top and bottom halves of the fruit. The orientation of the sliced pieces is determined by the up direction, which is computed as the vector perpendicular to the line segment used in the slice detection logic. These effects are accompanied by auxiliary components (such as VFX or particles) and triggered audio playback using an `AudioSource` acquired from the pool.

## Fruit and Bomb Behavior

Fruits are launched into the air with randomized velocity and direction using a physics-driven system. Each fruit follows a simple trajectory under gravity until it is sliced or exits the screen. Upon initialization, each fruit and bomb component receives a reference to a shared control block associated with the current game state. This control block maintains key gameplay data such as the player's current score and number of misses. When a fruit is successfully sliced, the component updates the control block accordingly. This mechanism also enables the fruit to query the UI system for stateful feedback without directly owning UI logic, keeping the interaction decoupled and modular.

Bombs behave similarly in terms of movement, but their interaction differs: slicing a bomb immediately ends the game. Active bombs also periodically emit sparks and sounds to signal their danger.

## Game Progression

The game tracks the player's score and remaining lives. Each missed fruit reduces the life count, and slicing a bomb results in instant game over. When all lives are lost or a bomb is triggered, the state machine transitions the game to a game-over state, which may display score summaries and offer restart options.

These mechanics combine to create a fast-paced and rewarding arcade-style gameplay loop that emphasizes precision, timing, and feedback.

# Multiplayer Mode (In Development)

The multiplayer mode builds upon the single-player foundation by introducing real-time competitive gameplay over a custom-designed UDP networking layer. This system is crafted using Winsock in C++, providing a lightweight and low-latency protocol tailored for arcade-style interactions.

## Networking Architecture

- **Transport Protocol**: UDP (User Datagram Protocol), selected for its minimal overhead and suitability for fast-paced, real-time updates.
- **Custom Connection Layer**: A handshake-based session system ensures peer identification and connection stability. The protocol includes:
  - A connection initiation handshake with session ID assignment (server-controlled).
  - Optional reliability for important packets (e.g., game state updates).
  - Direct peer address tracking and reconfiguration for NAT traversal and IP changes.

## Protocol Specification Highlights

The multiplayer communication protocol is detailed in the [UDP Connection Protocol Specification](https://github.com/DiceSpinner/FruitNinja/blob/main/Common/doc/udp_connection_protocol_spec.md). Key components include:

- **Packet Structure**: Each packet comprises a header and payload. The header contains fields such as packet type, sequence number, and session ID, facilitating proper routing and handling.
- **Handshake Mechanism**: A three-step handshake process establishes a session between peers, ensuring both parties are ready for data exchange.
- **Reliability Features**: While UDP is inherently unreliable, the protocol introduces selective acknowledgment and retransmission mechanisms for critical packets, balancing performance with data integrity.
- **Session Management**: The `UDPConnectionManager` oversees active sessions, handling timeouts and reconnections as necessary.

## Core Design Goals

- **Low Latency**: Emphasis on real-time responsiveness, crucial for gameplay fluidity.
- **Selective Reliability**: Implementing reliability only for essential data to maintain performance.
- **Threaded Architecture**: Background threads manage routing, packet queues, and filtering, preventing main game loop interruptions.

## Planned Multiplayer Features

- **Competitive Slicing**: Two players compete to survive the longest by slicing fruits while avoiding bombs. Each sliced fruit grants energy, which can be spent to launch bombs toward the opponent. Players lose if they miss too many fruits or slice a bomb. Real-time score and energy levels are tracked and synchronized between both clients.
- **Combo & Critical Mechanics**: Players can achieve bonus points through multi-fruit slices and precise timing.
- **State Synchronization**: The game follows a server-authoritative model where the full simulation runs on the server. Clients send their slice input to the server every frame, while the server periodically broadcasts the authoritative positions and states of all game objects at fixed intervals to ensure consistency and prevent cheating.

## Current Progress

- ✅ Connection management layer complete
- ✅ Basic packet exchange and dispatch system implemented
- 📀 Slice synchronization prototype in progress
- 📀 Real-time scoring and game state syncing under development
