+++
title = "Getting Started"
date = 2022-07-24T08:21:52-06:00
weight = 31
+++

Serverless containers are not enabled in your Amplify CLI project by default. To get started you will need to run amplify configure project in order to see the options for deploying to Fargate. To get started initialize your project and enable container-based deployments:

```bash
amplify configure project
```

Amplify will ask you to confirm that you want to enable container-based deployments.
```bash
> Do you want to enable container-based deployments? Yes
```

Our project is now configured to use container-based deployments for various services. Let's begin by adding hosting for our app and deploying it in a serverless container on AWS Fargate.

## Hosting

When using containers in the `amplify add hosting` workflow the setup will be largely the same, including the ability to define your backend with a single Dockerfile or Docker Compose file yaml. However the ECS cluster will be fronted by an Application Load Balancer (ALB) and CloudFront distribution, and you will be required to provide a domain name which you own. This can either be a domain which you have purchased on a 3rd party registrar or with Route53. The domain will be used with Amazon Certificate Manager to configure SSL between ALB and Cognito User Pools to perform authorization to your website hosted on Fargate containers.

> Hosting with Fargate in Amplify is only available in US-East-1 at this time

## Route53
Navigate to **Route53** page in AWS Console and then click on the **Hosted zones** item on the left-hand side menu. Look for a domain that ends with **awsps.myinstance.com**. This is your public domain for this workshop. Copy the full domain, as we'll need it in the next step.

Now we're ready to add hosting to our application.

```bash
amplify add hosting
```

The Amplify CLI will now take you through the hosting configurations. When you are prompted to provide the endpoint for your app, enter **nextamplified.** followed by the domain you copied from Route53. Don't forget the the period between nextamplified and the domain you're pasting. For example: **nextamplified.graebel1.blk.awsps.myinstance.com**

```bash
A registered domain is required. 
You can register a domain using Route 53: aws.amazon.com/route53/ or use an existing domain.

? Provide your web app endpoint (e.g. app.example.com or www.example.com): nextamplified.graebel1.blk.awsps.myinstance.com
? Do you want to automatically protect your web app using Amazon Cognito Hosted UI No
? Do you want to access other resources in this project from your container? Yes
? Select the categories you want this function to have access to. storage
? Select the operations you want to permit on dynamoca6a8539 create, read, update, delete

You can access the following resource attributes as environment variables from your Lambda function
        STORAGE_DYNAMOCA6A8539_ARN
        STORAGE_DYNAMOCA6A8539_NAME
        STORAGE_DYNAMOCA6A8539_STREAMARN
```

We need to update the Dockerfile that was created by Amplify and placed into the root of our project. Open **next-amplified/Dockerfile** and replace the contents with the following.

```docker
FROM public.ecr.aws/bitnami/node:14-prod-debian-10

ENV PORT=8080
EXPOSE 8080

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

CMD ["npm", "start"]
```

Now, let's push our changes to Amplify and publish our app to Fargate.

```bash
amplify publish
```

```
Overrides functionality is not implemented for this category
✔ Successfully pulled backend environment dev from the cloud.
Overrides functionality is not implemented for this category

    Current Environment: dev
    
┌──────────┬───────────────────┬───────────┬───────────────────┐
│ Category │ Resource name     │ Operation │ Provider plugin   │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Hosting  │ ElasticContainer  │ Create    │ awscloudformation │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Storage  │ dynamoca6a8539    │ No Change │ awscloudformation │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Api      │ containerb71b0d64 │ No Change │ awscloudformation │
└──────────┴───────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes
Overrides functionality is not implemented for this category
⠙ Updating resources in the cloud. This may take a few minutes...
...
✔ All resources are updated in the cloud

Publish started, you can check the status of the deployment on:
https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines/amplify-nextamplified2-dev-215934-site-service-api-8080/view
```

What's happening here is that when we issued the `amplify publish` command, Amplify noticed that there were also changes to the backend environment that needed to be provisioned before the application could be published. Once all resources have been updated in the cloud, the publish command will begin. Amplify uses CloudPipeline to build and deploy our application in our serverless container and you'll see a link to the pipeline that you can use to check the status of the deployment. Let's have a look at what's going on in our pipeline.

![Amplify Hosting Pipeline](/images/amplify-hosting-pipeline.png)

As you can see, Amplify created a 4 stage pipeline. Let's look a little closer at what each of these stages are doing.

- **Source**: This is where our pipeline begins. In this example, it's looking for a new file (in this case, a zip file containing our application source code) in an S3 bucket. When you issue the `amplify publish` command, Amplify will zip up our source and upload it to S3, which CodePipeline will then detect and kick-off our build.
- **Build**: This is the actual build process. In our example, it's parsing the **buildspec.yml** file in our project that Amplify created for us and running the phases contained within our buildspec definition.
- **Predeploy**: This is an helper stage that Amplify created for us that uses a Lambda function to prepare our container for deployment.
- **Deploy**: This it the final stage where Codepipeline takes that build artifact from our **Build** stage and actually deploys it in ECS.

With one simple command, Amplify created a complete pipeline to download, build, and deploy our application! Navigate to the domain that you provided for your hosting configuration, so we can confirm that our application has been published and is working as expected.

![Published Application](/images/published-application1.png)


While this is great, most teams leverage source control workflows for initiating builds, so let's see how we can improve this pipeline by integrating with a Git repository and start our build and deploy process when a new commit occurs on our Main branch.

