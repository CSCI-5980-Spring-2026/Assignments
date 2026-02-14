# Assignment 1: Implementing the Scene Graph

**Due: Thursday, February 26, 11:59pm CT**

At the end of our last live programming class, all of the nodes were stored in a flat vector in the `Scene` class. In this assignment, you will implement a scene graph structure. Each `Node` will contain an arbitrary number of child nodes, and update and draw calls will recursively propagate through the entire graph.

We weren't quite able to finish implementing the assignment starter code during our last live programming class. So, please watch the [video](https://mediaspace.umn.edu/media/t/1_zg0x21sl) that I have prepared to finish and explain the code you will be extending in this assignment.

Late submissions will be accepted until *Wednesday, March 18*. As described in the [syllabus](https://github.com/CSCI-5980-Spring-2026/Syllabus), the late penalty is 5% per day, which can be reduced by spending late points.

## Rubric

Graded out of 15 points. Steps that are worth 1 point are typically all or nothing. For steps that are worth more than one point, partial credit may be given for solutions that are reasonably attempted, but not entirely correct.

**Part 1: Restructuring the Scene and Node Classes** (4 points)

- As a first step, you remove the vector from `Scene`, along with the  `add_child()` and `create_child()` methods, and move them to the `Node` class. (1)
- The `Scene` class should have just a single node named `_root` that gets instantiated in the constructor and gets called in the appropriate places in the `update()` and `draw()` methods. (1)
- In general, member variables should **never** be public! Therefore, you will also need to write an accessor method `Scene::get_root()` that returns a `shared_ptr`, so we can access the root node from outside the class. (1)
- Next, you will need to add public mutator methods for the node's position, rotation, and scale, so they can be updated in the main loop of your program. There are multiple ways to implement this. For example, the method could return a memory reference of the member variable, such as `glm::vec3&`, so they can be modified directly. Alternatively, you could write getters and setters, which involves passing by value and is bit more verbose. You are free to choose the implementation. Just don't make the member variables public! (1)

**Part 2: Animating the Root Node** (2 points)

- In the `Node` class, you should remove all the hard coded modification of the `position_`, `rotation_`, and `scale_` from the constructor and the `update()` method. Then, add code to the `MainLoopTest` class that places the cube 2 units in front of the camera. (1)

- Next, add code to the `MainLoopTest` class that makes the root node rotate at a constant speed. (1)

**Part 3: Placing Child Nodes** (4 points)

- In the `MainLoopTest` class, add 1000 child nodes to the root node of the scene. Note that these will not be drawn until you add code in the next step. (1)

- To make the child nodes appear, you need to add code at the end of `Node::draw()` that recursively calls the `draw()` method for all its children. (1)

- Each child node should be initially placed 50 units away from the camera in a random direction. In other words, the nodes will be placed at random points on the surface of an imaginary sphere. When you are done, the child cubes should appear to surround the parent cube, but they will not be animated yet. (2)  
  *Hint: There is more than one way to do this. However, one straightforward way will be to construct a quaternion using two random angles, and then use it to rotate a vector.*

**Part 4: Scene Graph Propagation** (5 points)

- Add code at the end of `Node::update()` and `Node::update_matrices()` to recursively call these methods for all the node's children. (1)
- After completing the previous step, you may notice that the framerate has dropped, causing the root node's animation to appear choppy. In the terminal, you should be able to see the `cout` from all 1000 nodes when the local matrix is recomputed each frame. However, none of these nodes have been moved from their initial positions, so we are wasting a ton of computation by recomputing the local matrix! You can fix this by adding a `local_matrix_dirty_` flag as a member variable. This flag should be set to true whenever the public mutator methods for `position_`,  `rotation_`, and `scale_` are called. Then, modify `Node::update_matrices()` so that the `local_matrix_` is only recomputed when it is dirty. When this step is complete, the framerate should be back to normal, and you should only see the `cout` output for the root node each frame. (2)
- For the last step, you will need to modify the `Node::update_matrices()` method so that `world_matrix_` recursively propagates to all the child nodes. Each node's world matrix should be computed by combining it with the world matrix of its parent. Remember that matrix multiplication is not commutative! When you are finished, all of the child cubes should appear to orbit around the root node, following the animation you implemented in the previous step. (2)  
  *Note: this is the trickiest part of this assignment. The parent's world matrix is currently not accessible in the node, so you will need to figure out how to recursively propagate it through the scene graph.*

**Bonus Challenge** (1 point)

- For a bonus point, you can implement a new, useful feature to one of the GopherEngine modules. You have a lot of freedom to choose what you'd like to work on, but it should not be something excessively trivial, like adding an accessor method for a member variable.
  - To further optimize the scene graph, the node's world matrix should only be recomputed if either the local or parent matrix is dirty. You could modify the `update_matrices()` method to recursively propagate a parent matrix dirty flag through the scene graph if a node's world matrix was updated.
  - The `Node` class currently has methods for adding and creating children. However, there are no methods for searching and removing a node from the scene graph. Note that each node has a unique `id_` number that increments using a static counter each time a new node is created. You could add a method that takes in an id number and recursively searches the graph. If the node is found, it should be removed and a `shared_ptr` should be returned. Note that `shared_ptr` is nullable, so the method can return a `nullptr` if the id is not found.
  - These are just a couple ideas. Creativity is encouraged!


## Repository Setup

We are using GitHub classroom for submission of programming assignments. When you accept the first assignment, you will need to select your x500 from the class roster. The system will then create a new private repository with template code that is only accessible by you and the instructor. You will need to create a GitHub.com account if you do not already have one. Note that this is different from the University's github.umn.edu account.

**Step 1:** Create your private repository using the following link: TO BE ADDED.

**Step 2:** Note that GitHub Classroom may show you a "Repository Access Issue"  error. This can safely be ignored. To gain access to your repository, you will need to check the email associated with your GitHub account for an invitation. Note that this may be a different from your UMN email. Note that GitHub Classroom may show you a "Repository Access Issue"  error. This can safely be ignored

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

## License

Material for [CSCI 5980 Spring 2026](https://github.com/CSCI-5980-Spring-2026/Syllabus) by [Evan Suma Rosenberg](https://illusioneering.umn.edu/) is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).