---
author: micdemarco
comments: true
date: 2016-11-12 02:50:15+00:00
layout: post
slug: setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-1
title: Setting up aspnet core onion architecture solution with CD using docker hub + free aws EC2 container - Part 1 
tags:
- aspnet core
- github
- docker
- aws
- container
- onion architecture
- free
---

In this blog post I am going to run through the following:

- [Part 1](/2016/11/12/setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-1/) - Creating a new aspnet core solution in visual studio with multiple projects, following the [onion architecture](https://rules.ssw.com.au/do-you-know-the-layers-of-the-onion-architecture)  and running it in a docker container.
- [Part 2](/2016/11/19/setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-2/) - Setting up continuous deployment of the solution using docker hub running on a free aws EC2 container

# Part 1 - Getting started with visual studio and docker

# Step 1 - Create a solution in VS and scaffold a basic onion architecture.

I am using Visual Studio 2015 to create a new aspnet core web api in a solution, and then adding a business and a repository layer.

This is what the solution looks like:

![Solution structure](/assets/2016-11-13_15-42-11.png) 

The code is available [on github](https://github.com/micdemarco/Mic.Fraz)

In the solution I have created the basic scaffolding for an onion architecture.  It is for demonstration purposes, without implementing any repository or business logic.

I have added a User controller with a single Get method that returns a new User from a dummy repository through the UserManager.    

```csharp
    [Route("api/[controller]")]
    public class UserController : Controller
    {
        private readonly IUserManager _userManager;

        public UserController(IUserManager userManager)
        {
            _userManager = userManager;
        }

        // GET api/user/5
        [HttpGet("{id}")]
        public async Task<IActionResult> Get(int id)
        {
            var result = await _userManager.GetAsync(id);
            return Ok(result);
        }

    }
    
```

```csharp
  public class UserRepository: IUserRepository
    {
        public UserRepository()
        {
        }

        public async Task<User> GetAsync(int id)
        {          
            // normally an async database operation goes here  
            return new User()
            {
                Id = id,
                Name = $"Name-{id}-{Guid.NewGuid()}"
            };
        }
    }
```

When I run the solution I get this in my browser:

![Run in chrome](/assets/2016-11-13_15-42-12.png) 

# Step 2 - Install docker and run microsoft/dotnet:latest image

There are many tutorials on getting started with docker so I won't go into that.  The [docker getting started docs](https://docs.docker.com/engine/getstarted/) are excellent! 

In the next step I will be using the [microsoft/dotnet image](https://hub.docker.com/r/microsoft/dotnet/) which contains the sdk for building dotnet core applications.

# Step 3 - Create and run a docker image

Next create a docker file that will copy the solution files, compile it and run it.

In the solution root, create a Dockerfile and add the following:

```Dockerfile
FROM microsoft/dotnet:latest
COPY . /app
WORKDIR /app
 
RUN ["dotnet", "restore"]
RUN ["dotnet", "publish", "./src/Mic.Fraz.Api/"]

EXPOSE 5000/tcp
ENV ASPNETCORE_URLS http://*:5000

ENTRYPOINT ["dotnet","./src/Mic.Fraz.Api/bin/Debug/netcoreapp1.0/publish/Mic.Fraz.Api.dll"]
```

What this file does: 

1. Starts off with the microsoft/dotnet:latest image
2. Copies the whole solution to a folder /app inside the image
3. Changes to working directory /app
4. Runs restore for all projects 
5. Publishes the Api app
6. Exposes port 5000 and defines port 5000 as the port for ASPNETCORE_URLS
7. Defines the entry point command for the image as the published dll

*Note:*

*There are many different ways the compilation can be done resulting in smaller and more optimised images. This method can take a raw repository and compile and run it without any additional dependencies.*  

*You could alternatively build and publish the app outside of the image and just copy the published files, meaning you would not need to perform any of the build, restore and publish commands inside the image.  This will result in a quicker build time and smaller image.*

With the docker file in place, the docker image can be built.

In the root directory of the solution, run the command:

```
C:\code\Mic.Fraz>docker build -t mic.fraz:latest .
```

The output will look like:

```
C:\code\Mic.Fraz>docker build -t mic.fraz:latest .
Sending build context to Docker daemon 3.669 MB
Step 1 : FROM microsoft/dotnet:latest
 ---> 486c56e26c1a
Step 2 : COPY . /app
 ---> 171d0a0e724c
Removing intermediate container b7aa4403b96d
Step 3 : WORKDIR /app
 ---> Running in e54483eb5c2a
 ---> b8f8f755dff8
Removing intermediate container e54483eb5c2a
Step 4 : RUN dotnet restore
 ---> Running in dd6e1daae2c3
log  : Restoring packages for /app/src/Mic.Fraz.BusinessInterfaces/project.json...
log  : Restoring packages for /app/src/Mic.Fraz.Repository/project.json...
log  : Writing lock file to disk. Path: /app/src/Mic.Fraz.Repository/project.lock.json
log  : Writing lock file to disk. Path: /app/src/Mic.Fraz.BusinessInterfaces/project.lock.json
log  : /app/src/Mic.Fraz.Repository/project.json

...

...

Published 1/1 projects successfully
 ---> 5c851362e478
Removing intermediate container 0d45ece1c0b1
Step 6 : EXPOSE 5000/tcp
 ---> Running in 4eb7066b2109
 ---> e928a4f017d5
Removing intermediate container 4eb7066b2109
Step 7 : ENV ASPNETCORE_URLS http://*:5000
 ---> Running in 27904da28b2b
 ---> 9c3971c29400
Removing intermediate container 27904da28b2b
Step 8 : ENTRYPOINT dotnet ./src/Mic.Fraz.Api/bin/Debug/netcoreapp1.0/publish/Mic.Fraz.Api.dll
 ---> Running in da3970c66b69
 ---> 2878eaaf2b6a
Removing intermediate container da3970c66b69
Successfully built 2878eaaf2b6a
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.

```

Running `docker images` command will show the newly built image:

```
C:\code\Mic.Fraz>docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
mic.fraz               latest              2878eaaf2b6a        18 hours ago        580.9 MB
microsoft/aspnetcore   latest              859cfeaa74cd        4 days ago          266.8 MB
microsoft/dotnet       latest              486c56e26c1a        4 days ago          537.5 MB
```

*Note the size.  590.9 MB ... yes its huge.  The dotnet image is big to start.  The aspnetcore image that can be used for running a compiled app is around half the size.* 

The image can now be started in a container using the `docker run` command:

```
C:\code\Mic.Fraz>docker run -d -p 8080:5000 mic.fraz:latest
24f0f63dbcd5ba01d7e2271e73d2ee709ea09ee230b9e12b6c910935dcc68f05
```

The above command starts the image in detached mode (-d) and binds local port 8080 to the container's port 5000, 

Accessing the api on localhost port 8080 will show results from the api running in the container: 

![Run in chrome](/assets/2016-11-13_15-42-13.png) 

*Update Part 2 published*
# Part 2 - Click [here](2016/11/12/setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-2/)

In [Part 2](/2016/11/12/setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-2/), I describe the steps and gotchas of setting up continuous deployment using docker hub free aws EC2 container hosting.    

