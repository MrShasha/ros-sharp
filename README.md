## Purpose of this fork ##
** Solved problem ** : While working on my project I needed to subscribe to such custom ROS messages, that did not have their C# class counterparts in the RosSharp library.
One way would be to simply add these new C# classes to the RosSharp library manually, but this would require me to update this external library every time I need a support for new message type.
Since this need is quite common in robotics projects, I decided to fork the RosSharp repository and add a feature that would allow to subscribe to a topic and receive messages of any type, without having to serialize them to a C# class for this type in the RosSharp library while only using json to transfer the information.

** How to use**: To subscribe to a topic with custom message type, use the new `SubscribeWithMessageTypeJObject` method of the `RosSocket` class. You can call it from your code for example like this:
```csharp
  public Task<string> SubscribeDynamicAsync(string topic, string messageType, Action<object> callback)
{
    if (!IsConnected)
    {
        _logger.LogError("Not connected.");
        return Task.FromResult(string.Empty);
    }

    try
    {
        _logger.LogInformation("Creating dynamic subscription for topic '{Topic}' with type '{MessageType}'",
            topic, messageType);

        // Use new RosSharp method for raw JObject delivery
        string id = _rosSocket!.SubscribeWithMessageTypeJObject(
            topic,
            messageType,
            (Newtonsoft.Json.Linq.JObject jObj) =>
            {
                _uiDispatcher.InvokeAsync(() =>
                {
                    try
                    {
                        callback(jObj);
                    }
                    catch (Exception ex)
                    {
                        _logger.LogError(ex, "Error processing dynamic JObject message callback");
                    }
                });
            }
        );

        if (!string.IsNullOrEmpty(id))
        {
            Interlocked.Increment(ref _activeSubscriptions);
            UpdateSubscriptionState();
            _logger.LogInformation("Dynamic subscription successful: Topic='{Topic}', Type='{MessageType}', ID='{SubId}'",
                topic, messageType, id);
        }
        else
        {
            _logger.LogError("Dynamic subscription failed - empty ID returned");
        }

        return Task.FromResult(id);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to create dynamic subscription for topic '{Topic}'", topic);
        return Task.FromResult(string.Empty);
    }
}

```
I will be uploading this version of the RosSharp library to NuGet in the future but for now feel free to use it locally.



## Original README: ##

