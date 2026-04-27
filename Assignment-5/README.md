# Assignment 5: Physics and Spatial Audio

**Due: Saturday, May 9, 11:59pm CT**

In the culminating assignment for this class, you will implement a first-person camera controller that uses the physics system to walk across uneven terrain. You will also apply what you have learned about game engine design to create a new audio module with 3D spatialized sound.

Because this assignment is due instead of a final exam, **no late submissions** can be accepted.

## Code Organization

For this assignment, the engine has been extended with several new capabilities. For more explanation of how this code works, please see the video for programming class 14.

**Physics System**

`PhysicsWorld` now has full support for kinematic rigid bodies. This is set by passing `MotionMode::Kinematic` to the `RigidBodyComponent` constructor. As discussed in lecture 8, kinematic bodies (also known as game-driven bodies) are moved by game code (i.e., in `kinematic_update()`) instead of being driven by the rigid body simulation. They can still participate in collision, but are not affected by physics and have infinite mass.

`CollisionPair` now exposes more information about the collision besides just the two collider IDs. This includes the contact point, penetration depth, collision normal, and velocities of each colliding object. This information may be useful when manually calculating the collision response for a kinematic rigid body.

**Terrain**

Height-mapped terrain has been added to the engine as a new resource type. Height maps that are added to the `assets/terrain` folder will be processed by the asset conditioner, and the game-ready resource will be exported to the `resources/terrain` folder, similar to the other assets.  In the runtime engine, functions to load the terrain have also been added to the `ResourceManager`, which registers the resource as a `Terrain` object.

Height-mapped terrain is rendered using its own specialized material (`TerrainMaterial`) and shaders (`terrain.frag` and `terrain.vert`), and is represented in the Bullet physics world using `btHeightfieldTerrainShape`.  All of this functionality has already been added to the runtime engine, and the test scene includes a terrain object. So, any new objects you create with a collider and rigid body attached will participate in collisions with the terrain.

