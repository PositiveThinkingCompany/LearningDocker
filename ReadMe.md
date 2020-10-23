
# Learning Docker
<!-- vscode-markdown-toc -->
* [ Learning Plan](#LearningPlan)
* [Windows Containers](#WindowsContainers)
	* [Base images](#Baseimages)
	* [Host webservices](#Hostwebservices)
* [ DockerFile](#DockerFile)
	* [ Basic structure](#Basicstructure)
* [ Docker commandline starters](#Dockercommandlinestarters)
	* [ docker build .](#dockerbuild.)
		* [Example for building versioned image](#Exampleforbuildingversionedimage)
	* [ docker run  image:tag](#dockerrunimage:tag)
	* [ Building software inside container](#Buildingsoftwareinsidecontainer)
	* [Extracting content from image](#Extractingcontentfromimage)
* [Docker-Compose](#Docker-Compose)
	* [Docker-Compose.yml contents](#Docker-Compose.ymlcontents)
* [Docker Volumes](#DockerVolumes)
* [ Setup Rest Api](#SetupRestApi)
* [ Builds in container](#Buildsincontainer)
	* [Using Choco](#UsingChoco)
	* [installing VS2019 build tools](#installingVS2019buildtools)
	* [Adding future build step of sources without having the sources already in place](#Addingfuturebuildstepofsourceswithouthavingthesourcesalreadyinplace)
* [Usefull things](#Usefullthings)
	* [Store all images and container to other drive](#Storeallimagesandcontainertootherdrive)
	* [Running  a database in a container](#Runningadatabaseinacontainer)
	* [Running a externally availible registery](#Runningaexternallyavailibleregistery)
	* [Aditional links](#Aditionallinks)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->
## <a name='LearningPlan'></a> Learning Plan

* Windows Containers
* Docker installation
* parse Dockerfile
* Docker basics
* Labs:    git clone https://github.com/docker/labs
* Samples: git clone https://github.com/dockersamples/aspnet-monitoring
* Setup Rest Api ( IIS hosted)
* Build inside containers
* Docker Compose
* Docker volumes

## <a name='WindowsContainers'></a>Windows Containers
Docker can host both windows and linux containers at the same time.

### <a name='Baseimages'></a>Base images

For windows containers you have several base images ([link](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-base-images)) from which you can start:

Nano Server ( .net Core support) --> 100Mb  
Server Core ( also .Net Full framework support), approx size:2-3Gb [docker image](https://hub.docker.com/_/microsoft-windows-servercore)  
Windows Server( also support for DirectX, PrintServices,...) approx size 8-12Gb [docker image](https://hub.docker.com/_/microsoft-windows)
> Windows Server should be avoided to be used, only for full VM replacements if needed.

To interact with a windows container:
> docker run -it 114ef6544763  powershell

(Replace 114ef6544763 with the correct Id from your image)

> Attention: Windows Container versions must be compatible with Host OS version in some conditions [link](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-20H2%2Cwindows-10-20H2)

>Windows 10 Host ->  can't take latest windows versions -> tag 1803 worked

### <a name='Hostwebservices'></a>Host webservices

If we want a way to host an .NET Full framework web service, we can use an aspnet image [docker image](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet)

This image contains:

* Windows Server Core as the base OS
* IIS 10 as Web Server
* .NET Framework (multiple versions available)
* .NET Extensibility for IIS

For example to get a image with .NET 4.8 pre-installed
> docker pull mcr.microsoft.com/dotnet/framework/aspnet:4.8

## <a name='DockerFile'></a> DockerFile

### <a name='Basicstructure'></a> Basic structure

``` Docker
# Comment like this

#each line represents a build step -> move stable layers to top.
INSTRUCTION arguments

 FROM image
 SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"] # Optional to use  powershell commands. This catches errors in scripts to continue silently with other steps.
 ARG  key1=value1 (environment variable during build)
 ENV  key2=value2 ( environment variable during build and in running container)
 RUN preparation step (runs during build -> layer)
 EXPOSE 80  ( exposes interal port 80 to outside, if external port is left out, docker chooses port usually orgistrator -> so don''t specify external port here for production, use docker-compose)
 ONBUILD instruction #this instruction will be done in derived image builds (re-use this image)
 VOLUME ["/data"] #if the volume doesn't exist, docker will create the volume on the local disk
 CMD executable ( runs at start of container)
```

[Reference](https://docs.docker.com/engine/reference/builder/)  
[Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## <a name='Dockercommandlinestarters'></a> Docker commandline starters

### <a name='dockerbuild.'></a> docker build .

Builds an image from the given folder ( . means current folder) -> don't do this in c:\!!
This folder becomes the build context.
Here docker searches for dockerfile ( you can specify this file with -f parameter if elsewhere)

> Each line in DockerFile creates a new layer

> If you don't give a name, Docker composes one like admiring_merkle: [link](https://anushibin.wordpress.com/2020/04/09/how-do-docker-containers-get-their-name/)


#### <a name='Exampleforbuildingversionedimage'></a>Example for building versioned image
> docker build -t siduser/myApp:1.0.2 .

* Add tags to specify the version you are building
* You can add multiple tags to the same container  ( latest + version f.e.
### <a name='dockerrunimage:tag'></a> docker run  image:tag

To run powershell in that container interactively:   
> docker run -it  -rm image:tag powershell

This will launch the container and return a shell for you to use, -rm tell docker to remove the container when you exit the shell.

Some containers can be started interactively ( aspnet f.e.)
Start them detached
> docker run -d image:tag

Pass an environment variable
> docker run --env MYARG=value1 image:tag

If you want to connect to a running windows container
> docker exec -it 1267DE4F6B09 powershell

If you want to see output from container
> docker container logs  1267DE4F6B09

If you want have more details
> docker container inspect 1267DE4F6B09

This is Json so you can capture it in powershell shell for example
> docker container inspect 1267DE4F6B09 | convertfrom-json

If you want to know resource usage of the container
> docker container stats 1267DE4F6B09

If you want to remove a container ( even running)
> docker container rm -f 1267DE4F6B09

### <a name='Buildingsoftwareinsidecontainer'></a> Building software inside container

[docker sdk image](https://hub.docker.com/_/microsoft-dotnet-framework-sdk)  
[msbuild docker image example](https://github.com/NuGardt/docker-msbuild)
``` Docker
From buildimage as build
COPY Sources  .
RUN build command to build app

From releaseimage
WORKDIR App
COPY  build:c:\\source  .
RUN App.exe
```

### <a name='Extractingcontentfromimage'></a>Extracting content from image
We can retreive the build output from the image by creating a container ( not run) and ask to copy the folder).
> docker create --name testingconsole-1 examples/testingconsole  
> docker cp testingconsole-1:C:\\\\app\\\\ .\\\\app\\\\

## <a name='Docker-Compose'></a>Docker-Compose

Managing your images and starting them together

[Reference](https://docs.docker.com/compose/compose-file/)
### <a name='Docker-Compose.ymlcontents'></a>Docker-Compose.yml contents


## <a name='DockerVolumes'></a>Docker Volumes

Volumes can be managed like images and containers.  
They exists separate from containers.

List docker volume
> docker volume ls

[Reference](https://docs.docker.com/storage/volumes/)

## <a name='SetupRestApi'></a> Setup Rest Api

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

## <a name='Buildsincontainer'></a> Builds in container

We need a base image and install tooling, chocolatey is popular

### <a name='UsingChoco'></a>Using Choco

Start with Choco installed, make sure you add the least changing layers first ( re-use layers later builds)

Azure Devops build agent installers: [powershell scripts](https://github.com/akuryan/vsts-image-generation/tree/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers)
F.e: install docker on container: [Install-Docker.ps1](https://github.com/akuryan/vsts-image-generation/blob/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers/Install-Docker.ps1)

Error:
iwr : The request was aborted: Could not create SSL/TLS secure channel

Solution:
Try TLS1.2
[Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls'

### <a name='installingVS2019buildtools'></a>installing VS2019 build tools

Build container  
docker build -t examples/serverwithchoco:latest .  
Connect to container:  
docker run -it examples/serverwithchoco:latest powershell

### <a name='Addingfuturebuildstepofsourceswithouthavingthesourcesalreadyinplace'></a>Adding future build step of sources without having the sources already in place

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


## <a name='Usefullthings'></a>Usefull things

### <a name='Storeallimagesandcontainertootherdrive'></a>Store all images and container to other drive

Another option would be to create/modify the C:\ProgramData\Docker\config\daemon.json file as referenced in the getting started guide [here](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/docker/configure_docker_daemon)
```json
{
    "data-root": "D:\\ProgramData\\Docker"
}
```
### <a name='Runningadatabaseinacontainer'></a>Running  a database in a container

> docker run --name demodb -d -p 1433:1433 -e sa_password=azurePASSWORD123 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-express

When the container is running, you can connect to it with localhost\SQLEXPRESS

### <a name='Runningaexternallyavailibleregistery'></a>Running a externally availible registery

[link](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry)

### <a name='Aditionallinks'></a>Aditional links

* [Migrate and mordernize with Kubernetes and Windows Container](https://www.youtube.com/watch?v=VJv-Jfs0fyk)
* [Docker MVP](https://blog.sixeyed.com/)
* [Example Docker images for windows](https://github.com/sixeyed/dockerfiles-windows)
* Docker is built of an industry standard for Containers --> [Containerd](https://containerd.io/).   
Because of this, containers built by docker are 
useable by other container technologies(Kubernetes, Podman etc)