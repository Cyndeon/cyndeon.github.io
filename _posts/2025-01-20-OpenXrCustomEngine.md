---
title: How I implemented OpenXR rendering into a custom engine in C++
date: 2025-01-20 18:33:00 +/-0100
categories:
  - Programming
  - Virtual-Reality
tags:
  - cpp
  - openxr
  - opengl
description: A blog about how to implement OpenXR rendering into a custom C++ engine.
toc: true
media_subpath: /assets/
---
## Intro

I am Justin, a second year student at BUAS and for the past 7 weeks, I have been working on implementing OpenXR into a custom engine made by the lecturers here called "Bee".
During the process of implementing OpenXR I have learned a lot about not only OpenXR, but also OpenGL, which is the rendering library that Bee uses. Of course, none of this hasn't been without running into many obstacles. In this blog post I will explain how I implemented OpenXR and combined it with the pre-existing Bee library. I will go over what my original plans were, how I got the required scripts to run, how I made the rendering using the OpenXR swapchains combined with OpenGL and much more!

It is worth keeping in mind that I used my Rift S to test any code that I wrote. While OpenXR does support most, if not all major headsets, some things might work differently based on the headset you use.

For those wishing to skip straight to the end and/or take a look at my code, I below will be the entire class. Please keep in mind that there are some variables and functions for OpenXR's input that do not work as of writing this! The files are unfiltered and will likely contain things your project might not, though, you could always check these files to see the full versions of what I will explain here in this post.
[VrManager Class](https://github.com/Cyndeon/cyndeon.github.io/blob/main/assets/vrmanager.rar)

### Showcase
Here is a little demo of what it will look like in the end. I do have some models rendering here but it still shows nicely of what it will look like at the end. 
|{% include embed/video.html src='{VR.mp4}' %}|
## Requirements
- Visual Studio (or another environment for programming)
- VR headset compatible with OpenXR
- An engine in C++ (At the very least, it should be able to open a window and render graphics to a framebuffer, with a rendering function that we can call manually in our script. The framebuffer will also have to be accessible and the size needs to be able to be set before creation)

## First steps

So the first step to any project, deciding on what you want to make and setting up some milestones for yourself. These don't always have to be time-bound, but in my case, since this is for school, I have 1 milestone every 2 weeks excluding the first week, so week 3, 5 and 7 all had milestones attached to them.
For me, the end result was that I wanted to be able to render a scene to my VR headset and be able to move around some physics objects with my hands. I also wanted to add some basic movement, which at the time I hadn't exactly decided on yet, but I did know that simple walking around when moving the thumbstick would be the easiest and thus most likely to be implemented. For being able to use physics, I was going to use the Jolt library, which is a library can deal with the physics calculations.
Now, there are multiple libraries that can be used to implement VR into a custom engine in C++. The two main ones I looked at were [OpenVR](https://github.com/ValveSoftware/openvr) and the C++ version of [OpenXR](https://github.com/KhronosGroup/OpenXR-Hpp).
I did some research on this topic (which isn't relevant here) and in the end, I decided upon using OpenXR. I had worked with it in the past, though that was in Unity and using C#.
Below here will show my original milestones as well as what changed over time due to circumstances.
```
Week 3 milestone: Implement the OpenXR library for using the VR capabilities and have the 3D sample scene rendered to the headset

Week 5 milestone: Have head tracking which allows for controlling the camera by moving the headset.
Also implement controllers tracking, moving their in-game counterparts when they are moved

Week 7 milestone: Implement controller input and then implement Jolt physics for a small playable demo with physics objects to play with in VR,
this includes being able to grab/throw objects as well as interact with some by pressing the trigger (button on back of Rift S controller)
Players will also be able to move and look around using the joysticks
```
And this is how it looked at the end:
```
Week 3 milestone: Implement the OpenXR library for using the VR capabilities and have something rendered to the headset

Week 5 milestone: Allow headset to render a scene (background + model or cube) Have head tracking which allows for controlling the camera by moving the headset. Also implement controllers tracking, moving their in-game counterparts when they are moved

Week 7 milestone: Implement controller input and then implement Jolt physics for a small playable demo with physics objects to play with in VR, this includes being able to grab/throw objects as well as interact with some by pressing the trigger (button on back of Rift S controller) Players will also be able to move and look around using the joysticks.
```

As one might be able to tell, I got stuck on the rendering part for a good while before finally getting it to work properly and I will go into more detail when we get to the rendering step.
As one can also see, the milestones had changed a lot since when I first started compared to when the project was over. Plans change, stuff like this happens, mostly when researching subjects that one does not have any knowledge of. New things will most likely take a lot of time to do properly and thus, the milestones should evolve over time to better and more accurately reflect one's progress.

## Implementing OpenXR
So, the goals have been set, now to implement the library and get to work!
Personally, I downloaded the NuGet package with all the libraries. The package is called "OpenXR.Headers" by Khronos Group. I used version 1.0.10.2. Installing this package should also download OpenXR.Loader automatically.

After downloading that, it is time to start programming.
The class name I decided on was VrManager, since it will be managing all of the VR-related functionality.

For those using the NuGet Package of OpenXr, I will suggest putting the following code below your includes but above the namespace and classes:
```cpp
#define OPENXR_CHECK(x, y)                                                                                       \
    {                                                                                                            \
        XrResult result = (x);                                                                                   \
        if (!XR_SUCCEEDED(result))                                                                               \
        {                                                                                                        \
            std::cerr << "ERROR: OPENXR: " << int(result) << "("                                                 \
                      << (m_instance ? GetXRErrorString(m_instance, result) : "")                                \
                      << ") " << y << std::endl; \
        }                                                                                                        \
    }
```
Then, in your class header file (i'd recommend making this one private):
```cpp
inline const char* GetXRErrorString(XrInstance xrInstance, XrResult result)
{
    static char string[XR_MAX_RESULT_STRING_SIZE];
    xrResultToString(xrInstance, result, string);
    return string;
}
```
This will help you log any errors you may find along the way. I implemented this pretty late into the project, so it is likely that this will not be required for most if not all steps, however, it is still really helpful to have and some of you may try to challenge yourself by seeing if you can change my slightly messy code to look cleaner with the help of this function!

Now onto the actual interesting subjects.

The header will be explained first.

### Includes

First the includes. In my header I include the following:
```cpp
#pragma once
#include <string>
#include <vector>
// Define platform and graphics API
#define XR_USE_PLATFORM_WIN32
#define XR_USE_GRAPHICS_API_OPENGL

#include <openxr/openxr.h>
#include <openxr/openxr_platform.h>

#include <entt//entity/entity.hpp>

#include <glad/glad.h>
#include "glm/vec3.hpp"
#include "glm/vec4.hpp"
#include "glm/mat4x4.hpp"
#include <glm/gtc/type_ptr.hpp>

#include <iostream>
```

As shown here, I include some basic C++ things like strings and vectors, but also OpenXR and some Glad for math, you are free to use whatever math library you'd like, though you might have to create your own conversion functions to accommodate for this. 

### Variables

After that, we will need a couple of variables, so it should look something like this:
```cpp
struct Dimensions
{
    int width;
    int height;
};
struct XrMatrix4x4f
{
    float m[16];
};
class VrManager
{		
private:
	XrInstance m_instance = XR_NULL_HANDLE;
	XrSystemId m_systemId = 0;
	XrSession m_session = XR_NULL_HANDLE;
		
	Renderer* m_renderer; // this will be a pointer to your own rendering class

	std::vector<XrSwapchain> m_swapchains;
	std::vector<std::vector<XrSwapchainImageOpenGLKHR>> m_swapchainImages;
	std::vector<Dimensions> m_swapchainDimensions;
	std::vector<XrViewConfigurationView> m_viewConfigViews;
	std::vector<XrViewConfigurationType> m_viewConfigTypes = {XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO, XR_VIEW_CONFIGURATION_TYPE_PRIMARY_MONO};
	XrViewConfigurationType m_viewConfig = XR_VIEW_CONFIGURATION_TYPE_MAX_ENUM;
	XrSpace m_referenceSpace;
	XrSessionState m_sessionState = XR_SESSION_STATE_UNKNOWN;
	uint32_t m_viewCount = 0;
	std::vector<XrView> m_views;
}
```
This might look a bit intimidating at first, but when you take a closer look at it, it's not that difficult.
First of all, the instance and session are vital for the operation of OpenXR in general. The instance "encompasses the application setup state, OpenXR API version and any layers and extensions" while the session itself keeps track of the currently running session while the sessionState keeps will be set to the state of the session.

Then you have a bunch of vectors filled with data related to the swapchains, their images and dimensions. The swapchains are very similar to framebuffers, but in this case each eye has a certain amount of swapchain images assigned to it, in my case for the Rift S, each eye has 3 images, meaning I have 6 in total. These images will be swapped out during the next frame " in order to create the illusion of motion within the image."
That is also why the images are vectors inside of a vector, one vector stores each eye and the other one stores the images inside of the eye.

After that, there are the view configurations, they have to do with the way that the camera is rendered in preparation for the swapchainImages.

There is also the XrSpace, this is a class that has to do with the VR tracking in the real world and converting it to data that OpenXR can use, it basically sort of keeps track of your real life space in that sense.

I also feel like I should mention I have a pointer to a Renderer class, for me, like I explained in the [Requirements](#requirements), I already have a custom engine with a rendering system built-in. For you, this will have to be a function that renders to a framebuffer and then that framebuffer has to be accessible in the VrManager.


Now that we have all the variables we are going to need, we can start putting them to use! We'll need some functions to do that, so let's add those first.
```cpp
struct Dimensions
{
    int width;
    int height;
};
struct XrMatrix4x4f
{
    float m[16];
};
class VrManager
{	
public
	VrManager();
	~VrManager();
		
	bool Initialize();
	void Update();

private:
	void Render();
	bool CreateInstance();
	void GetInstanceProperties();
	void GetViewConfigurationViews();
	bool GetSystem();
	bool CreateSession();
	bool BeginSession();
	bool CreateSwapchains();
	void PollEvents();

    // math / conversion functions
	 inline glm::vec3 XrVec3ToGLMVec3(XrVector3f xrVec) { return glm::vec3(xrVec.x, xrVec.y, -xrVec.z); }
    inline glm::vec4 XrQuatToGLMVec4(XrQuaternionf xrQuat) { return glm::vec4(xrQuat.x, xrQuat.y, xrQuat.z, xrQuat.w); }
	 inline static void XrMatrix4x4f_To_glm_mat4x4(glm::mat4& result, XrMatrix4x4f xrmatrix4x4f)
    {
        result = glm::make_mat4(xrmatrix4x4f.m);
    }

    void XrMatrix4x4f_CreateProjectionFov(XrMatrix4x4f* result, const XrFovf fov, const float nearZ, const float farZ);
    void XrMatrix4x4f_CreateProjection(XrMatrix4x4f* result,
                                       const float tanAngleLeft,
                                       const float tanAngleRight,
                                       const float tanAngleUp,
                                       float const tanAngleDown,
                                       const float nearZ,
                                       const float farZ);

		// variables
	XrInstance m_instance = XR_NULL_HANDLE;
	XrSystemId m_systemId = 0;
	XrSession m_session = XR_NULL_HANDLE;
		
	Renderer* m_renderer; // this will be a pointer to your own rendering class

	std::vector<XrSwapchain> m_swapchains;
	std::vector<std::vector<XrSwapchainImageOpenGLKHR>> m_swapchainImages;
	std::vector<Dimensions> m_swapchainDimensions;
	std::vector<XrViewConfigurationView> m_viewConfigViews;
	std::vector<XrViewConfigurationType> m_viewConfigTypes = {XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO, XR_VIEW_CONFIGURATION_TYPE_PRIMARY_MONO};
	XrViewConfigurationType m_viewConfig = XR_VIEW_CONFIGURATION_TYPE_MAX_ENUM;
	XrSpace m_referenceSpace;
	XrSessionState m_sessionState = XR_SESSION_STATE_UNKNOWN;
	uint32_t m_viewCount = 0;
	std::vector<XrView> m_views;
}
```
Now we've got some functions to properly set up and use the variables we just created. What these functions do is relatively self-explanatory based on the names of the variables, but we'll get to that soon enough.

### Creating the functions
I will start from the top and work my way through each function, one at a time.
If you would prefer to create one function at a time, you can click the headers on the side of screen and jump between them.


#### VrManager() (constructor)
First, the constructor, it's very short and simple. I call the Initialize() function that will set up everything for OpenXR, after that I create the pointer to my renderer but I also give the renderer the dimensions of my swapchain, this is because the framebuffer that the renderer creates should have the same size as the swapchainImages, since those are the ones that will be displayed to the user. I also use the first element in the array which is the left eye, it does not matter whether element 0 or 1 is used since both will have the same dimensions.
```cpp
VrManager::VrManager()
{
    Initialize();
    m_renderer = &Engine.CreateRenderer(m_swapchainDimensions[0].width, m_swapchainDimensions[0].height);
}
```

#### Initialize() 
Since the Initialize() function is rather big, I will go through it step by step to make it more digestible.

```cpp
bool result = CreateInstance();
if (!result)
{
    std::cerr << "Instance creation failed" << std::endl;
    return false;
}

result = GetSystem();
if (!result)
{
    std::cerr << "System detection failed" << std::endl;
    return false;
}
```
First, the essentials. We create the instance and get the system details, as you can see here, I also have a boolean for result, if at any point, result returns false, something went wrong, we log it, and quit the setup. If your program would require some specific actions to happen if this setup fails, you can modify the constructor to do something if Initialize() returns false.

```cpp
GetViewConfigurationViews();

XrSystemProperties systemProperties = {XR_TYPE_SYSTEM_PROPERTIES};
result = xrGetSystemProperties(m_instance, m_systemId, &systemProperties);
if (result != XR_SUCCESS)
{
    std::cerr << "System Properties Not Found" << std::endl;
}
else
{
    XrSystemGraphicsProperties graphicsProperties = systemProperties.graphicsProperties;
        std::cout << "Max swapchain width: " << graphicsProperties.maxSwapchainImageWidth << '\n';
        std::cout << "Max swapchain height: " << graphicsProperties.maxSwapchainImageHeight << '\n';
}

XrGraphicsRequirementsOpenGLKHR graphicsRequirements = {XR_TYPE_GRAPHICS_REQUIREMENTS_OPENGL_KHR};
PFN_xrGetOpenGLGraphicsRequirementsKHR pfnGetOpenGLGraphicsRequirementsKHR = NULL;
result = xrGetInstanceProcAddr(m_instance,
                               "xrGetOpenGLGraphicsRequirementsKHR",
                               (PFN_xrVoidFunction*)&pfnGetOpenGLGraphicsRequirementsKHR);

if (result == XR_SUCCESS && pfnGetOpenGLGraphicsRequirementsKHR)
{
    result = pfnGetOpenGLGraphicsRequirementsKHR(m_instance, m_systemId, &graphicsRequirements);

    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to get OpenGL graphics requirements" << '\n';
    }
}
```
Here we set up the configuration views, get the system's properties and we also make sure that the graphics requirements are met for the device to be able to run VR.

```cpp
 result = CreateSession();
 if (!result)
 {
     std::cerr << "Session creation failed" << std::endl;
     return false;
 }

 result = CreateSwapchains();
 if (!result)
 {
     std::cerr << "Swapchain creation failed" << std::endl;
     return false;
 }

 return true;
```
Lastly, we'll create the session, set up the swapchains and finally return true if nothing went wrong during the entire setup.

Now that we have set all of that up, it is time to fill those functions!

#### CreateInstance()
```cpp
bool VrManager::CreateInstance()
{
    const char* extensions[] = {"XR_KHR_opengl_enable"};

    XrInstanceCreateInfo createInfo = {XR_TYPE_INSTANCE_CREATE_INFO};
    createInfo.enabledExtensionCount = 1;
    createInfo.enabledExtensionNames = extensions;
    strcpy(createInfo.applicationInfo.applicationName, "VR Application");
    createInfo.applicationInfo.apiVersion = XR_CURRENT_API_VERSION;

    XrResult result = xrCreateInstance(&createInfo, &m_instance);
    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to create OpenXR instance" << std::endl;
        return false;
    }
    return true;
}
```
In this function, we create the instance for OpenXR, this is also where extra extensions can be enabled for specific rendering libraries such as OpenGL (like we are using in this example), OpenGL_ES or Vulkan, but also more platform specific ones that can enable hand-tracking, body tracking, face tracking and more, if your device can use those functions of course.
For more information about what extensions are available, [there is a list of them available on their site here.](https://registry.khronos.org/OpenXR/specs/1.1/html/xrspec.html?extension-appendices-list#extension-appendices-list)

#### GetSystem()
```cpp
bool VrManager::GetSystem()
{
    XrSystemGetInfo systemInfo = {XR_TYPE_SYSTEM_GET_INFO};
    systemInfo.formFactor = XR_FORM_FACTOR_HEAD_MOUNTED_DISPLAY;

    XrResult result = xrGetSystem(m_instance, &systemInfo, &m_systemId);
    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to get XR system, make sure a headset is connected" << std::endl;
        return false;
    }
    return true;
}
```
Here we simply ask for the headset's information and ID we'll need to use later.

#### GetViewConfigurationViews()
This one is a little larger so let's go through this step by step
```cpp
void VrManager::GetViewConfigurationViews()
{
    // Gets the View Configuration Types. The first call gets the count of the array that will be returned. The next call fills
    // out the array.
    uint32_t viewConfigurationCount = 0;
    xrEnumerateViewConfigurations(m_instance, m_systemId, 0, &viewConfigurationCount, nullptr);
    m_viewConfigTypes.resize(viewConfigurationCount);
    xrEnumerateViewConfigurations(m_instance,
                                  m_systemId,
                                  viewConfigurationCount,
                                  &viewConfigurationCount,
                                  m_viewConfigTypes.data());
```
First we enumerate the configurations to see how many there need to be, that is why we give a nullptr the first time around. We then resize the vector we want to fill with the amount of view configurations there will be before enumerating again and now filling the vector with all these configurations. This "enumerate twice, once for size, once for contents" will be a recurring subject throughout the setup process.

```cpp
 // Pick the first application supported View Configuration Type con supported by the hardware.
 for (const XrViewConfigurationType& viewConfiguration : m_viewConfigTypes)
 {
     if (std::find(m_viewConfigTypes.begin(), m_viewConfigTypes.end(), viewConfiguration) != m_viewConfigTypes.end())
     {
         m_viewConfig = viewConfiguration;
         break;
     }
 }
 if (m_viewConfig == XR_VIEW_CONFIGURATION_TYPE_MAX_ENUM)
 {
     std::cerr << "Failed to find a view configuration type. Defaulting to XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO."
               << std::endl;
     m_viewConfig = XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO;
 }
```
Here we go through the view configurations and, like the comment states, find the first one that is supported and set the default view configuration (m_viewConfig) to that one. We then will check if m_viewConfig is still the default value we set it to (XR_VIEW_CONFIGURATION_TYPE_MAX_ENUM), in which case something likely went wrong with getting the configs, however, we can still set it to primary_stereo as the default since that is the view configuration that is most often and assuming everything else works, this should work regardless. It is a good thing to debug since this might cause issues later, but it isn't something that will make the entire thing not work, thus, a simple std::cerr rather than a return.

```cpp
 // Gets the View Configuration Views. The first call gets the count of the array that will be returned. The next call fills
 // out the array.
 uint32_t viewConfigurationViewCount = 0;
 xrEnumerateViewConfigurationViews(m_instance, m_systemId, m_viewConfig, 0, &viewConfigurationViewCount, nullptr);
 m_viewConfigViews.resize(viewConfigurationViewCount, {XR_TYPE_VIEW_CONFIGURATION_VIEW});
 xrEnumerateViewConfigurationViews(m_instance,
                                   m_systemId,
                                   m_viewConfig,
                                   viewConfigurationViewCount,
                                   &viewConfigurationViewCount,
                                   m_viewConfigViews.data());
```

#### CreateSession()
So everything thus far has been set up correctly and we are ready to create the session!
Another big function upcoming (that could potentially be split up into smaller functions if one would prefer that).
```cpp
    // Verify OpenGL context exists
    HGLRC currentContext = wglGetCurrentContext();
    HDC currentDC = wglGetCurrentDC();

    if (currentContext == NULL || currentDC == NULL)
    {
        std::cerr << "No active OpenGL context found!" << std::endl;
        std::cerr << "Current Context: " << currentContext << ", Current DC: " << currentDC << std::endl;
        return false;
    }

    // Get OpenGL version
    const char* glVersion = reinterpret_cast<const char*>(glGetString(GL_VERSION));
    const char* glRenderer = reinterpret_cast<const char*>(glGetString(GL_RENDERER));

    std::cout << "OpenGL Version: " << (glVersion ? glVersion : "Unknown") << std::endl;
    std::cout << "OpenGL Renderer: " << (glRenderer ? glRenderer : "Unknown") << std::endl;

    // Create the graphics binding structure
    XrGraphicsBindingOpenGLWin32KHR graphicsBinding = {XR_TYPE_GRAPHICS_BINDING_OPENGL_WIN32_KHR};
    graphicsBinding.hDC = wglGetCurrentDC();
    graphicsBinding.hGLRC = wglGetCurrentContext();

    XrSessionCreateInfo sessionCreateInfo = {XR_TYPE_SESSION_CREATE_INFO};
    sessionCreateInfo.systemId = m_systemId;
    sessionCreateInfo.next = &graphicsBinding;
```
First we have some functions set up to make sure OpenGL exists, this part is specific to this project since it uses OpenGL and thus will be different for other rendering libraries. the last part also is specific to my device as you can tell, it is for OpenGL and Windows, thus, this might also be different, keep that in mind!
The last part also sets up the graphics binding, basically telling the session what is next in the structure chain, this is something that "in most cases one graphics API extension specific struct needs to be in this next chain." [as stated here.](https://registry.khronos.org/OpenXR/specs/1.0/html/xrspec.html#XrSessionCreateInfo)

```cpp
 XrResult result = xrCreateSession(m_instance, &sessionCreateInfo, &m_session);
 if (result != XR_SUCCESS)
 {
     std::cerr << "Failed to create session. Error code: " << result << std::endl;

     // Additional error diagnostics
     switch (result)
     {
         case XR_ERROR_GRAPHICS_DEVICE_INVALID:
             std::cerr << "Graphics device is invalid" << std::endl;
             break;
         case XR_ERROR_RUNTIME_FAILURE:
             std::cerr << "Runtime failure during session creation" << std::endl;
             break;
         default:
             std::cerr << "Unknown session creation error" << std::endl;
     }
     return false;
 }
```
Then we actually create the sources, quite straight-forward. We attempt to create a session, and if it goes wrong, we get slightly detailed information about what exactly went wrong, whether it is the graphics, runtime or a different error.

```cpp
 // Create reference space
 XrReferenceSpaceCreateInfo referenceSpaceInfo = {XR_TYPE_REFERENCE_SPACE_CREATE_INFO};
 referenceSpaceInfo.referenceSpaceType = XR_REFERENCE_SPACE_TYPE_LOCAL;
 referenceSpaceInfo.poseInReferenceSpace.orientation.w = 1.0f;
 referenceSpaceInfo.poseInReferenceSpace.position = {0.0f, 0.0f, 0.0f};

 result = xrCreateReferenceSpace(m_session, &referenceSpaceInfo, &m_referenceSpace);
 if (result != XR_SUCCESS)
 {
     std::cerr << "Failed to create reference space" << std::endl;
     return false;
 }
```
Next we will create the reference space, this will give the user the data that they need in order to position the player properly. In this case, we will be using a local reference space, since this still does most of the tracking properly for us, but can also auto-adjust slightly, which is more helpful for a seated experience and also if the user temporarily loses tracking or light or something, the program can still attempt to adjust itself.
There are also View and Stage spaces, both of which have their own uses, [which are worth checking out here.](https://registry.khronos.org/OpenXR/specs/1.0/html/xrspec.html#XrSpace)

```cpp
 // Explicitly begin the session
 XrSessionBeginInfo sessionBeginInfo = {XR_TYPE_SESSION_BEGIN_INFO};
 sessionBeginInfo.primaryViewConfigurationType = XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO;

 result = xrBeginSession(m_session, &sessionBeginInfo);
 if (result != XR_SUCCESS)
 {
     std::cerr << "Failed to begin session. Error code: " << result << std::endl;
     return false;
 }

 return true;
```
Lastly, we will begin the actual session, this will allow us to start actually doing our rendering and other VR things (if you'd implement those, we only do rendering for this post).

#### CreateSwapChains()
```cpp
// Query the number of views required for the selected configuration
uint32_t viewCountOutput;
XrResult result =
    xrEnumerateViewConfigurationViews(m_instance, m_systemId, m_viewConfigTypes[0], 0, &viewCountOutput, nullptr);
if (result != XR_SUCCESS)
{
    std::cerr << "Failed to enumerate view configuration views" << std::endl;
    return false;
}

std::vector<XrViewConfigurationView> viewConfigViews(viewCountOutput, {XR_TYPE_VIEW_CONFIGURATION_VIEW});
result = xrEnumerateViewConfigurationViews(m_instance,
                                           m_systemId,
                                           m_viewConfigTypes[0],
                                           viewCountOutput,
                                           &viewCountOutput,
                                           viewConfigViews.data());
if (result != XR_SUCCESS)
{
    std::cerr << "Failed to retrieve view configuration views" << std::endl;
    return false;
}
    // Update the view count and resize necessary storage
    m_viewCount = viewCountOutput;
    m_views.resize(m_viewCount, {XR_TYPE_VIEW});
    m_swapchains.resize(m_viewCount);
    m_swapchainDimensions.resize(m_viewCount);
    m_swapchainImages.resize(m_viewCount);
```
Here we gather data for the view configuration views, this might seem a bit odd since we did something similar before, however, these ones are the views, rather than the configurations and types which is what we did before in the GetViewConfigurationViews(). Once again, we get the amount of viewConfigViews and then fill that vector with the data we gathered, if anything goes wrong, we will log this.

```cpp
// Create swapchains for each view
for (uint32_t i = 0; i < m_viewCount; ++i)
{
    const auto& viewConfig = viewConfigViews[i];

    XrSwapchainCreateInfo swapchainInfo = {XR_TYPE_SWAPCHAIN_CREATE_INFO};
    swapchainInfo.usageFlags = XR_SWAPCHAIN_USAGE_COLOR_ATTACHMENT_BIT | XR_SWAPCHAIN_USAGE_SAMPLED_BIT;
    swapchainInfo.format = GL_RGBA8;  // Verify this format with your OpenGL context
    swapchainInfo.sampleCount = 1;
    swapchainInfo.width = viewConfig.recommendedImageRectWidth;
    swapchainInfo.height = viewConfig.recommendedImageRectHeight;
    swapchainInfo.faceCount = 1;
    swapchainInfo.arraySize = 1;
    swapchainInfo.mipCount = 1;

    result = xrCreateSwapchain(m_session, &swapchainInfo, &m_swapchains[i]);
    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to create swapchain for view " << i << std::endl;
        return false;
    }

    // Store the dimensions
    int width = swapchainInfo.width;
    int height = swapchainInfo.height;
    m_swapchainDimensions[i] = {width, height};

    uint32_t imageCount;
    result = xrEnumerateSwapchainImages(m_swapchains[i], 0, &imageCount, nullptr);
    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to enumerate swapchain images for view " << i << std::endl;
        return false;
    }

    // Resize the inner vector to hold images for this view
    m_swapchainImages[i].resize(imageCount, {XR_TYPE_SWAPCHAIN_IMAGE_OPENGL_KHR});

    // Retrieve the images
    result = xrEnumerateSwapchainImages(m_swapchains[i],
                                        imageCount,
                                        &imageCount,
                                        reinterpret_cast<XrSwapchainImageBaseHeader*>(m_swapchainImages[i].data()));
    if (result != XR_SUCCESS)
    {
        std::cerr << "Failed to retrieve swapchain images for view " << i << std::endl;
        return false;
    }
}
```
This for-loop sets up the swap chains for us, it gets the information it requires, such as the width and height of the screens, the images that need to be prepared and fills the swapchainImages with the data for that eye.
Do keep in mind that depending on what device you are using, you may have more or less images per eye. For the Rift S for instance, I get 3 per eye, meaning that the swapchainImages vector contains 2 vectors, 1 per eye, each with 3 images each.

```cpp
 // Ensure that the OpenGL context and swapchain images are compatible
 for (uint32_t i = 0; i < m_viewCount; ++i)
 {
     for (const auto& image : m_swapchainImages[i])
     {
         // if image has not been set up properly, set it up
         if (!glIsTexture(image.image))
         {
             std::cerr << "Swapchain image for view " << i << " is not a valid OpenGL texture: " << image.image << std::endl;
             return false;
         }
     }
 }

 return true;
```
This last part is to make sure that the images are compatible with OpenGL's textures. This is not required and it should all work just fine without this, but this is more like a failsafe just in case something went wrong with OpenGL's setup for OpenXR.


So, that was it, the entire Initialize() function should now no longer give you any errors for functions that aren't found!

#### Update()
```cpp
void VrManager::Update()
{
    Render();
}
```
In the update function, we only render. Usually in here you might want to do things like doing the controller tracking, or other VR functionality.

#### PollEvents()
```cpp
void VrManager::PollEvents()
{
    XrEventDataBuffer eventData = {XR_TYPE_EVENT_DATA_BUFFER};
    while (xrPollEvent(m_instance, &eventData) == XR_SUCCESS)
    {
        switch (eventData.type)
        {
            case XR_TYPE_EVENT_DATA_SESSION_STATE_CHANGED:
            {
                auto stateEvent = reinterpret_cast<XrEventDataSessionStateChanged*>(&eventData);
                m_sessionState = stateEvent->state;

                // Handle session state transitions
                if (m_sessionState == XR_SESSION_STATE_READY)
                {
                    XrSessionBeginInfo beginInfo = {XR_TYPE_SESSION_BEGIN_INFO};
                    beginInfo.primaryViewConfigurationType = XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO;
                    xrBeginSession(m_session, &beginInfo);
                }
                else if (m_sessionState == XR_SESSION_STATE_STOPPING)
                {
                    xrEndSession(m_session);
                }
                break;
            }

            default:
                return;
        }
    }
}
```
Since the render function will be the biggest function, let's first take care of the PollEvents function.
In here, we basically request OpenXR for any potential event, things that have changed since we last checked. We only need to handle 1 type of event, which is if the session's state has changed, but this function could also be used for input, if there are events that are lost or some other events, for more events, [you can check this page here.](https://registry.khronos.org/OpenXR/specs/1.1/man/html/xrPollEvent.html)
What we do once a session state has changed, is that we check what the current state of the session is, if it is ready, we begin a session, like we did earlier. If it is stopping, we end the session.


Next up, the fun stuff:
#### Render()
Here's where the magic truly starts (well not really, we are programmers, not magicians), we set up OpenXR with all the components required to render images, now it is time to actually render it, well, after we do some more checks of course!
```cpp
 // Poll OpenXR events
 PollEvents();

 // Exit if session is not in a renderable state
 if (m_sessionState != XR_SESSION_STATE_READY && m_sessionState != XR_SESSION_STATE_SYNCHRONIZED &&
     m_sessionState != XR_SESSION_STATE_VISIBLE && m_sessionState != XR_SESSION_STATE_FOCUSED)
 {
     return;
 }

 // Wait for the next frame
 XrFrameWaitInfo beginWaitInfo = {XR_TYPE_FRAME_WAIT_INFO};
 XrFrameState frameState = {XR_TYPE_FRAME_STATE};
 if (xrWaitFrame(m_session, &beginWaitInfo, &frameState) != XR_SUCCESS)
 {
     return;
 }

 // Begin the frame
 XrFrameBeginInfo beginInfo = {XR_TYPE_FRAME_BEGIN_INFO};
 if (xrBeginFrame(m_session, &beginInfo) != XR_SUCCESS)
 {
     return;
 }
```
First we will poll events, just to make sure nothing changed or went wrong and then we do some more checks. We make sure that the session state is ready, synchronized, visible and focussed, basically, the session state has to be ready for us to use OpenXR's systems.
Then we wait for the frame to be ready and then begin the frame.

```cpp
// Prepare to render each view
std::vector<XrCompositionLayerProjectionView> projectionViews;
std::vector<XrCompositionLayerBaseHeader*> layers;
if (!m_views.empty())
{
    projectionViews.resize(m_views.size());

    for (size_t i = 0; i < m_views.size(); ++i)
    {
        // Locate the views (headset position and orientation)
        XrViewLocateInfo viewLocateInfo = {XR_TYPE_VIEW_LOCATE_INFO};
        viewLocateInfo.viewConfigurationType = XR_VIEW_CONFIGURATION_TYPE_PRIMARY_STEREO;
        viewLocateInfo.displayTime = frameState.predictedDisplayTime;
        viewLocateInfo.space = m_referenceSpace;

        XrViewState viewState = {XR_TYPE_VIEW_STATE};
        uint32_t viewCountOutput;
        xrLocateViews(m_session, &viewLocateInfo, &viewState, (uint32_t)m_views.size(), &viewCountOutput, m_views.data());

        // Acquire swapchain image
        XrSwapchain swapchain = m_swapchains[i];
        XrSwapchainImageAcquireInfo acquireInfo = {XR_TYPE_SWAPCHAIN_IMAGE_ACQUIRE_INFO};
        uint32_t imageIndex;
        xrAcquireSwapchainImage(swapchain, &acquireInfo, &imageIndex);

        XrSwapchainImageWaitInfo waitInfo = {XR_TYPE_SWAPCHAIN_IMAGE_WAIT_INFO};
        waitInfo.timeout = XR_INFINITE_DURATION;
        xrWaitSwapchainImage(swapchain, &waitInfo);
```
Here we first create some vectors that will be filled with temporary data, our projectionViews and layers.
after that, we, as the comment suggests, locate the views and space, we basically ask OpenXR the position as well as rotation of the views. We can use this later to render both eyes separately, as they will each have a slight offset, mimicking the natural separation between human eyes to create the illusion of depth. We also wait for the swapchainImage to be acquired. The waitInfo.timeout "indicates how many nanoseconds the call **may** block waiting for the image to become available for writing", and thus, if restrictions on how long this is allowed to be waiting for need to be in place, you can set the duration to a specific amount.

```cpp
// Set up projection view
projectionViews[i] = {XR_TYPE_COMPOSITION_LAYER_PROJECTION_VIEW};
projectionViews[i].pose = m_views[i].pose;
projectionViews[i].fov = m_views[i].fov;
projectionViews[i].subImage.swapchain = swapchain;
projectionViews[i].subImage.imageRect.offset = {0, 0};
projectionViews[i].subImage.imageRect.extent = {m_swapchainDimensions[i].width, m_swapchainDimensions[i].height};
projectionViews[i].subImage.imageArrayIndex = 0;

// I looked at and compared Javier's code for how he renders the eyes and did it slightly differently
const glm::vec3 eyeWorldPos = XrVec3ToGLMVec3(m_views[i].pose.position);
const glm::vec4 eyeWorldRot = XrQuatToGLMVec4(m_views[i].pose.orientation);

for (const auto& [e, camera, cameraTransform] :
     bee::Engine.ECS().Registry.view<bee::Camera, bee::Transform>().each())
{
    // Rot and pos
    cameraTransform.SetRotation(glm::quat(eyeWorldRot.w, eyeWorldRot.x, eyeWorldRot.y, eyeWorldRot.z));
    cameraTransform.SetTranslation(eyeWorldPos);

    // Projection
    XrMatrix4x4f xrProj;
    XrMatrix4x4f_CreateProjectionFov(&xrProj, m_views[i].fov, 0.01f, 100.0f);
    XrMatrix4x4f_To_glm_mat4x4(camera.Projection, xrProj);
}
```
This code was from a classmate of mine, but I will explain it to the best of my abilities regardless.
First we set up the projection view for each eye, what this does is basically slightly offset the eye to be in the place it should. Rather than having one "view" like PC games, we have 2 views here that are slightly offset to, like I stated before, mimic how real eyes work as well. After receiving all that data, we have to convert it to something that OpenGL can use, thus the conversion to glm data types.
After that there is a for-loop that goes through each camera. I use Entt here, which is a library for managing entities and it might seem a little complicated for those unexperienced with it, but all it does is get my camera and its transform, nothing more. So if you are going to follow this tutorial by copy-pasting (which I can't really blame you for to be honest), make sure to remove the for-loop and replace it with a private variable for the projection that you save in the header and then modify here since the Camera class only has a glm::mat4 Projection, and that is it, although you will have to use this projection in your renderer, so keep that in mind. We set the camera's position (translation) and rotation and create the projection for it as well.

```cpp
m_renderer->Render();
```
Here I call my render function from my renderer. The data has all been set up properly and my renderer uses the projection from my camera.

```cpp
 // Copy the rendered image to the swapchain framebuffer
 GLuint sourceFramebuffer = m_renderer->GetFinalBuffer();
 glBindFramebuffer(GL_READ_FRAMEBUFFER, sourceFramebuffer);
 GLuint swapchainFramebuffer;
 glGenFramebuffers(1, &swapchainFramebuffer);
 glBindFramebuffer(GL_DRAW_FRAMEBUFFER, swapchainFramebuffer);

 glFramebufferTexture2D(GL_DRAW_FRAMEBUFFER,
                        GL_COLOR_ATTACHMENT0,
                        GL_TEXTURE_2D,
                        m_swapchainImages[i][imageIndex].image,
                        0);

 if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
 {
     std::cerr << "Swapchain framebuffer is not complete!" << std::endl;
 }

 glBlitFramebuffer(0,
                   0,
                   m_swapchainDimensions[i].width,
                   m_swapchainDimensions[i].height,  // Source dimensions
                   0,
                   0,
                   m_viewConfigViews[i].recommendedImageRectWidth,
                   m_viewConfigViews[i].recommendedImageRectHeight,
                   GL_COLOR_BUFFER_BIT,
                   GL_NEAREST);

 glBindFramebuffer(GL_FRAMEBUFFER, 0);
 glDeleteFramebuffers(1, &swapchainFramebuffer);
 
            // Release the swapchain image
XrSwapchainImageReleaseInfo releaseInfo = {XR_TYPE_SWAPCHAIN_IMAGE_RELEASE_INFO};
xrReleaseSwapchainImage(swapchain, &releaseInfo);
}
```
Here we get the framebuffer from our renderer that has been prepared with the image for this eye and blit that to our swapchain's framebuffer that is generated here. What blitting is, is just copying pixels from one source to another, so in this case, from our sourceFrameBuffer to our swapchainFrameBuffer and release that data to the swapchain.

```cpp
    // End the frame
    XrFrameEndInfo endInfo = {XR_TYPE_FRAME_END_INFO};
    endInfo.displayTime = frameState.predictedDisplayTime;
    endInfo.environmentBlendMode = XR_ENVIRONMENT_BLEND_MODE_OPAQUE;
    endInfo.layerCount = (uint32_t)layers.size();
    endInfo.layers = layers.empty() ? nullptr : layers.data();
    xrEndFrame(m_session, &endInfo);
}
```
After the eyes have been rendered and the swapchain images are prepared, we end the XrFrame with the data it needs.


#### Conclusion
And that's it!
If you start up the project, assuming all of this has been followed and your own code has been added for the rendering, it should now render to your headset!

If you have any questions or parts that do not work, do not hesitate to e-mail me!
I hope this post was helpful and gave you the help you needed to set up OpenXR for your project.
## Sources
- [Helpful example of a main script: https://github.com/KHeresy/openxr-simple-example/blob/master/main.cpp,](https://github.com/KHeresy/openxr-simple-example/blob/master/main.cpp)
- [Official tutorial: https://openxr-tutorial.com//windows/opengl/1-introduction.html#introduction](https://openxr-tutorial.com//windows/opengl/1-introduction.html#introduction)
- [Official site for extensions (and more): https://registry.khronos.org/OpenXR/specs/1.1/html/xrspec.html?extension-appendices-list#introductiont](https://registry.khronos.org/OpenXR/specs/1.1/html/xrspec.html?extension-appendices-list#introduction)