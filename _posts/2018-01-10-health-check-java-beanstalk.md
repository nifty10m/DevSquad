---
layout: post
title: Checking JVM health in AWS beanstalk
author-name: Jörg Herbst
author-avatar: joe
author-jobtitle: Cloud Ninja
---

We are running a lot of test and production instance on AWS. Most of them use the elastic beanstalk environment for this purpose.
The elastic beanstalk environment contains a build in load balancer and health check. So AWS is tearing down unhealthy instances
and creates new one (cattle over pets).

While this seems a good idea in general, the default settings just monitors the ec2 environment. This seems fine but in java
applications is much more important to monitor the java virtual machine (JVM) than the linux operating system.
For some reason this is not supported by default or via the web ui.

If you want to use the load balacing based on the health check you have to add a file
`ebextension/autoscaling.config`
```
Resources:
  AWSEBAutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300
```
More information could be found at [AutoScaling and Healthcheck](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environmentconfig-autoscaling-healthchecktype.html)
or at [Creating an Amazon EC2 Auto Scaling group.](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html)


{% include twitter.html %}
