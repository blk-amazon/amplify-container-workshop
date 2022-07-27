+++
title = "Deploy a single container"
date = 2022-07-24T08:22:14-06:00
weight = 32
+++

In order to update our pipeline to kick-off builds when it detects a new commit, we first need to create and link a repository to our pipeline. We're going to leverage AWS CodeCommit, but you can also use other Git providers such as Github and Bitbucket, as well as other sources such as Jenkins.

First, navigate to the CodeCommit page within the AWS Console and click the orange **Create repository** button. Give your repository a name such as `nextamplified` and then click the orange **Create** button. This will create a Git repository that we can then use to commit our Next.js application to.

![CodeCommit Create Repository](/images/codecommit-create-repo.gif)

Once your repository has been created, find the **Clone URL** dropdown at the top right corner of the repository page and select the **Clone HTTPS** option. This will copy the repository URL to your clipboard.

Next, return to your Cloud9 environment and, first making sure that you are in the Next.js project folder, in the terminal type `git remote add origin [CodeCommit URL]`, replacing [CodeCommit URL] with the value your copied previously. This will add a new remote to your project's Git config.

Our Git repository is now created and our local codebase is connected to it. Next, let's update our pipeline and connect our new repository as it's source. Navigate back to our pipeline in AWS CodePipeline, and click the **Edit** button. This will take us to a view where we can edit each of the stages of our pipeline. Click on the **Edit stage** button in the **Source** stage and then click on the **+ Add action** button that appears. This will bring up a view where we can specify the **Action provider** and any settings associated with that provider.

![CodePipeline Edit Source Action](/images/codepipeline-source-action.png)

Give your action a name, such as CodeCommitSource and select **AWS CodeCommit** for the Action provider. Next, select your repository name and the Main branch and input **SourceArtifact** for the Output artifact name. You can leave all other settings in their default values. Click the orange **Done** button.

![CodePipeline Edit Source Action](/images/codepipeline-extra-source.png)

You'll notice that we now have two actions for our Source stage. We can now delete the original S3 source, so that we're only left with our new CodeCommit source, by clicking the **X** at the bottom right corner of the **Amazon S3** source box. Click the **Done** button for our Source stage and then the orange **Save** button at the top of the screen, followed by the orange **Save** button on the **Save pipeline changes** confirmation dialog.

We need to update our pipeline's service role permissions to grant it access to our repository. On the left-hand side menu in AWS CodePipeline, click on the **Settings** menu item under **Pipeline > Pipelines > Settings**. From the Settings screen, locate the **Service role ARN** link and open it in a new window.

![CodePipeline Service Role Permissions](/images/pipeline-service-role-permissions.png)

Click on the **Add permissions** button and select **Attach policies**. Under the **Other permissions policies** section, search for the policy named **AWSCodeCommitFullAccess** and click the checkbox next to it, followed by the blue **Attach policies** button.

![CodeCommit Full Access Policy](/images/codecommit-full-access-policy.png)

Our pipeline is now ready to use our newly connected repository.

Now that we have our database and API deployed, let's update our frontend to interact with our backend resources. Open the file **pages/index.js** in the Cloud9 IDE and replace it's contents with the following.

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