
# Learning Docker

##  Learning Plan

* Setup Rest Api ( IIS hosted)
* parse Dockerfile
* Docker basics
* Labs:    git clone https://github.com/docker/labs
* Samples: git clone https://github.com/dockersamples/aspnet-monitoring


##  Setup Rest Api

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

##  docker commandline starters

###  docker build .

Builds an image from the given folder ( . means current folder) -> don't do this in c:\!!
This folder becomes the build context.
Here docker searches for dockerfile ( you can specify this file with -f parameter if elsewhere)

> Each line in DockerFile creates a new layer

> If you don't give a name, Docker composes one like admiring_merkle: [link](https://anushibin.wordpress.com/2020/04/09/how-do-docker-containers-get-their-name/)


####  docker build -t siduser/myApp:1.0.2 .

Add tags to specify the version you are building
You can add multiple tags to the same container  ( latest + version f.e.)
###  docker run  image:tag

To run powershell in that container interactively:   docker run -it  image:tag powershell


##  DockerFile

###  Basic structure:

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
### Multi stage structure
``` Docker
From image as base
RUN preparation

From base
RUN further preparation
```

## Docker-Compose

Managing your images and starting them together

[Reference](https://docs.docker.com/compose/compose-file/)
### Docker-Compose.yml contents


##  Builds in container

We need a base image and install tooling, chocolatey is popular

### Using Choco

Start with Choco installed, make sure you add the least changing layers first ( re-use layers later builds)

Azure Devops build agent installers: [powershell scripts](https://github.com/akuryan/vsts-image-generation/tree/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers)
F.e: install docker on container: [Install-Docker.ps1](https://github.com/akuryan/vsts-image-generation/blob/07a3c1f547a9a3304f965ed44d437f0d8d8d7589/images/win/scripts/Installers/Install-Docker.ps1)

Error:
iwr : The request was aborted: Could not create SSL/TLS secure channel

Solution:
Try TLS1.2
[Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls'

### installing VS2019 build tools
Build container
docker build -t examples/serverwithchoco:latest .
Connect to container:
docker run -it examples/serverwithchoco:latest powershell

### Adding future build step of sources without having the sources already in place

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


## Usefull things

### Store all images and container to other drive

Another option would be to create/modify the C:\ProgramData\Docker\config\daemon.json file as referenced in the getting started guide [here](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/docker/configure_docker_daemon)
```json
{
    "graph": "D:\\ProgramData\\Docker"
}
```

### Running a externally availible registery

[link](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry)