#Linking Unity with ROS on windows: Why?
If you work on robotic, you sometimes need to test the behavior of your robot. But testing "in real life" can be harmful for the test subject (you or other people) and/or for the robot, and it can also take a lot of time to set up the testing environment. This is why during an internship, we decided to use Unity to simulate a VR environment for our robot. 

I used Windows 7 Service Pack 1 for my dev setup, and my algorithm was written in C++. Other setups haven't been tested but it could work using some tweaks.
I recommend you to use a PC with a good amount of RAM (>=8Go), a quad-core CPU, and a decent GPU, because you will run 4 simultaneous programs talking to each other and one of them is Unity.

# Installation

## Prerequisites

### Build tools

#### Visual Studio
You need VS to build your algorithm and to edit Unity C# scripts.
I used VS 2010 x86 and .NET Framework 4.
Downloads: [VS 2010 SP1 x86][vs-2010-sp1-dl] ; [.NET Framework 4][net-4-framework-dl]

#### CMake
Cmake is used to make the VS solution from raw c++ source folder.
I used CMake 3.5.2.
Download: [CMake 3.5.2][cmake-3.5.2-dl]

### Libraries

## Main program

### Your algorithm
Difference with your original algorithm:
inputs are coming from the VR environment through ROS, so you have to replace your inputs with topics.

To build from the original:
- create a new VS project (skip this if you already have a VS project)
- import the algorithm files (skip this if you already have a VS project)
- right click on the project -> Properties -> Configuration properties -> ADD the following to the included properties: 
    + VC++ Directories -> Include Directories:
        * \path\to\rosdeps\hydro\x86\include;
        * \path\to\ros\hydro\x86\include;
    +  VC++ Directories -> Library Directories:
        *  \path\to\ros\hydro\x86\lib;
        *  \path\to\rosdeps\hydro\x86\lib;
    +  Linker -> Input:
        *  roscpp.lib;
        *  roscpp_serialization.lib;
        *  rosconsole.lib;
        *  rostime.lib;

### Win-ROS
To communicate between your algorithm and Unity we use ROS.
Win-ROS is a Windows implementation of ROS that contains all the necessary communication libraries and the ROS server.
Download: [Win-ROS][winros-dl]
Install tutorial: [winros dependencies and tutorial][winros-install] , follow the order of installation.
Usage: [How to use winros in a project][winros-usage]

Don't forget to add these to the Path of your computer: 
\path\to\rosdeps\hydro\x86\lib;
\path\to\rosdeps\hydro\x86\bin;
\path\to\ros\hydro\x86;
\path\to\ros\hydro\x86\bin;

Also add these to your environment (from [env documentation][winros-env-doc], [ROS.NET readme][ros.net]) :
- ROS_HOSTNAME = localhost
- ROS_IP = 127.0.0.1
- ROS_MASTER_URI = http://localhost:11311 

### Unity
We use Unity as an environment simulator with 2 assets (see below).
Download: [Unity][unity-dl]

### ROS.NET_Unity
ROS.Net is a .Net (C#) implementation of ROS communication (allows communication with ROS server).
ROS.Net_Unity is a Unity asset based on ROS.Net.

To install:
- git clone https://github.com/uml-robotics/ROS.NET
- git clone --recursive https://github.com/uml-robotics/ROS.NET_Unity
- copy all the folders and files from the ROS.NET folder to the ".ros.net" folder of ROS.NET_Unity
- run "buildUnityScriptSource.bat" from the ROS.NET_Unity folder
- a folder "COPY_TO_UNITY_PROJECT" will be created
- inside your Unity project, create a folder "Plugins" and it's sub-folder "x86"
- copy the content of "COPY_TO_UNITY_PROJECT" inside Plugins/x86

To use it:
- inside a C# script of your Unity project, add "using Ros_CSharp;" among the packages you use. You can also add "using Messages;" to work with the different types of ROS messages.
- Don't forget to add ROS.Init inside the Start method of your project and ROS.Shutdown in your OnApplicationQuit method

### MiddleVR (optional)
MiddleVR is a middleware that manage VR input/outputs using configurations. It is useful when you use VR, it adapts the camera view to your screen (PC screen, HTC Vive, Oculus, etc.).

Download: [MiddleVR for Unity - Presentation][middlevr-dl]
Video Tutorials: [tutorial 1][video-tuto-1] ; [tutorial 2][video-tuto-2] ; [tutorial 3][video-tuto-3]
Official Tutorials: [Official tutorials][official-tutorials]

# Run

## Runtime order
- roscore
- Unity -> play (wait for the play button and the editor window to change color)
- Your algorithm

## Ros server
Commands:
- Open a Visual Studio command prompt
- run \path\to\ros\hydro\x86\setup.bat (or \path\to\ros\hydro\x86\ROSsetup.bat)
- run \path\to\ros\hydro\x86\bin\roscore
- leave this command prompt open
More info: the ros server is the core to communicate, it could be possible to create a roslaunch that run the server but I followed the simple method described in the Winros package doc.

# Runtime tips
When you run the assistance algorithm, ROS.NET Unity prints in the unity console when the range subscribers connects, so you know it's communicating. (~= working)
You can open a new VS command prompt, run the setup.bat, and use the debbuging tools of rostopic (info, list, etc.).

# Ideas to develop

## Docker
You could use a ROS docker image in which you would run the assistance algorithm and roscore, I already tested to communicate with a rosmaster in a docker container, it works. It will limit development/adapting on Windows, which takes time and not in the philosophy of "not modifying too much the initial code".
It needs time to test the performance, but if 10 000 docker containers can manage a big database on a modest server, I'm sure with 2 containers (1 for roscore + 1 for the assistance algorithm), you will do fine. Don't do it if you use devices (USB or other), and at this moment docker doesn't manage Windows device interface (which is different from Unix device files).

[net-4-framework-dl]: https://www.microsoft.com/fr-fr/download/details.aspx?id=17851
[vs-2010-sp1-dl]: https://www.microsoft.com/fr-fr/download/details.aspx?id=23691
[cmake-3.5.2-dl]: https://cmake.org/download/
[winros-dl]: http://wiki.ros.org/win_ros
[winros-install]: http://wiki.ros.org/win_ros/hydro/Msvc%20SDK
[winros-usage]: http://wiki.ros.org/win_ros/hydro/Msvc%20SDK%20Projects
[winros-env-doc]: http://wiki.ros.org/ROS/EnvironmentVariables
[unity-dl]: https://store.unity.com/
[ros.net]: https://github.com/uml-robotics/ROS.NET
[middlevr-dl]: http://www.middlevr.com/middlevr-for-unity/
[video-tuto-1]: https://www.youtube.com/watch?v=XEBN8teItFk
[video-tuto-2]: https://www.youtube.com/watch?v=Ym_KVxTZahY
[video-tuto-3]: https://www.youtube.com/watch?v=SBtiggeljXk
[official-tutorials]: http://www.middlevr.com/tutorials/
[unity_project]: ./unity_project.png
