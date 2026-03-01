# Assignment 2: Rendering Pipeline

**Due: Thursday, March 19, 11:59pm CT**

At the end of our last live programming class, we had almost completed the code for the `BlinnPhongMaterial`.  First, please watch the [video](https://mediaspace.umn.edu/media/t/1_u7ee44mz) that explains the finished code you will be extending in this assignment.

In this assignment, you will be additional functionality to the game engine, including interactive camera control, a wireframe rendering mode for the two existing materials, additional geometric primitives, and new shaders/materials.

Late submissions will be accepted until *Wednesday, April 1*. As described in the [syllabus](https://github.com/CSCI-5980-Spring-2026/Syllabus), the late penalty is 5% per day, which can be reduced by spending late points.

## Rubric

Graded out of 20 points. Steps that are worth 1 point are typically all or nothing. For steps that are worth more than one point, partial credit may be given for solutions that are reasonably attempted, but not entirely correct.

**Part 1: Camera Control** (2 points)

First, you should add some interactive camera controls using the keyboard and mouse. To enable this, input callbacks have been added to the `MainLoop` and `MainLoopTest` classes.  These callbacks are triggered by the engine every frame during the game loop just before the `update()` methods are called. 

You are free to choose the input scheme. For example, you may choose to implement a first person camera using WASD keys for movement and the mouse for aiming, or you could implement a camera that orbits around a fixed point using the mouse and zooms in/out using the mouse wheel. Note that [GopherGfx](https://github.com/illusioneering/GopherGfx/tree/main/src/interaction) includes implementations of both types of camera control, which you may choose to adapt. Alternatively, you can come up with your own custom control method; just make sure that both the position and rotation of the camera are controllable.

**Part 2: Wireframe Mode** (3 points)

- Add a wireframe mode to `UnlitMaterial` and `BlinnPhongMaterial`. (2)  
- The wireframe mode should be toggled on and off when the user presses `1` on the keyboard. (1)

Hint: The code for the actual wireframe rendering is extremely simple, because OpenGL has a built-in polygon mode that controls whether it draws filled polygons, lines, or points. However, remember that functions modifying OpenGL state variables are global, so you should reset the polygon mode back to drawing filled polygons at the end of the `draw()` method. Otherwise, you may encounter *state leak* when you add more materials in Part 4! 

**Part 3: Geometric Primitives** (7 points)

Currently, the `MeshFactory` class only contains a function for creating a cube.  Add functions to generate more geometric primitives with appropriate parameters to control their dimensions:

- Plane/Quad (1)
- Cylinder (2)
- Cone (2)
- Sphere (2)

As noted in class, I adapted the code from the `Geometry3Factory` class in [GopherGfx](https://github.com/illusioneering/GopherGfx/blob/main/src/geometry/Geometry3Factory.ts) to generate the arrays for the cube. You are welcome to do the same. However, note that GopherGfx only includes the implementation of an [icosphere](https://en.wikipedia.org/wiki/Geodesic_polyhedron), which is more mathematically complicated than a UV sphere. You are free to implement either one. You can find a good implementation of the UV sphere geometry in [Three.js](https://github.com/mrdoob/three.js/tree/dev/src/geometries) and many other libraries.

You should add at least one new object to the scene for each type of geometry.

**Part 4: Materials** (8 points)

For this part, you will need to implement **two** new materials the game engine, which will also require adding vertex/fragment shaders. (4 points per material)

You are free to adapt shaders that you can find online. [ShaderToy](https://www.shadertoy.com/browse) is a treasure trove with nearly 10,000 fragment shaders, most of which are straightforward to integrate with our engine. In my test implementation of this assignment, I adapted the [Octograms](https://www.shadertoy.com/view/tlVGDt) and [Phantom Star](https://www.shadertoy.com/view/ttKGDt) shaders.

In general, the shaders on ShaderToy are just WebGL fragment shaders with a predefined set of uniform inputs. Most of the shaders just use `fragCoord`, `iResolution` and `iTime`, which refer to the screen-space pixel of the fragment, the resolution of the viewport, and a cumulative time variable that controls the animation. 

For the vertex shader, we can just use our existing unlit vertex shader without modification. To make the fragment shaders work with our engine, we simply remove `fragCoord` and `iResolution` and replace those variables with the texture coordinate passed in from the vertex shader.  For example, both of the shaders mentioned above begin with this line of code:

`vec2 p = (fragCoord.xy * 2. - iResolution.xy) / min(iResolution.x, iResolution.y);`

This computes a position between (-1, -1) and (1, 1). Replacing this calculation with the texture coordinate was straightforward:

`vec2 p = tex_coord * 2.0 - 1.0;`

The rest of the shader code in the main and helper functions was a direct copy and paste.  On the CPU side, the material just needed to set the time appropriately and then map it to a uniform in the material UBO.

The "full screen" shader effect will then be mapped to the UV coordinates of the mesh. In the case of a sphere, this will render the image output shown in ShaderToy on every face of the cube. You can use this to create some impressive animated textures!

You can apply the new materials onto some of the existing objects you added in part 3 or add entirely new objects to the scene to demonstrate them.

**Bonus Challenge** (1 points)

In part 4, you will get full credit for implementing new materials using fragment shaders you find on ShaderToy or elsewhere online. However, you can earn a bonus point for going above and beyond the base requirements by writing your own shader code from scratch or integrating a more complex set of vertex/fragment shaders that require significant CPU-side implementation. For example, you could implement a shader that involves vertex deformation, mesh morphing, particles, or a wide variety of other visual effects. Creativity is encouraged!


## Repository Setup

We are using GitHub classroom for submission of programming assignments. When you accept the first assignment, you will need to select your x500 from the class roster. The system will then create a new private repository with template code that is only accessible by you and the instructor. You will need to create a GitHub.com account if you do not already have one. Note that this is different from the University's github.umn.edu account.

**Step 1:** Create your private repository using the following link: [https://classroom.github.com/a/nVq88ZdT](https://classroom.github.com/a/nVq88ZdT)

**Step 2:** Note that GitHub Classroom may show you a "Repository Access Issue"  error. This can safely be ignored. To gain access to your repository, you will need to check the email associated with your GitHub account for an invitation. Note that this may be a different from your UMN email. Note that GitHub Classroom may show you a "Repository Access Issue"  error. This can safely be ignored.

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
