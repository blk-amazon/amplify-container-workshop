+++
title = "Deploy multiple containers"
date = 2022-07-24T19:26:56-06:00
weight = 33
+++

Next, let's add a NoSQL Database table named posts to our application.

```bash
amplify add storage
```

You can use the auto-generated value when prompted for a "friendly name" for your database. Make sure to name your table "posts" and then add the following columns and types to your "posts" table:
- id (number)
- title (String)
- description (String)

When prompted to select a **partition key** for your table, select the **id** column. We don't need a sort key, global secondary indexes, or Lambda Triggers for our application, so you can answer **No** for all three.

```bash
? Select from one of the below mentioned services: NoSQL Database

Welcome to the NoSQL DynamoDB database wizard
This wizard asks you a series of questions to help determine how to set up your NoSQL database table.

✔ Provide a friendly name · dynamoca6a8539
✔ Provide table name · posts

You can now add columns to the table.

✔ What would you like to name this column · id
✔ Choose the data type · number
✔ Would you like to add another column? (Y/n) · yes
✔ What would you like to name this column · title
✔ Choose the data type · string
✔ Would you like to add another column? (Y/n) · yes
✔ What would you like to name this column · description
✔ Choose the data type · string
✔ Would you like to add another column? (Y/n) · no

Before you create the database, you must specify how items in your table are uniquely organized. You do this by specifying a primary key. The primary key uniquely identifies each item in the table so that no two items can have the same key. This can be an individual column, or a combination that includes a primary key and a sort key.

To learn more about primary keys, see:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.PrimaryKey

✔ Choose partition key for the table · id
✔ Do you want to add a sort key to your table? (Y/n) · no

You can optionally add global secondary indexes for this table. These are useful when you run queries defined in a different column than the primary key.
To learn more about indexes, see:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.SecondaryIndexes

✔ Do you want to add global secondary indexes to your table? (Y/n) · no
✔ Do you want to add a Lambda Trigger for your Table? (y/N) · no
✅ Successfully added resource dynamoca6a8539 locally

⚠️ If a user is part of a user pool group, run "amplify update storage" to enable IAM group policies for CRUD operations
✅ Some next steps:
"amplify push" builds all of your local backend resources and provisions them in the cloud
"amplify publish" builds all of your local backend and front-end resources (if you added hosting category) and provisions them in the cloud
```

Now that we have a database for our application, let's add a REST API using the ExpressJS template and grant it access to our newly created DynamoDB table.

```bash
amplify add api
```

For this example, we'll deploy our ExpressJS template in a container deployed on Fargate, with an API Gateway proxy to our API. For our application, we'll let Amplify fully manage the countainer source for us. We also want to enable our API to be able to access our storage resource with full CRUD capabilities. To simplify this example, we're going to make our API public, so make sure to respond with **No** when the tool asks **Do you want to restrict API access**

{{% notice note %}}
We will need the name of our API later, so we recommend using **postscontainerapi** for your API name. Otherwise, you'll need to copy your API and paste it into your frontend code later.
{{% /notice %}}

```bash
? Select from one of the below mentioned services: REST
? Which service would you like to use API Gateway + AWS Fargate (Container-based)
? Provide a friendly name for your resource to be used as a label for this category in the project: postscontainerapi
? What image would you like to use ExpressJS - REST template
? When do you want to build & deploy the Fargate task On every "amplify push" (Fully managed container source)
? Do you want to access other resources in this project from your api? Yes
? Select the categories you want this function to have access to. storage
? Select the operations you want to permit on dynamoca6a8539 create, read, update, delete

You can access the following resource attributes as environment variables from your Lambda function
        STORAGE_DYNAMOCA6A8539_ARN
        STORAGE_DYNAMOCA6A8539_NAME
        STORAGE_DYNAMOCA6A8539_STREAMARN
? Do you want to restrict API access No
✅ Successfully added resource containerb71b0d64 locally.

✅ Next steps:
- Place your Dockerfile, docker-compose.yml and any related container source files in "amplify/backend/api/containerb71b0d64/src"
- Amplify CLI infers many configuration settings from the "docker-compose.yaml" file. Learn more: docs.amplify.aws/cli/usage/containers
- To access AWS resources outside of this Amplify app, edit the /home/ec2-user/environment/next-amplified/amplify/backend/api/containerb71b0d64/custom-policies.json
- Run "amplify push" to build and deploy your image
✅ Successfully added resource containerb71b0d64 locally

✅ Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud
```

{{% notice note %}}
Note the environment variables printed to the screen. You will need to update the TableName variable at the top of **amplify/backend/api/\<apiname\>/src/DynamoDBActions.js** with the environment variable.
{{% /notice %}}

```javascript
const TableName = process.env.STORAGE_<DB_FRIENDLY_NAME_>_NAME;
```

Finally run amplify push to deploy the backend:

```bash
amplify push
```

```bash
✔ Successfully pulled backend environment dev from the cloud.

    Current Environment: dev
    
┌──────────┬───────────────────┬───────────┬───────────────────┐
│ Category │ Resource name     │ Operation │ Provider plugin   │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Storage  │ dynamoca6a8539    │ Create    │ awscloudformation │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Api      │ containerb71b0d64 │ Create    │ awscloudformation │
└──────────┴───────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes

In a few moments, you can check image build status for containerb71b0d64 at the following URL:
https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines/${Token[TOKEN.367]}-containerb71b0d64-service-api-8080/view

It may take a few moments for this to appear. If you have trouble with first time deployments, please try refreshing this page after a few moments and watch the CodeBuild Details for debugging information.
⠹ Updating resources in the cloud. This may take a few minutes...
...

```

Once this completes your container will be built via an automated pipeline and deployed to Fargate Tasks on an ECS Cluster fronted by an Amazon API Gateway HTTP API using a direct Cloud Map integration to your VPC. If you selected Yes to protect your API with Authentication, an Amazon Cognito User Pool will be created with an Authorizer integration for that API.

