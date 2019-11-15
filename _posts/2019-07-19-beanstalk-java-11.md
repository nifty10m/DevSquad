---
layout: post
title: Run your Java 11 Application on AWS Elastic Beanstalk
author: sja
author-jobtitle: Cloud Baron
---

When running Applications on AWS Beanstalk you are limited to the supported "platforms" (see [Supported AWS Plattforms](https://docs.aws.amazon.com/de_de/elasticbeanstalk/latest/dg/concepts.platforms.html)). 
At the time of this writing your Java Applications are capped to JDK-8. 

While the current long time release of java is 11 and currently at JDK-12 is available and JDK-13 announced to be released in the end of 2019, we needed to find a way to deploy and run JDK-8+ Applications on AWS.

The first solution which came to our minds was to build Docker-Images / -Containers and ship them via [AWS-ECS](https://docs.aws.amazon.com/de_de/AmazonECS/latest/developerguide/Welcome.html). 

Sounds simple, but it's a real pain to configure those environments and even provide a stable and shared persistence layer. We tried to get this running and aborted about 2 days later. 
Our next approach was to run a Docker-Image using the Elastic-Beanstalk Environment and we got this running in about 4 hours. It was a pleasant surprise, how easy and intuitive it was.

So here's a small How-To ship your Application using Docker on AWS Elastic Beanstalk with Gradle and Gitlab-Ci

# Preconditions

* An executable Jar-File which launches your Application (e.g. a Spring-Boot-Application).

# Create a Dockerfile of your Application
We thought it would be a good idea to start with JDK-11 due to the fact, this is a long time release and there are several base images available. 

```dockerfile
FROM openjdk:11-alpine

LABEL maintainer="10M (https://10m.de/)"

ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 5000

ADD build/libs/world-domination-planner*.jar app.jar 
```

# Build and Upload your Docker-Container in your CI-Environment
First of all, you have to think about, where you should publish your Docker-Container. Our goal is to deploy our Docker-Image to AWS so we are using the [AWS-ECR](https://aws.amazon.com/de/ecr/) which is some Sort of Docker-Image Registry where you just push your images to.

## Create AWS-ECR for your project
Just go to your AWS-Console - ECR - Create Repository. The only thing you need for this is a Repository name. AWS gives you a link to the Repository, like 
`0000000001.dkr.ecr.eu-central-1.amazonaws.com/world-domination-planner`.

## Build and Upload your Docker-Container
We are using Gitlab-Ci and so the build-step for publish our docker image is:
```yaml
variables:
  ECR_URI: 0000000001.dkr.ecr.eu-central-1.amazonaws.com/world-domination-planner

...

build-upload-docker:
  image: jansauer/docker-plus-awscli:1.16.198
  stage: publish
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_DRIVER: overlay2
  services:
    - docker:dind
  script:
    # login to AWS-ECR
    - $(aws ecr get-login --no-include-email --region eu-central-1) 
    
    # build the docker image
    - docker build
      # tag the docker image with either the Git-tag or if not available the latest commit hash
      --tag ${ECR_URI}:${CI_COMMIT_TAG:-$CI_COMMIT_SHORT_SHA}
      --label "version=${CI_COMMIT_TAG}"
      --label "commithash=${CI_COMMIT_SHA}" .
    
    # push the docker image
    - docker push ${ECR_URI}:${CI_COMMIT_TAG:-$CI_COMMIT_SHORT_SHA}
  only:
    - master
    - tags
```

`${CI_COMMIT_TAG}` and `${CI_COMMIT_SHORT_SHA}` are just Environment variables which are provided by Gitlab-Ci. 
You can chose your tag however you want.

## Create Beanstalk Environment
Go to your AWS-Console - Elastic Beanstalk.

Create an Application and create an Environment:

* Web server environment
* Environment name: `world-domination-planner-production`
* Preconfigured platform: `Docker`
* Application code: `Sample Application`

After that, you can configure your Environment correctly, like Adding a RDS-Database, Enable AWS-Cloudwatch for logs, Adding https to your Load-Balancer, ..

## Create Dockerrun.aws.json
To deploy your application to Beanstalk, you have to describe, where Beanstalk finds your Docker-Container and how it should run it. That's made in the [Dockerrun.aws.json](https://docs.aws.amazon.com/de_de/elasticbeanstalk/latest/dg/single-container-docker-configuration.html) which is the only thing you need to deploy to Beanstalk

```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "0000000001.dkr.ecr.eu-central-1.amazonaws.com/world-domination-planner:0.0.1",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "5000"
    }
  ]
}
```
We wrote a Gradle task, which generates the Dockerrun.aws.json with the correct Ttg (Version). To simplify this How-To let's say we tagged the image with Version 0.0.1

## Deploy your Dockerrun.aws.json

You can now just upload your Dockerrun.aws.json to your Beanstalk Environment and Beanstalk will create a new EC2-Instance which then will run your Docker-Container

We wrote ourselves a Gitlab-Ci-Job for that. For the deployment we use the [Elastic Beanstalk-CLI (EB CLI)](https://docs.aws.amazon.com/de_de/elasticbeanstalk/latest/dg/eb-cli3-getting-started.html)

```yaml
deploy-docker-production:
  image: jansauer/docker-plus-awscli:1.16.198
  stage: deploy
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_DRIVER: overlay2
  services:
    - docker:dind
  script:
    # got to the home of your Dockerrun.aws.json
    - cd build/generated-remote-docker
    
    # init the beanstalk-application in the folder world-domination-planner
    - eb init -p docker --region eu-central-1 world-domination-planner
    
    # deploy to your beanstalk-environment
    # --label defines the file name of your Version, which will be uploaded to AWS-S3
    - eb deploy --label world-domination-planner-${CI_COMMIT_TAG} world-domination-planner-production
  tags:
    - privileged
  only:
    - tags
``` 

# Wrap up
Its fairly easy to deploy your published Docker-Images to Beanstalk by using the EB CLI. After you understand, which steps you need to take to build and deploy, it's rather simple to automate it and integrate it to your CI.

{% include twitter.html %}
