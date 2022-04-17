# GSoC 2022 OpenSCAD Proposal: 3D Viewport Graphical Enhancements

## Project Information

OpenSCAD uses a 3D viewport to render the model defined by the user's code. There are two modes: the "preview," triggered by the F5 key; and the "render," triggered by the F6 key. When either of these views are shown, they are presented in the same location. Both modes allow examination of the design through the standard orbit/pan/zoom functions of a 3D viewer. Additional features added to this viewport will give users more options for customizing the render and enhanced ability to view their models.

| Project Title |
| --- |
| 3D Viewport Graphical Enhancements |

## Project Summary

The project would allow the 3D viewport to support custom shaders for the render mode (F6). This requires retrieving the necessary uniforms and other shader input/outputs, then providing them to the program. Moreover, currently the GUI does not have options to allow the user to manage which shader is used, or add their own. My project would add these interface options to support shader management for the render by the user. Additionally, a set of default shaders could be included which the user could switch between.

## Detailed Description

According to the GitHub issue definition, the shader that is used in the 3D viewport preview is defined as `shaders/Preview.vert` and `shaders/Preview.frag`. Currently, the name `Preview` is hardcoded, and replacing the contents of `shaders/Preview.vert` or `shaders/Preview.frag` would alter the shading used by the preview. N.B. the use of custom shaders for preview mode rendering currently has bugs that can lead to artifacts when the object is produced with CSG operations. Also the shader is only used when "show edges" is enabled. Fixing these issues for the preview are not currently in the scope of my proposed GSoC project, although the work could provide good background for making those changes in the future.

Shading for the render, on the other hand, is currently handled fully by [fixed-function](https://www.khronos.org/opengl/wiki/Fixed_Function_Pipeline). In order to add custom shading capability for the render mode, it will be modernized to use GLSL just like the preview. However, the aforementioned artifact issues caused by CSG will not be a problem because the rendered object already has its geometry precomputed.

In order for a different shader to be used, the files for vertex and fragment shaders, respectively, would need to be replaced. In order to allow the user to change the shaders in the OpenSCAD interface, this change would need to be accomplished dynamically. So the editor would no longer rely on the static `Preview` shader filename and could be altered on-the-fly.

The user could add their own shader files through a new menu option within OpenSCAD which allows them to be imported:

> View > Shading > Import New Shader

The files themselves could be stored in a directory on the filesystem. Shaders included with OpenSCAD could be placed in the existing `/usr/share/openscad` directory which is created by default on Linux. This directory is currently used for storing color schemes, among other configuration options. However, permission restrictions would prevent user-added shaders from being stored there. The user's shaders would be placed in another location, which will be determined through further consultation with OpenSCAD developers. One suggestion for Linux is to use `~/.local/share/OpenSCAD` because users will likely have permission for that directory.

A newly added set of menu options would also allow switching between shaders in the 3D viewport:

> View > Shading > Change Shader > `<Shader Name>`

The list of available shaders would be updated dynamically as new files are added by the user. Some preexisting shader examples would be shipped with OpenSCAD by default. Examples of these have been [added](https://github.com/openscad/openscad/pull/3860) on GitHub. This list, however, could be expanded or edited as necessary.

Care would need to be taken to ensure that crashes would not occur if the user chooses an invalid shader file as input. During the [compilation and linking](https://www.khronos.org/opengl/wiki/Shader_Compilation#Validation) of the supplied shader, appropriate checks of the exit status from these operations would be made, and an error message box would be created to inform the user if there was an error.

Also, certain shader features require information from the scene in order to be used with OpenSCAD. Functions that can be used for obtaining these requirements include [`glGetActiveAttrib`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glGetActiveAttrib.xhtml) and [`glGetActiveUniform`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glGetActiveUniform.xhtml). It is possible that existing code from older branches of the repository can be adapted to assist these integrations.

On the OpenSCAD side there will need to be the proper class structure to allow all the necessary information about the scene to be passed to shaders. The goal is for these interfaces to be scalable so that new features that unlock additional possibilities in the future can easily be integrated. The guiding principles that will allow this to be achieved are creating a single source of truth for each type of scene attribute that could be used by a shader, and adding useful abstractions that efficiently make the data available. Some examples of areas in the codebase currently where uniforms are sent to the shader are throughout the initialization steps in `Render.cc` for face and edge color. To support a greater range of custom shaders, the sources for other attributes will need to be aggregated. This discovery process will involve a mix of strategies such as lexical searches through the codebase, tracing execution with `gdb`, and consultation with veteran OpenSCAD developers who are knowledgeable about the location of specific elements.

Additionally, before a new shader replaces the current viewport shader, the [OpenGL runtime checks](https://www.khronos.org/registry/OpenGL-Refpages/es2.0/xhtml/glValidateProgram.xml) would be performed to ensure that the appropriate context is available for the shader. Ideally, any subsequent rendering exceptions that occur would still be caught and allow the user to revert to a different shader.

## Planned Resources

The relevant libraries and frameworks for this project are already dependencies for other parts of the OpenSCAD codebase. Here are the main tools:

- [OpenGL](https://www.khronos.org/opengl/)
- [Qt](https://code.qt.io/cgit/)

## Deliverables

The final result will include the set of changes that are necessary to add the new viewport shader features to OpenSCAD. Likely this will take the form of GitHub pull request(s) which contain all the additional classes, resources, tests, and documentation that represent a complete feature that could be integrated into the codebase.

Specifically, as outlined in the project description, my contributions would include the implementation of a new set of menu options which allow users to manage the shaders that can be used in the 3D viewport render.

## Development Schedule

| Date | Event |
| --- | --- |
| June 13 | Coding Begins |
| July 1 | Milestone 1: Backend proof of concept for viewport shader management |
| July 29 | Milestone 2: Official Google Evaluation deadline; "Is it on track towards a robust shader management structure?" |
| August 26 | Milestone 3: Usable UI for managing shaders, and complete interface for passing scene data to shaders |
| September 5 | Final Week: Code, documentation, and tests completed |