[<img src="https://github.com/siemens/ros-sharp/wiki/img/Home_RosSharpLogo.png" width="480" alt ="ROS#"/>](https://github.com/siemens/ros-sharp)

## Overview ##

ROS# is a set of open-source software libraries and tools in [C\# ](https://docs.microsoft.com/de-de/dotnet/csharp/csharp) for communicating with [ROS](http://www.ros.org/) from .[NET](https://www.microsoft.com/net) applications, in particular [Unity](https://unity3d.com/). With ROS#, developers can effortlessly create .NET applications that communicate with ROS nodes, subscribe to and publish topics, handle actions and services, and interact with ROS messages. This enables the development of robotics applications, simulations, and automation systems within the .NET ecosystem.

<div style="text-align: center;">
  <img src="https://github.com/siemens/ros-sharp/wiki/img/Home_RosSharp_arch.png" alt="ROS# Architecture" style="padding-top: 5px; padding-bottom: 5px; width: 100%; height: 100%;">
</div>

[Here](https://github.com/siemens/ros-sharp/wiki/Info_Showcases) are some showcases illustrating what can be done with ROS#. The community provided various other application examples for ROS# [here](https://github.com/siemens/ros-sharp/issues/20). 

<div style="text-align: center;">
<img src="https://github.com/siemens/ros-sharp/wiki/img/Home_RosSharp_showcase.png" alt="ROS# Showcase" style="padding-top: 5px; padding-bottom: 5px; width: 100%; height: 100%;">
</div>

## Installation ##

ROS# can be used with Unity Engine __and/or__ with any compatible .NET project, see [platform support](https://github.com/siemens/ros-sharp/tree/master?tab=readme-ov-file#platform-support) and [external dependencies](https://github.com/siemens/ros-sharp/tree/master?tab=readme-ov-file#external-dependencies).

* ### For Unity Integration: Unity Package Manager ###
  1. In Unity Package Manager, click the "Add package from git URL" option at the top left corner.
  2. Paste the following URL: https://github.com/siemens/ros-sharp.git?path=/com.siemens.ros-sharp
  <img src="https://github.com/siemens/ros-sharp/wiki/img/User_Inst_InstallationROS_sharp_From_GitCombined.png" alt="ROS# Package Install from Git" style="padding-top: 10px; padding-bottom: 0px; width: 90%; height: 100%;">

> For more installation options, detailed instractions, and __getting started__ see wiki: [Installing and Configuring ROS# for Unity](https://github.com/siemens/ros-sharp/wiki/User_Inst_InstallationROS_sharp). 

* ### For .NET Projects: NuGet Gallery ###
  1. Head to the [NuGet](https://www.nuget.org/profiles/MartinBischoff) page.
  2. Install the required packages individually from NuGet.

> For more installation options, detailed instractions, and __getting started__ see wiki: [Installing and Configuring ROS# for .NET](https://github.com/siemens/ros-sharp/wiki/User_Inst_InstallationROS_sharp_DotNET). 


## Platform Support ##

* The [ROS#](https://github.com/siemens/ros-sharp/tree/master/Libraries/) dependencies require __.NET 8__ or __.NET Standard 2.1__ and __Visual Studio 2022__ or higher.
* The [Unity package](https://github.com/siemens/ros-sharp/tree/master/com.siemens.ros-sharp) has been developed with __Unity 2022.3.17f1 (LTS)__ and should also be compatible with older versions. __Unity 6__ support has not yet been thoroughly tested. However, some community members were able to use ROS# with Unity 6. 
> For Unity versions below 2022.3: See [wiki](https://github.com/siemens/ros-sharp/wiki/User_Inst_Unity3DOnWindows).

## Repository Structure ##

Below is an overview of the main directories and their purposes:

- **`com.siemens.ros-sharp/`**: Contains the custom Unity [UPM](https://docs.unity3d.com/Manual/CustomPackages.html) package for integrating ROS# into Unity projects. The package includes the whole ROS# .NET solution, as well as Unity specific scripts, external dependencies, and samples. This folder follows a planned layout as Unity [recommends](https://docs.unity3d.com/Manual/cus-layout.html).
  - **`Runtime/`**: Core ROS# .NET scripts and components for Unity integration.
  - **`Editor/`**: Unity specific editor scripts for mainly UI extensions.
  - **`Plugins/`**: External dependencies with specific versions.
  - **`Samples~/`**: Example samples that need to be imported through the package manager window. For more info about example scenes and reference code, see [wiki](https://github.com/siemens/ros-sharp/wiki).

- **`Libraries/`**: .NET solution containing the core ROS# libraries and tools for communicating with ROS.
  - **`RosBridgeClient/`**: Core ROS# .NET library for communicating with ROS via websockets.
  - **`MessageGeneration/`**: Tools for generating C# classes from ROS message definitions, including messages, actions, and services with both ROS1/2 support.
  - **`Urdf/`**: Library for parsing URDF files and creating Unity GameObjects.
  - **`PostBuildEvents/`**: OS specific post build scripts. Please see [post build events](https://github.com/siemens/ros-sharp/wiki/Dev_PostBuildEvents) for more information.

- **`ROS Packages/`**: ROS [packages](https://github.com/siemens/ros-sharp/tree/8330f5b433da22fff028dd1f355233b2f3e008e7/ROS%20Packages) that are used by ROS# for tutorial, testing and demonstration purposes.



## Further Info ##

* [Read the Wiki](https://github.com/siemens/ros-sharp/wiki)
* [External Dependencies](https://github.com/siemens/ros-sharp/wiki/Dev_ExternalDependencies)
* [Project Team, Contributors and Acknowledgements](https://github.com/siemens/ros-sharp/wiki/Info_Acknowledgements)
* [Contact the Project Team](mailto:ros-sharp.ct@siemens.com)

---
© Siemens AG, 2017-2025

