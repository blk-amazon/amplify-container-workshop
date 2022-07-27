+++
title = "Build Pipeline"
date = 2022-07-24T08:46:18-06:00
weight = 33
+++

Amplify creates APIs as an ECS Service to ensure that your application is monitored and tasks are in a healthy and active state, automatically recovering if an instance fails. When you make changes to your source code, the build and deployment pipeline will take your source code and Dockerfile/Docker Compose configuration as inputs. One or more containers will be built in AWS CodeBuild using your source code and pushed to ECR with a build hash as a tag, allowing you to roll back deployments if something unexpected happens in your application code. After the build is complete, the pipeline will perform a rolling deployment to launch Fargate Tasks automatically. Only when all new versions of the image are in a healthy & running state will the old tasks be stopped. Finally the build artifacts in S3 (in the fully managed scenario) and ECR images are set with a lifecycle policy retention of 7 days for cost optimization.