The canyon terrain was created using an open-licensed height and diffuse map obtained from [Motion Forge Pictures](https://www.motionforgepictures.com/height-maps/). The test scene is using the 4k diffuse texture by default, but if this loads or renders slowly on your computer, you can change it to the 2k version that is also located in the assets folder.

**Gameplay Module**

The `OrbitControls` class has been moved into a new `Gameplay` module, and a new `FirstPersonControls` class has been added. The first-person controls use the standard conventions for mouse and keyboard movement (WASD). You can also press `F` to toggle fly mode. When fly mode is off, movement is constrained to the XZ plane. Note that the first-person controls **do not** currently detect collisions, so you will still be able to fly inside the terrain or any other objects in the scene, even if they have colliders attached.

**Audio Resources**

Sound clips have also been added as a new resource type. The asset conditioner processes any files in the `assets/sounds` folder, decodes them to linear PCM using SFML's audio library, and then outputs the binary resource to `resources/sounds`. In the runtime engine, functions to load sound clips have also been added to the `ResourceManager`, which registers the resource as a `SoundClip` object.

## Rubric

Graded out of 20 points. Steps that are worth a single point are typically all or nothing. For steps that are worth more than one point, partial credit may be given for solutions that are reasonably attempted, but not entirely correct.

**Part 1: Walking over Uneven Terrain** (8 points)

In this part, you will need to extend `FirstPersonControls` so that when fly mode is toggled off, you can walk along the surface of the terrain. This involves the following additions:

- A collider component. In my implementation, I used a simple sphere collider with a radius of 1.6m as an approximation, though a capsule collider would be more realistic. (2)
- A kinematic rigid body. This is necessary for the collisions to work correctly when the object is game-driven. If you don't add a kinematic rigid body and only have a collider on a node, Bullet physics will consider the object to be static, and collision calculations may be inconsistent and unpredictable. (2)
- Gravitational force. Since the object is kinematic and not controlled by the simulation, you will need to move the object downward each frame in `kinematic_update()`. In my implementation, I used the explicit Euler method, as discussed in lecture 8, which involves keeping track of the velocity and applying a constant acceleration of 9.8 m/s<sup>2</sup>. However, games often apply non-realistic gravity to the camera/player to improve the gameplay experience, so other gravity implementations are also acceptable. Note that gravity should not be applied in fly mode, which is turned on when the application is started. (2)
- Collision response. When you add gravity, the camera will fall through the terrain. Although collisions with the static terrain collider will be detected by the physics system, we need to manually adjust the node's position to move out of the terrain so that the collider is resting on the surface. This correction should be done in the `update()` method, which is triggered after the physics update. This can be tricky to get right, and handling it properly will require querying the `PhysicsWorld` for the collision normal and the penetration depth. Be careful to use the collision normal with the correct sign, based on which collider in the pair represents the player controller. (2)

Note that in production, a first-person controller would probably want to do more sophisticated calculations to handle slopes correctly, such as specifying a maximum slope angle that is walkable, and deal with other edge cases that arise when moving over uneven terrain. You don't need to worry about that to receive full credit for this part. However, enhancing the controller with additional functionality such as this would be a bonus challenge opportunity.

**Part 2: Jumping** (2 points)

Once the player is able to walk on the terrain surface, it would probably also be useful to be able to jump. For this part, you should trigger a jump using the `Space` key. Because I used the explicit Euler method for gravity, this extension was quite simple: a jump can be achieved by adding an impulse (instantaneous change) to the `y` component of the velocity. However, if you used a different method to simulate falling, then adding the jump may require additional wiring. 

**Part 3: Spatialized Audio** (10 points)

For the last part of this assignment, you will need to add a new `Audio` module from scratch that implements spatial sound. Fortunately, SFML has an audio library that is quite simple to use. We are already using SFML for window management, so it's a convenient option.

First, I'd recommend you read some of the documentation on how the SFML audio library works:

- [Playing sounds and music](https://www.sfml-dev.org/tutorials/3.1/audio/sounds/)
- [Spatialization: Sounds in 3D](https://www.sfml-dev.org/tutorials/3.1/audio/spatialization/)

Your `Audio` module should include a **spatial sound component** that can be attached to a node. Its functionality should include:

- Creation of a `sf::SoundBuffer` and `sf::Sound` from a `SoundClip`. The `SoundClip` resource contains the binary audio sample data along with the sample rate, channel count, and sample count. This is all the information necessary to load the sound buffer using its `loadFromSamples()` method, except for a channel map. For this assignment, you can assume the channel map contains only `sf::SoundChannel::Mono`. See the [sf::SoundBuffer documentation](https://www.sfml-dev.org/documentation/3.1.0/classsf_1_1SoundBuffer.html#a8189b2f4dc98bfaeefa49cd2c5a0fc33) for more details. (2)
- Wrapping the necessary functions that control playback, pausing, stopping, and audio parameters. See the [sf::Sound documentation](https://www.sfml-dev.org/documentation/3.1.0/classsf_1_1Sound.html) for a full list. (2)
- An overridden `sync_transforms()` method that extracts its position from the world matrix and sets it in the `sf::Sound`. (2)

A component that specifies the listener (you can assume only one per scene). Its functionality should include:

- An overridden `sync_transforms()` method that extracts the position, forward direction, and up vector from the node's world matrix and sets them in the `sf::Listener`. (2)

I will also expect you to create this module using the best practices we have learned in class.  This includes:

- The audio module is defined correctly in the CMakeLists.txt file, separate from other modules and without linking to extraneous libraries. (1)
- The audio header files respect module boundaries, using techniques such as forward declaration to avoid leaking SFML dependencies to other modules that include them. (1)

Your test scene should include at least one sound that demonstrates the spatialization is working correctly. The assets folder already contains a mildly annoying beep sound effect obtained from [freesound.org](https://freesound.org/people/Entershift/sounds/704134/) that you may find useful for testing. You are also welcome to use other sound effects you find online (or record your own). However, note that spatial sounds should be **monophonic**. [Audacity](https://www.audacityteam.org/) is a free and open-source digital audio editor that you can use to mix stereo sounds down to a single channel.

**Bonus Challenge** (1 point)

As always, you can earn bonus credit by going above and beyond these requirements. There are plenty of opportunities to extend the first person controls with more sophisticated collision-aware movement behaviors. Alternatively, the audio module could also interact with the physics system. For example, you could perform a ray cast to determine if the path to the sound source is blocked by an object, and then attenuate the sound. Or you could come up with something completely different, like procedurally-generated audio. Creativity is encouraged!


## Repository Setup

We are using GitHub classroom for submission of programming assignments. When you accept the first assignment, you will need to select your x500 from the class roster. The system will then create a new private repository with template code that is only accessible by you and the instructor. You will need to create a GitHub.com account if you do not already have one. Note that this is different from the University's github.umn.edu account.

**Step 1:** Create your private repository using the following link: [https://classroom.github.com/a/SmOmF4IK](https://classroom.github.com/a/SmOmF4IK)

**Step 2:** To gain access to your repository, you will need to check the email associated with your GitHub account for an invitation. Note that this may be different from your UMN email. Note that GitHub Classroom may show you a "Repository Access Issue" error. This can safely be ignored.

**Step 3:** Your private repository will be added to the [GitHub course organization](https://github.com/CSCI-5980-Spring-2026). It will already contain the starter code that was forked from the assignment template project. You can then check out the code to your local machine. If you are not familiar with using `git` from the command line, then I recommend using a GUI such as [GitHub Desktop](https://desktop.github.com/).

## Prerequisites

To work with this code, you will first need to install the following software (same or newer versions):

- [Visual Studio Code](https://code.visualstudio.com/)
- [CMake 4.2.1](https://cmake.org/download/)
- [LLVM 21.1.8](https://github.com/llvm/llvm-project/releases/tag/llvmorg-21.1.8)
- [Ninja 1.13.2](https://github.com/ninja-build/ninja/releases/tag/v1.13.2)

On Windows 11, the easiest way to install these is to use the winget package manager.

```
winget install code
winget install cmake
winget install llvm
winget install Ninja-build.Ninja
```

**Important:** You will need to add the LLVM binary path to your system path environment variable. By default, this is installed in `C:\Program Files\LLVM\bin`.

## Visual Studio Code Extensions

You will also need to install the following VS code extensions:

- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
- [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)

## MSVC Platform Libraries

Although we are using the LLVM compiler, our application will still need to link to the MSVC and Windows SDK libraries. Therefore, you will need to install [Microsoft Visual Studio Community](https://visualstudio.microsoft.com/downloads/?q=build+tools) even though we are not using it as an IDE (there does not appear to be any way around this on Windows). In the installer, make sure to check *Desktop development for C++*, and then it will install the required dependencies.
