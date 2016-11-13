---
author: micdemarco
comments: true
date: 2016-11-12 02:50:15+00:00
layout: post
slug: setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-1
title: Setting up aspnet core onion architecture solution with continuous deployment using docker hub and free aws EC2 container - Part 1 
tags:
- aspnet core
- bitbucket
- docker
- aws
- container
- onion
- repository layer
---
In this blog post I am going to run through the steps of setting up a new aspnet core solution with multiple projects and deploying it to docker hub running aws EC2 containers, all for free.

# Step 1 - Create a solution in VS and scaffold an basic onion architecture.

I am using Visual Studio 2015 to create a new aspnet core web api in a solution, and then added a business and repository layer.

This is what the solution looks like:

![Solution structure](/assets/2016-11-13_15-42-11.png) 

The api simply returns a new User from the repository.

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

The code is available [on github](https://github.com/micdemarco/Mic.Fraz)

# Step 2 - Install docker and run microsoft/dotnet:latest image

There are many tutorials on getting started with docker so I won't go into that.  The [docker getting started docs](https://docs.docker.com/engine/getstarted/) are excellent! 

For the next step we will be using the [microsoft/dotnet image](https://hub.docker.com/r/microsoft/dotnet/) which contains the sdk for building dotnet core applications.

# Step 3 - Create and run a docker image

Next we will create a docker file that will take out solution, compile it and run it.

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

What this file does is: 

1. Starts off with the microsoft/dotnet:latest image
2. Copies the whole solution to a filder /app inside the image
3. Changes to working directory /app
4. Runs restore for all projects 
5. Publishes the Api app
6. Exposes port 5000 and defines port 5000 as the port for ASPNETCORE_URLS
7. Defines the entry point command for the image as the published dll

*Note/Disclaimer:*

*There are many different ways the compilation can be done resulting in smaller and more optimised images. This method can take a raw repository and compile and run it without any additional dependencies.*  

*You could alternatively build and publish the app outside of the image and just copy the published files, meaning you would not need to perform any of the build, restore and publish commands inside the image.  This will result in a quicker build time and smaller image.*

Next we are ready to build the image.

In the root directory of the solution, run the command:

```
C:\work\personal\Mic.Fraz>docker build -t mic.fraz:latest .
```

The output will look like:

```
C:\work\personal\Mic.Fraz>docker build -t mic.fraz:latest .
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

Now if we run docker images we will see the newly built image

```
C:\work\personal\Mic.Fraz>docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
mic.fraz               latest              2878eaaf2b6a        18 hours ago        580.9 MB
```

Next we can test run the image in a container to see if it works

```
C:\work\personal\Mic.Fraz>docker run -d -p 8080:5000 mic.fraz:latest
24f0f63dbcd5ba01d7e2271e73d2ee709ea09ee230b9e12b6c910935dcc68f05
```

The above command starts the image that we just created in detached mode and bunds local port 8080 to the container's port 5000 

Accessing the api on localhost port 8080 will show results from the api running in the container: 

![Run in chrome](/assets/2016-11-13_15-42-13.png) 


# Part 2 - Coming soon

In Part 2, I will describe the steps and gotchas of setting up continuous deployment to docker hub and hosting it in a free aws EC2 container.    
