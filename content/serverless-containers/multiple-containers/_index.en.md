+++
title = "Deploy multiple containers"
date = 2022-07-24T19:26:56-06:00
weight = 33
+++

Our application is now building and deploying successfully in our serverless container, but it's a simple web page with no dynamic content. Let's add a NoSQL Database table named **posts** to our application, so we can start creating some blog posts.

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

With our database and API deployed, let's update our frontend to interact with our backend resources. Open the file **pages/index.js** in the Cloud9 IDE and replace it's contents with the following.

```javascript
// pages/index.js
import React from 'react';
import { Amplify, API, withSSRContext } from 'aws-amplify';
import Head from 'next/head';
import awsExports from '../aws-exports';

import styles from '../styles/Home.module.css';

Amplify.configure({ ...awsExports, ssr: true });

const apiName = "<YOUR API NAME HERE>";

export async function getServerSideProps({ req }) {
  const SSR = withSSRContext({ req });
  const response = await SSR.API.get(apiName, '/posts');

  return {
    props: {
      posts: response
    }
  };
}

export default function Home(props) {
  const [posts, setPosts] = React.useState(props.posts);
  
  async function handleCreatePost(event) {
    event.preventDefault();
  
    const form = new FormData(event.target);
  
    try {
      const options = {
          body: {
            id: Date.now(),
            title: form.get('title'),
            description: form.get('content')
          }
      };

      const response = await API.post(apiName, '/post', options);
      console.log(response);
      
      setPosts([
        ...posts,
        response
      ]);
  
    } catch ({ errors }) {
      console.error(...errors);
      throw new Error(errors[0].message);
    }
  }

  return (
    <div className={styles.container}>
      <Head>
        <title>Amplify + Next.js</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main}>
        <h1 className={styles.title}>Amplify + Next.js</h1>

        <p className={styles.description}>
          <code className={styles.code}>{posts.length}</code>
          posts
        </p>

        <div className={styles.grid}>
          {posts.map((post) => (
            <a className={styles.card} href={`/posts/${post.id}`} key={post.id}>
              <h3>{post.title}</h3>
              <p>{post.description}</p>
            </a>
          ))}

          <div className={styles.card}>
            <h3 className={styles.title}>New Post</h3>
            <form onSubmit={handleCreatePost}>
              <fieldset>
                <legend>Title</legend>
                <input
                  defaultValue={`Today, ${new Date().toLocaleTimeString()}`}
                  name="title"
                />
              </fieldset>

              <fieldset>
                <legend>Content</legend>
                <textarea
                  defaultValue="I built an Amplify app with Next.js!"
                  name="content"
                />
              </fieldset>

              <button type="submit" onSubmit={handleCreatePost}>Create Post</button>
            </form>
          </div>
        </div>
      </main>
    </div>
  );
}
```

Once you've updated and saved **pages/index.js**, we need to commit our changes and push them to our CodeCommit repository to start a new build/deploy.

```bash
git add .
git commit -m "added functionality to create and display blog entries"
git push
```

<hr />
# Old Stuff

The single Dockerfile scenario allows you to take an application running in a single Container which has been built with a Dockerfile and deploy it to AWS Fargate with the Amplify CLI.

If you are unfamiliar with using a Dockerfile review the Dockerizing a Node.js web app guide or or add an API with an Amplify-provided template.

A simple Dockerfile example is below, which would start a NodeJS application (index.js) in a built image by copying all the source files and installing dependencies. This example also shows how could can specify environment variables and use the EXPOSE statement for defining your container's communication port.

```docker
FROM public.ecr.aws/bitnami/node:14.15.1-debian-10-r8

ENV PORT=8080
EXPOSE 8080

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install
COPY . .

CMD [ "node", "index.js" ]
```

## Local development and testing

It is recommended to test your application locally first before deploying with amplify push, otherwise your Fargate Task may fail to start if there are application issues such as missing dependencies. With a Single Dockerfile you can do this by navigating to amplify/backend/api/<name>/src and running docker build -t to build and tag your image followed by docker run to launch your container similar to the below example:

```bash
cd ./amplify/backend/api/<name>/src
docker build -t node-app:1.0 .
docker run -p 8080:8080 -d node-app:1.0
curl -i localhost:8080  ## Alternatively open in a web browser
```

You can also run your application using standard tooling such as running node index.js or python server.py in Node or Python. Once you are satisfied with the Dockerfile and your application code, run amplify push and the amplify/backend/api/<name>/src directory will be bundled for the build pipeline to run and deploy your image to Fargate. At the end of the deployment the endpoint URL will be printed and client configuration files will be updated.

<hr />

