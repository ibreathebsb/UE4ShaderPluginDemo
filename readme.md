# Hello!

The project has now been updated to UE 5.3, using the RenderGraph API to stay in line with the rest of the engine.


There has also been several contributions and fixes to earlier branches however, and I'd be remiss not to thank everyone who has helped out!

@tomotaco
@thescratt
@darksylinc
@patlj09
@Umax1004
@ValentinKraft
@Boonto

------

This project is a tutorial project for how to create shaders in UE5. There are branches for some major versions, but not all. If you are working on an older version of the engine, please try and find the branch that best fits your needs! 

Most material effects can be created in-editor using the excellent tools that Epic has provided us with. There are some times though, where you simply want to work directly with the graphics API, or UE5's excellent RHI abstraction. This plugin attempts to help you get to that point as quickly as possible.
It is worth to note that this is not a tutorial on how to program shaders in general, or how to write HLSL, but rather how to get shaders working in UE5. For learning how to program shaders or HLSL, I recommend other resources,
such as MSDN: https://msdn.microsoft.com/en-us/library/bb509561(v=VS.85).aspx or instructional books for more advanced users such as "GPU Pro" by Wolfgang Engel. If you want to learn more about graphics in general, there are also excellent books on that subject. I like to recommend "Real-Time Rendering" from CRC.

------

**Writing shaders in UE5:**

It is possible to host your own shader files (called *.usf files in UE5) in your plugin, but it does take some extra work, and the support isn't always perfect. The language used in usf files are plain HLSL with some extra features, much like how Epic have adopted cpp in engine code.

**Shader compilation:**

When you have placed your shader code in the correct folder, you need to compile it. This is done using a macro in your Cpp source called IMPLEMENT_SHADER_TYPE (and variations). This macro will tell the engine to instantiate a shader given some usf file, but since this then needs to be linked directly into the rendering module, it has to be done very early in the engine startup procedure. The best way to do this is to host the shader declaration in a plugin with its load phase property set to "PostConfigInit".

**Shader parameters:**

Adding parameters to your shaders is very easy! You can just add a parameter struct to your shader type and then call SetShaderParameters(). No more need for global uniform buffers (unless you want to :3).

**Shader invocation:**

When you have declared your shader in the appropriate plugin, you can now start up your project and start using it. Shaders in UE5 have to be invoked on the rendering thread, to hopefully nobody's surprise. There are two main ways of doing this. 

First off there is a macro called ENQUEUE_RENDER_COMMAND (and variations) that let you post a lambda function to be run at the render thread's first opportunity. This is not very deterministic though, and it can be hard to ensure that it is run exactly once per frame. It is excellent when you have some one-time task you just want to get done though.

If we have something we do want to run once per frame though, we can subscribe to one of the render module hooks. This is what we demonstrate in this plugin.

Whether we use the ENQUEUE_RENDER_COMMAND functions or a render module hook, we will end up with a callback function where we are on the render thread with access to the immediate RHI command list as well as the render graph system. RHI stands for "Render Hardware Interface" which as the name suggests is a way for our code to be platform independent. Most function names on the RHI seem to be inspired by directx rather than any other API though, which might be good to know. If you are having trouble finding a particular function, the best way is therefore to check MSDN before any OpenGL resources or otherwise.
As of 5.3, this demo project mainly showcases how to interact with the RenderGraph API as that's what most of the engine uses these days. It simplifies many things, like when to perform memory allocations, and figuring out shader dependencies for us. I would recommend avoiding using the immediate command list unless you absolutely have to. Also, if the functions you are looking for are only available on the immediate API, consider adding a render graph pass for it instead! There is an example of this in the PixelShader example for you to peruse!

**Shader output:**

After you have run your shader it is of course time to harvest your output. There is no special UE5 magic to this step as we simply elect to draw to a UObject based render target that we are then able to consume from other UE5 code.
There is also good support for buffer readbacks, and there is a small demo of this in the ComputeShader demo file.

**Rendering resource types:**

There is a caveat when it comes to UE5 rendering resource types though. They come in generally 3 different flavors; UObject render resources (like UTexture), pooled render resources (like IPooledRenderTarget and their new render graph wrappers like FRDGTexture) and low level render resources (like FRHITexture). In some situations you can get these resource types to talk to each other via the low level types. As in, you can access an RHI texture both from the pooled render targets and UTextures. But unfortunately, this ends up being quite limited. There is for example no way (that I know of) to get a UTexture to interact directly with a rendering graph without doing an annoying resource copy somewhere.

**How to run this project:**

To get the project to run, you first need to bind it to an engine, you do this like any other project by right clicking the uproject file and switching version if needed, and then doing Generate Project.
You then need to open up the solution file and build the source code.

**How to use this project:**

I would recommend checking out the example project supplied in this repository to understand how to best use the project code. All the relevant files in the repository are:
* Everything under the Plugins/ folder                     (For declaring the shaders and running rendering code)
* Source/ShaderPluginDemo/ShaderPluginDemoCharacter.cpp/.h (This code shows how to consume the plugin code)
* Everything under the Content/ShaderPluginDemo/ folder    (These are the editor objects that I use to set up the shader use in the scene)
* The project settings file                                (I have created some new input bindings)

**Project controls:**

W/A/S/D - Movement
Space - Jump
Move mouse - Look around
Left mouse button - Paint the object you are aiming at with the output of the compute shader/pixel shader chain
Q/E - Change the blend amount on the pixel shader. Q moves the blend closer to the pixel shader simple gradient, while E moves the blend closer to the compute shader output.

I hope someone finds this useful :)

Best regards,
Temaran
