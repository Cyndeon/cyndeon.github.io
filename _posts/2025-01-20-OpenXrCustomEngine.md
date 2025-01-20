---
title: How I implemented OpenXR rendering into a custom engine in C++
date: 2025-01-20 18:33:00 +/-0100
categories:
  - Vr
  - Gamedev
tags:
  - cpp
  - openxr
  - opengl
  - programming
  - entt
description: A blog about how to implement OpenXR rendering into a custom C++ engine.
toc: true
---
# This blog post is unfinished and still a work in progress!

### Intro:

I am Justin, a second year student at BUAS and for the past 7 weeks, I have been working on implementing OpenXR into a custom engine made by the lecturers here called "Bee".
During the process of implementing OpenXR I have learned a lot about not only OpenXR, but also OpenGL, which is the rendering library that Bee uses. Of course, none of this hasn't been without running into many obstacles. In this blog post I will explain how I implemented OpenXR and combined it with the pre-existing Bee library. I will go over what my original plans were, how I got the required scripts to run, how I made the rendering using the OpenXR swapchains combined with OpenGL and much more!

It is worth keeping in mind that I used my Rift S to test any code that I wrote. While OpenXR does support most, if not all major headsets, some things might work differently based on the headset you use.

For those wishing to skip straight to the end and/or take a look at my code, I below will be the entire class. Do keep in mind that there are some variables and functions for OpenXR's input that do not work as of writing this! The files are unfiltered and will likely contain things your project might not.
[VrManager](../assets/vrmanager.rar)

#### Requirements:
- Visual Studio (or another environment for programming)
- VR headset compatible with OpenXR (I used a Rift S for this)
- An engine in C++ (At the very least, it should be able to open a window and render graphics to a framebuffer, with a rendering function that we can call manually in our script. The framebuffer will also have to be accessible. It also will need to use Entt or you will have to modify the code to remove the Entt parts)

##### First steps:

So the first step to any project, deciding on what you want to make and setting up some milestones for yourself. These don't always have to be time-bound, but in my case, since this is for school, I have 1 milestone every 2 weeks excluding the first week, so week 3, 5 and 7 all had milestones attached to them.
For me, the end result was that I wanted to be able to render a scene to my VR headset and be able to move around some physics objects with my hands. I also wanted to add some basic movement, which at the time I hadn't exactly decided on yet, but I did know that simple walking around when moving the thumbstick would be the easiest and thus most likely to be implemented. For being able to use physics, I was going to use the Jolt library, which is a library can deal with the physics calculations.
Now, there are multiple libraries that can be used to implement VR into a custom engine in C++. The two main ones I looked at were OpenVR (https://github.com/ValveSoftware/openvr) and the C++ version of OpenXR (https://github.com/KhronosGroup/OpenXR-Hpp).
I did some research on this topic (which isn't releveant here) and in the end, I decided upon using OpenXR. I had worked with it in the past, though that was in Unity and using C#.
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

##### Implementing OpenXR
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