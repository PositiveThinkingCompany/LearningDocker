
# Learning Docker
<!-- vscode-markdown-toc -->
* 1. [ Learning Plan](#LearningPlan)
* 2. [Windows Containers](#WindowsContainers)
* 3. [ Setup Rest Api](#SetupRestApi)
* 4. [ docker commandline starters](#dockercommandlinestarters)
	* 4.1. [ docker build .](#dockerbuild.)
		* 4.1.1. [ docker build -t siduser/myApp:1.0.2 .](#dockerbuild-tsidusermyApp:1.0.2.)
	* 4.2. [ docker run  image:tag](#dockerrunimage:tag)
* 5. [ DockerFile](#DockerFile)
	* 5.1. [ Basic structure:](#Basicstructure:)
	* 5.2. [Multi stage structure](#Multistagestructure)
* 6. [Docker-Compose](#Docker-Compose)
	* 6.1. [Docker-Compose.yml contents](#Docker-Compose.ymlcontents)
* 7. [ Builds in container](#Buildsincontainer)
	* 7.1. [Using Choco](#UsingChoco)
	* 7.2. [installing VS2019 build tools](#installingVS2019buildtools)
	* 7.3. [Adding future build step of sources without having the sources already in place](#Addingfuturebuildstepofsourceswithouthavingthesourcesalreadyinplace)
* 8. [Usefull things](#Usefullthings)
	* 8.1. [Store all images and container to other drive](#Storeallimagesandcontainertootherdrive)
	* 8.2. [Running a externally availible registery](#Runningaexternallyavailibleregistery)
	* 8.3. [External links:](#Externallinks:)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->
##  1. <a name='LearningPlan'></a> Learning Plan

* Setup Rest Api ( IIS hosted)
* parse Dockerfile
* Docker basics
* Labs:    git clone https://github.com/docker/labs
* Samples: git clone https://github.com/dockersamples/aspnet-monitoring
* Windows Containers

##  2. <a name='WindowsContainers'></a>Windows Containers
Docker can host both windows and linux containers at the same time.

### Base images

For windows containers you have several base images from which you can start:

Nano Server ( .net Core support) --> 100Mb
Server Core ( also .Net Full framework support) --> 1.5Gb [docker image](https://hub.docker.com/_/microsoft-windows-servercore)
Windows Server( also support for DirectX, PrintServices,...) --> 3.57Gb [docker image](https://hub.docker.com/_/microsoft-windows)
> Windows Server should be avoided to be used, only for full VM replacements if needed.

To interact with a windows container:
docker run -it 114ef6544763  powershell 
(Replace 114ef6544763 with the correct Id from your image)

> Attention: Windows Container versions must be compatible with Host OS version in some usage cases [link](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-20H2%2Cwindows-10-20H2)

### Host webservices

If we want a way to host an .NET Full framework web service, we can use aspnet [docker image](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet)
This image contains:

* Windows Server Core as the base OS
* IIS 10 as Web Server
* .NET Framework (multiple versions available)
* .NET Extensibility for IIS

> For example to get a image with .NET 4.8 pre-installed
docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8



##  3. <a name='SetupRestApi'></a> Setup Rest Api

Create new project in VS2019: . NET Full Framework Web Application for Web API, I choose with docker and https support.
Enabling Docker Support here triggered the download of the required layers.
Image aspnet  --> 8.4Gb

Removed MVC unnecessary parts
Added basic .gitignore file ( search for Visual Studio gitignore)

Debugging with docker -> start container for my aplication ( tagged dev)

VS opens the ip adress of the container with used portnumber.
Append  api\values
also exposed under localhost ( check Docker Desktop to find port)

Building container is done through Microsoft. VisualStudio. Azure. Containers. Tools. Targets Nuget package.
Takes care of creating, tagging, running, attaching, stopping, deleting containers based on latest code.

[Info on Docker in VS](https://docs.microsoft.com/en-us/visualstudio/containers/?view=vs-2019)

##  4. <a name='dockercommandlinestarters'></a> docker commandline starters

###  4.1. <a name='dockerbuild.'></a> docker build .

Builds an image from the given folder ( . means current folder) -> don't do this in c:\!!
This folder becomes the build context.
Here docker searches for dockerfile ( you can specify this file with -f parameter if elsewhere)

> Each line in DockerFile creates a new layer

> If you don't give a name, Docker composes one like admiring_merkle: [link](https://anushibin.wordpress.com/2020/04/09/how-do-docker-containers-get-their-name/)


####  4.1.1. <a name='dockerbuild-tsidusermyApp:1.0.2.'></a> docker build -t siduser/myApp:1.0.2 .

Add tags to specify the version you are building
You can add multiple tags to the same container  ( latest + version f.e.)
###  4.2. <a name='dockerrunimage:tag'></a> docker run  image:tag

To run powershell in that container interactively:   docker run -it  image:tag powershell


##  5. <a name='DockerFile'></a> DockerFile

###  5.1. <a name='Basicstructure:'></a> Basic structure:

``` Docker
# Comment like this

#each line represents a build step -> move stable layers to top.
INSTRUCTION arguments

 FROM image
 SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"] # Optional to use  powershell commands. This catches errors in scripts to continue silently with other steps.
 ARG  key1=value1 (environment variable during build)
 ENV  key2=value2 ( environment variable during build and in running container)
 RUN preparation step (runs during build -> layer)
 EXPOSE 80  ( exposes interal port 80 to outside, if external port is left out, docker chooses port usually orgistrator -> so don't specify external port here for production, use docker-compose)
 VOLUME ["/data"]
 CMD executable ( runs at start of container)
```

[Reference](https://docs.docker.com/engine/reference/builder/)  
[Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
###  5.2. <a name='Multistagestructure'></a>Multi stage structure
``` Docker
From image as base
RUN preparation

From base
RUN further preparation
```

##  6. <a name='Docker-Compose'></a>Docker-Compose

Managing your images and starting them together

[Reference](https://docs.docker.com/compose/compose-file/)
###  6.1. <a name='Docker-Compose.ymlcontents'></a>Docker-Compose.yml contents


##  7. <a name='Buildsincontainer'></a> Builds in container

We need a base image and install tooling, chocolatey is popular

###  7.1. <a name='UsingChoco'></a>Using Choco

Start with Choco installed, make sure you add the least changing layers first ( re-use layers later builds)

Azure Devops build agent installers: [powershell scripts](https://github.com/akuryan/vsts-image-generation/tree/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers)
F.e: install docker on container: [Install-Docker.ps1](https://github.com/akuryan/vsts-image-generation/blob/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers/Install-Docker.ps1)

Error:
iwr : The request was aborted: Could not create SSL/TLS secure channel

Solution:
Try TLS1.2
[Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls'

###  7.2. <a name='installingVS2019buildtools'></a>installing VS2019 build tools
Build container
docker build -t examples/serverwithchoco:latest .
Connect to container:
docker run -it examples/serverwithchoco:latest powershell

###  7.3. <a name='Addingfuturebuildstepofsourceswithouthavingthesourcesalreadyinplace'></a>Adding future build step of sources without having the sources already in place

[ONBUILD](https://docs.docker.com/engine/reference/builder/#onbuild)
Perform this build step in later builds for applications:

```Docker
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```
Usefull for standardized builder.

Alternative example:
[Example](https://blog.alexellis.io/3-steps-to-msbuild-with-docker/) how to setup a docker to download and setup build tooling  
[Corresponding gist](https://gist.github.com/alexellis/1bceff8a360515f44c566e1a0ba8885f)

Alternative way of building containers: 
> BuildKit is a toolkit for converting source code to build artifacts in an efficient, expressive and repeatable manner
[github](https://github.com/moby/buildkit)  
[blogpost](https://blog.mobyproject.org/introducing-buildkit-17e056cc5317)


##  8. <a name='Usefullthings'></a>Usefull things

###  8.1. <a name='Storeallimagesandcontainertootherdrive'></a>Store all images and container to other drive

Another option would be to create/modify the C:\ProgramData\Docker\config\daemon.json file as referenced in the getting started guide [here](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/docker/configure_docker_daemon)
```json
{
    "graph": "D:\\ProgramData\\Docker"
}
```

###  8.2. <a name='Runningaexternallyavailibleregistery'></a>Running a externally availible registery

[link](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry)

###  8.3. <a name='Externallinks:'></a>External links:

[Migrate and mordernize with Kubernetes and Windows Container](https://www.youtube.com/watch?v=VJv-Jfs0fyk)
