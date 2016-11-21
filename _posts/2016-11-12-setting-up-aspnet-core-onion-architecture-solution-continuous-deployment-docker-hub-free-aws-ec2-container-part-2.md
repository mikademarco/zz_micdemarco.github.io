---
author: micdemarco
comments: true
date: 2016-11-19 02:50:15+00:00
layout: post
slug: setting-up-aspnet-core-onion-architecture-solution-continuous-deployment-docker-hub-free-aws-ec2-container-part-2
title: Setting up aspnet core onion architecture solution with CD using docker hub + free aws EC2 container - Part 2
tags:
- aspnet core
- github
- docker
- aws
- container
- onion architecture
- free
---

# Step 4 - Create a docker hub account and AWS account

1. Sign up for a free docker hub account [here](https://hub.docker.com//)

2. Sign up for a free AWS account [here](https://aws.amazon.com) (note you will need to enter your credit card details)   

# Step 5 - Link Docker hub with AWS and GitHub

1. Follow [this](https://docs.docker.com/docker-cloud/infrastructure/link-aws/) guide from docker on how to link your docker account with AWS

2. In the [docker console](https://cloud.docker.com) under [ Cloud Settings ] -> [ Source Providers ] connect your BitBucket/GitHub account 

When you are finished you will be able to see all the linked providers under Cloud Settings.  It should looks like this:

![Linked services on Docker Hub](/assets/2016-11-21_22-09-39.png) 

# Step 6 - Create a cluster in Docker Hub

1. Go to [ Infrastructure ] -> [ Node Clusters ] and click Create

2. Enter a name, provider, region and select a size for your node cluster
 *Note - At time of writing the AWS free tier allows you to run a t2.micro instance for 750 hrs per month*
![Node create settings](/assets/2016-11-21_22-29-18.png)

Once it is deployed you will see it under node clusters :
![Deployed node cluster](/assets/2016-11-21_22-59-18.png)

# Step 7 - Create a repository and link it to GitHub

1. Go to repositories and click create
2. Enter a name for the reporitory
3. Select a repository that you would like to publish and choose a source branch
![Repository settings](/assets/2016-11-21_22-53-16.png)

4. Initially there will be no builds available.  You can trigger a build by clicking the spanner in the builds tab or by committing some code to the branch.
![Build settings](/assets/2016-11-21_23-05-08.png)

*Gotcha - If you are using dotnet:latest as your base docker image, it could be that the latest sdk+runtime are more recent than your app's and may not work.  To be sure the image will work, you can specify the exact version of sdk that you want to use:*

```
FROM microsoft/dotnet:1.0.1-sdk-projectjson
```

# Step 8 - Run the image in a container

Once the container has been built you will be able to run the built docker image in a container.  Steps to do this:

1. Open the repository and click the [ Launch Service ] button on the General tab
2. Set autoredeploy to true.  This will ensure the container will be deployed with the latest image every time one gets built
![Autoredeploy on](/assets/2016-11-22_08-47-22.png)
3. Set the port to published and enter a node port.  This will tell the container to map port 5000 to external port port 8080
![Publish port](/assets/2016-11-22_08-55-05.png)
4. Click [ Create & Deploy ].  The service will show up as running in the services page
![Service running](/assets/2016-11-22_09-14-23.png)
5. Test the image using the service endpoint. Read [here](https://docs.docker.com/docker-cloud/apps/ports/) about the difference between container and service endpoints
![Endpoint working](/assets/2016-11-22_09-22-44.png) 

# Step 9 - Redeploy

The repository has now set up for Continuous Deployment.  If a change is made to the source control, Docker will automatically build the image and update the container with the latest build.

1. Let's change the format of the user name to 'Fraz-...'
```csharp
public async Task<User> GetAsync(int id)
{     
    // TODO: replace with an async database operation  
    return new User()
    {
        Id = id,
        Name = $"Fraz-{id}-{Guid.NewGuid()}"
    };
}
```
2. Commit and push changes
3. Check the builds tab in docker
![Building](/assets/2016-11-22_09-34-56.png)
4. Once the build is done. Test
![Changes](/assets/2016-11-22_09-37-51.png)

# Conclusion

Microsoft have done a great job of getting docker support for dotnet and aspnet core with their base docker images.  

Docker Hub is also very easy to integrate with AWS, Azure, GitHub and BitBucket and by using the free tier of AWS, it is easy to have a hosted repository continuously deployed free of charge.

The free tier is limited, and as soon as you want to run more services (which is one of the main reasons for running containers), you will need to either pay or make sure you do not exceed the 750hrs of contianer runtime that AWS free tier allows you.

Either way it great to be able to get started using containers in the cloud at no cost, and then scale accordingly.