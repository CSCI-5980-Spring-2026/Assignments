# Assignment 3: Asset Conditioning Pipeline

**Due: Wednesday, April 15, 11:59pm CT** (extended)

In this assignment, you will implement the offline asset conditioning pipeline to prepare assets (meshes, materials, and textures) in a game-ready format. In the runtime engine, you will also add code to the resource manager that loads these processed resources.

Late submissions will be accepted until *Sunday, April 26*. As described in the [syllabus](https://github.com/CSCI-5980-Spring-2026/Syllabus), the late penalty is 5% per day, which can be reduced by spending late points.

## Code Organization

This assignment includes an additional executable program called AssetConditioner. This is a console-only program that processes source assets (located in the `assets` folder) and outputs game-ready resource files (generated in the `resources` folder). Note that since the asset conditioner runs offline, we don't need to worry too much about speed or robust error handling.

In VS Code, you can switch between the runtime engine (MainLoopTest) and the offline asset conditioner by opening the Command Palette (CTRL+SHIFT+P) and selecting `CMake: Set Launch/Debug Target`.

The asset conditioner starter code contains a main function that does the following:

- Creates the following output folders if they don't already exist: `resources`, `resources/meshes`, `resources/materials`, and `resources/textures`. 
- Deletes any files in these resources directories that were generated during a previous run, so the asset conditioner starts with a clean slate.
- Recursively scans the input folder `assets/meshes` for all files matching the extensions  `.obj, .gltf, .glb`, and calls the `process_mesh()` function for each one.
- Recursively scans the input folder `assets/textures` for all files matching the extensions  `.png, .jpg, .jpeg`, and calls the `process_texture()` function for each one.

The `process_mesh()` and `process_texture()` functions are currently stubs. You will complete the implementation for each one, as described in the rubric.

Several meshes and textures are already included in the `assets` folder, and these are enough to sufficiently test all the program functionality. However, if you want to add more assets to the project, you may need to expand the list of file extensions that are scanned by the main function.

## Rubric

Graded out of 20 points. Steps that are worth a single point are typically all or nothing. For steps that are worth more than one point, partial credit may be given for solutions that are reasonably attempted, but not entirely correct.

**Part 1: Mesh Resources** (4 points)

In the asset conditioner, process each mesh using the Open Asset Import Library (assimp) and export the following to the `resources/meshes` folder. Similar to class, you can assume each asset only has one mesh; handling multiple submeshes in a single asset is beyond the scope of this assignment.

- A binary file named using the resource GUID that contains the interleaved vertex array and the index array. (1)
- An ASCII file with the same name and the `.metadata` extension. It should include the resource name (asset path), the size of the vertex array, and the size of the index array. This metadata file is necessary for the runtime engine to know how to load the resource properly. (1)

Next, in the runtime resource manager, add code to the `load_mesh()` function. 

- In the `on_complete()` handler, you will first need to synchronously load the metadata file to get the resource name and buffer sizes. This file is extremely small, so don't worry about asynchronous loading. You can just use the standard `ifstream`, and it will complete very quickly. (1)
- Once you have the buffer sizes, you can then parse the binary `file_data` and populate all the fields of the `Mesh` object. (1)

When you are ready to test this part, uncomment the code in `MainLoopTest` that loads `bunny.obj`, and the bunny mesh will replace the cube mesh in the scene.

**Part 2: Texture Resources** (4 points)

In the asset conditioner, process each texture using SFML and export the following to the `resources/textures` folder, using the same naming conventions as Part 1.

- A binary file containing the pixel data. (1)
- An ASCII metadata file that includes the resource name (asset path), image width, and image height. (1)

In the runtime resource manager, add code to the `load_texture()` function.

- Synchronously load the metadata file in the `on_complete()` handler. (1)
- Parse the binary pixel data and populate the fields of the `Texture` object. (1) 

When you are ready to test this part, uncomment the code in `MainLoopTest` that loads a texture and adds it to the bunny mesh component.

**Part 3: Material Resources** (5 points)

After completing the first two parts, the engine can now load mesh and texture files separately. However, some mesh file formats also include materials and texture references. Currently, these are ignored by the asset conditioner.

For example, `spot.obj` references a material defined in the `spot.obj.mtl` that is also stored in the `assets/meshes` folder. The material references a texture `spot.png` that is stored in the `assets/textures` folder.

First, you will need to add some code to the asset conditioner.

- If a mesh references a material, then you should export an ASCII material file to `resources/materials`. The file should include the resource name, ambient RGB, diffuse RGB, specular RGB, and shininess coefficient. Note that you can generate a new resource name by appending `:material` to the mesh name (asset path). (1)
- You should also add a string to the exported mesh metadata file that specifies the material resource name or `none` if the mesh does not reference a material. (1)

Next, you will need to extend the runtime resource manager:

- Complete the `load_material()` function to parse the material data and populate the fields of the `BlinnPhongMaterial` object. (1)
- In the `load_mesh()` function, if the metadata specified a material name other than `none`, then you should:
  - Add a new entry to `mesh_material_references_`. This is a dictionary that maps each mesh GUID (key) to material GUID (value). This will be needed for resolving cross-references later. (1)
  - Start an asynchronous load by calling the `load_material` function. (1)

**Part 4: Resolving Cross-References** (1 point)

In `MainLoopTest`, uncomment the code that loads `spot.obj`. Note that the object will not appear yet, because no material has been added to the mesh component. Because both the mesh and the material are loaded asynchronously, we need to wait until all resources are finished loading to resolve the cross reference. To achieve this, a new `on_load_complete()` callback has been added to the resource manager.

- Add code to the appropriate callback in `MainLoopTest` that looks up the material GUID associated with the spot mesh GUID, retrieves the material object from the resource manager, and then assigns it to the mesh component. (1)

When this part is completed, a cow will appear in the scene.

**Part 5: External Textures** (3 points)

The material for `spot.obj` references a texture file, but the object does not display a texture yet.

In the asset conditioner, you will need to add a string to the material file that describes the texture.

- If the material references a texture, specify its resource name (asset path). This will be needed by the resource manager to load the texture and generate the GUID. If there is no texture, then specify `none`. (1)

*Note: in the original version of the assignment, there was a line here that mistakenly described appending `:texture` to the path. That note actually belonged in Part 6 for embedded textures. I have extended the assignment deadline by two days in case anyone was confused by this error.*

In the runtime resource manager's `load_material()` function, if the material references a texture, then we can follow a similar procedure as Part 3.

- Add a new entry to `material_texture_references_`. (1)
- Start an asynchronous load by calling the `load_texture` function. (1)

This is all we need to do! The texture should now be displayed on the cow.

Note that we don't need to manually resolve the cross-references between materials and textures. The resource manager owns both, so handles this automatically by calling `resolve_material_texture_references()` when all assets are loaded.

**Part 6: Embedded Textures** (3 points)

For this part, you can uncomment the code in `MainLoopTest` that loads `ogre.glb`. This file is a binary GLTF that contains an embedded texture instead of storing it as a separate image. This situation is handled by assimp as follows:

- If a texture is embedded, the path will be equal to "*".
- Embedded textures are stored as `aiTexture` objects in the `mTextures` field of the `aiScene`. 
- If the `mHeight` of the `aiTexture` is 0, then the texture is embedded as a compressed image. You can use SFML's `loadFromMemory()` function to decode it!
- If the `mHeight` of the `aiTexture` is > 0, then the binary texture data is stored in BGRA format. You will need to reorder it before exporting the resource.

To complete the assignment, you will need to:

- Add embedded texture support to the offline asset conditioner. (2)
- For the texture path that is written to the material file and used to generate the texture GUID, you can generate this by appending `:texture` to the mesh name (asset path). This works because the texture is not actually stored as a separate file.
- Add code to `MainLoopTest` to resolve the cross-reference between the ogre mesh and material. (1)

Note that this task is intentionally less structured than the previous parts of the assignment, and it will likely require consulting the online documentation for assimp. AI tools can also be very useful in explaining how embedded textures are represented in the import library's data structures.

**Bonus Challenge** (1 point)

For this assignment, we assume that the assets we are loading only include a single mesh that may optionally have a material and diffuse texture map. However, many mesh formats provide multiple submeshes, materials, and textures. Modern formats like FBX, Collada, and GLTF actually allow full hierarchical scene graphs. There are also plenty of other features we did not implement, such as PBR materials.

For a bonus point, you can extend the asset conditioner and runtime engine with support for an additional resource feature or format. Of course, you will need to find suitable asset files online for demonstration purposes (or create them yourself). Creativity is encouraged!


## Repository Setup

We are using GitHub classroom for submission of programming assignments. When you accept the first assignment, you will need to select your x500 from the class roster. The system will then create a new private repository with template code that is only accessible by you and the instructor. You will need to create a GitHub.com account if you do not already have one. Note that this is different from the University's github.umn.edu account.

**Step 1:** Create your private repository using the following link: [https://classroom.github.com/a/BOWzmEGg](https://classroom.github.com/a/BOWzmEGg)

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
