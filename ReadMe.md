
# Learning Docker

##  1. <a name='LearningPlan'></a>Learning Plan

* Setup Rest Api ( IIS hosted)
* parse Dockerfile
* Docker basics

##  2. <a name='SetupRestApi'></a>Setup Rest Api

Create new project in VS2019: . NET Full Framework Web Application for Web API, I choose with docker and https support.
Enabling Docker Support here triggered the download of the required layers.
Image aspnet  --> 8.4Gb

Removed MVC unnecessary parts
Added basic .gitignore file ( search for Visual Studio gitignore)

Debugging with docker -> start container for my aplication ( tagged dev)

VS opens the ip adress of the container with used portnumber.
Append  api\values

Building container is done through Microsoft. VisualStudio. Azure. Containers. Tools. Targets Nuget package.
Takes care of creating, tagging, running, attaching, stopping, deleting containers based on latest code.

[Info on Docker in VS](https://docs.microsoft.com/en-us/visualstudio/containers/?view=vs-2019)

##  3. <a name='dockercommandlinestarters'></a>docker commandline starters

###  3.1. <a name='dockerbuild.'></a>docker build .

Builds an image from the given folder ( . means current folder) -> don't do this in c:\!!
This folder becomes the build context.
Here docker searches for dockerfile ( you can specify this file with -f parameter if elsewhere)

> Each line in DockerFile creates a new layer

####  3.1.1. <a name='dockerbuild-tsidusermyApp:1.0.2.'></a>docker build -t siduser/myApp:1.0.2 .

Add tags to specify the version you are building
You can add multiple tags to the same container  ( latest + version f.e.)
###  3.2. <a name='dockerrunimage:tag'></a>docker run  image:tag

##  4. <a name='DockerFile'></a>DockerFile

###  4.1. <a name='Basicstructure:'></a>Basic structure:

> \# Comment
> INSTRUCTION arguments

 FROM image
 RUN preparation step
 CMD executable


[Reference](https://docs.docker.com/engine/reference/builder/)  
[Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)


##  5. <a name='buildincontainer'></a>build in container

[Example](https://blog.alexellis.io/3-steps-to-msbuild-with-docker/) how to setup a docker to download and setup build tooling  
[Corresponding gist](https://gist.github.com/alexellis/1bceff8a360515f44c566e1a0ba8885f)

Example with Choco installed, make sure you add the least changing layers first ( re-use layers later builds)

Problem:
iwr : The request was aborted: Could not create SSL/TLS secure channel

Solution:
Try TLS1.2
[Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls'

