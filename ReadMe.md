
# Learning Docker
<!-- vscode-markdown-toc -->
* [Docker Concepts](#DockerConcepts)
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


## <a name='DockerConcepts'></a>Docker Concepts

An quick summary of important terms

Term | Definition
-|-
Docker | Docker is an open source container technology, based on industry standards.
Docker API  | Docker exposes an rest api to handle all interactions with the docker daemon/service.   
Docker image | A Docker image represents a collections of layers starting from an base os with additional modifications built on top.  
Docker container | A docker container is an instance of an image hosted by the docker runtime on a host OS.  
Docker compose | A way of running some dependent containers together on an single machine. Usually used in production. It's of describing how you want the containers to be launch in a certain environment.
Docker Swarm | A way of managing  Docker containers in a distributed fashion. Needs multiple Servers or at least virtual machines  
Kubernetes |  An container orchestrator that can work with Docker containers but not limited to that. ( Containerd standard)  
Docker Volume | Docker can mount external storage as volumes to containers to provide permanent storage ( database data file f.e.) You create a Volume seperately and refer to them in your containers. If a container asks for a volume which doesn't exist, Docker will create one behind the scenes.
Scratch Space | A container can not change it's image, but it want to write to existing files, it gets duplicated in a writeble layer, usually referred to as Scratch Space. The C: drive in a Windows container represents a virtual free size of 20GB. When we change a file in the container, this doesn't change the image. This layer is individual for each container and not shared! This layer is not the same as an attached volume, which points to an external storage device. When you change something in the volume, this is permanent.
Docker Network |Docker containers can talk to each other by their given docker name. Docker runtime acts as a kind of DNS.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|



## <a name='WindowsContainers'></a>Windows Containers

Docker can host both windows and linux containers at the same time.  
(You no longer need to switch between hosted eiter one of them)

It's possible you need specify the platform when requesting an image 
> docker image pull --platform linux  ubuntu


### <a name='Baseimages'></a>Base images

For windows containers you have several base images ([link](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-base-images)) from which you can start:

Name | Description | Approx. size | Docker Image
-|-|-|-
Nano Server | .net Core support  | 100Mb  
Server Core | also .Net Full framework support| size:2-3Gb |[link](https://hub.docker.com/_/microsoft-windows-servercore)  
Windows Server| also support for DirectX, PrintServices,...| 8-12Gb |[link](https://hub.docker.com/_/microsoft-windows)
> Windows Server should be avoided to be used, only for full VM replacements if Server Core doesn't support it's needs.

To interact with a windows container:
> docker run -it 114ef6544763  powershell

(Replace 114ef6544763 with the correct Id from your image)

> Attention: Windows Container versions must be compatible with Host OS version in some conditions [link](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-20H2%2Cwindows-10-20H2)

Windows 10 Host ->  couldn't take latest windows version -> tag 1803 worked for my  OS

> Attention: Microsoft has been moving their images to their own docker registry, so some images can exist with diffent namespaces

Use ```mcr.microsoft.com/windows/servercore``` instead of   ```microsoft/windows/servercore```

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
Here docker searches for DockerFile ( you could specify this file with -f parameter if elsewhere)

> Each line in DockerFile creates a new layer

#### <a name='Exampleforbuildingversionedimage'></a>Example for building versioned image

Build in the current folder
> docker build -t siduser/myApp:1.0.2 .

* a DockerFile is expect in the folder
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

> If you don't give a name for the container, Docker invents one like admiring_merkle: [link](https://anushibin.wordpress.com/2020/04/09/how-do-docker-containers-get-their-name/)


### <a name='Buildingsoftwareinsidecontainer'></a> Building software inside container

[docker sdk image](https://hub.docker.com/_/microsoft-dotnet-framework-sdk)  
[msbuild docker image example](https://github.com/NuGardt/docker-msbuild)
[.NET Microservices: Architecture for Containerized .NET Applications](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/)
``` Docker
From buildimage as build

#Separate package download from building sources
#avoid download same packages again and again! 
COPY packagefiles . #.csproj / package.config
CMD nuget restore / dotnet restore/npm install

COPY Sources  .
RUN build command to build app

From releaseimage
WORKDIR App
COPY  build:c:\\source  .
RUN App.exe
```

### <a name='Extractingcontentfromimage'></a>Extracting content from image

Example with TestingConsole makes an image with the console program copied from the builder to the final image.

Make the image:
>docker build -t examples/testingconsole:latest -f DockerFile.build .

We can retreive the build output from the image by creating a container ( not run it) and ask to copy the folder).
> docker create --name testing examples/testingconsole  
> docker cp testing:C:\\\\app\\\\ .\\\\app\\\\

We could run the container to let the program run too.
> docker container run -d examples/testingconsole TestingConsole.exe

But after our program has finished, the container stops again.  
How do we know the program did its job?
> docker container logs bea15qdf

*Hello Arcelor*


## <a name='Docker-Compose'></a>Docker-Compose

Managing your images and starting them together

[Reference](https://docs.docker.com/compose/compose-file/)
### <a name='Docker-Compose.ymlcontents'></a>Docker-Compose.yml contents


## <a name='DockerVolumes'></a>Docker Volumes

Volumes can be managed like images and containers.  
They exists separate from containers.

List docker volume
> docker volume ls

**Bind Mounts are not Volumes**

Volumes are permanent storage on the host that Docker is aware of.  
Bind mounts are direct connections to storage on the host.  
With Volumes a location is the docker container is made to represent the storage.  
With bind mount, you go directly to the host storage location.

If a Volume doesn't exist, Docker will create one for you.
If a bind mount location doesn't exist, Docker will throw an error.

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

For this demo we run straigth from the default image.
> docker run --name demodb -d -p 1433:1433 -e sa_password=azurePASSWORD123 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-express

When the container is running, you can connect to it with localhost\SQLEXPRESS

Attenttion: After the initial run, you don't have to add all the parameters.
> docker run demodb 

This will start the existing container again and the environment variables and port assignments remain in place.


### <a name='Runningaexternallyavailibleregistery'></a>Running a externally availible registery

* Setting up a  registry: [link](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry)
* Running an registery in Azure: [link](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro)
  * Build and push to Azure registry: [link](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/acr-template?view=azure-devops)

### <a name='Aditionallinks'></a>Aditional links

* [Migrate and mordernize with Kubernetes and Windows Container](https://www.youtube.com/watch?v=VJv-Jfs0fyk)
* [Docker MVP](https://blog.sixeyed.com/)
* [Example Docker images for windows](https://github.com/sixeyed/dockerfiles-windows)
* Docker is built of an industry standard for Containers --> [Containerd](https://containerd.io/).   
Because of this, containers built by docker are 
useable by other container technologies(Kubernetes, Podman etc)
* Labs:    git clone https://github.com/docker/labs
* Samples: git clone https://github.com/dockersamples/aspnet-monitoring
