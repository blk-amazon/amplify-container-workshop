+++
title = "Authentication and access"
date = 2022-07-20T22:41:01-06:00
weight = 25
+++

Our blogging application is working pretty well, but currently anyone can access our application and create or even delete any blog entry. This presents a serious security and usability problem. Fortunately, Amplify makes it very easy to add authentication and then provide granular access controls.

Let's begin by updating our GraphQL schema. Open the file **amplify/backend/api/nextamplified/schema.graphql** in your Cloud9 IDE and paste in the following:

```graphql
type Post
  @model
  @auth(rules: [{ allow: owner }, { allow: public, operations: [read] }]) {
  id: ID!
  title: String!
  content: String!
}
```

Notice that this updated schema is nearly identical to our original schema! In fact, the only difference is on the line that begins with @auth. What we've done here is removed Create, Update, and Delete permissions from **public** access and added an **allow: owner** directive, which grants full access to a Post to the owner of that Post.

After you have saved your updated schema, let's check in with Amplify and see how the updated schema has impacted Amplify's understanding of the project status.

```bash
amplify status
```

```bash
> Current Environment: dev
    
┌──────────┬────────────────┬───────────┬───────────────────┐
│ Category │ Resource name  │ Operation │ Provider plugin   │
├──────────┼────────────────┼───────────┼───────────────────┤
│ Api      │ nextamplified  │ Update    │ awscloudformation │
├──────────┼────────────────┼───────────┼───────────────────┤
│ Hosting  │ amplifyhosting │ No Change │                   │
└──────────┴────────────────┴───────────┴───────────────────┘
```

As you can see, Amplify detected that there was a change needed in the API due to our schema change. Let's go ahead and update our environment in the cloud with these updates.

```bash
amplify push
```

```bash
> Current Environment: dev
    
┌──────────┬────────────────┬───────────┬───────────────────┐
│ Category │ Resource name  │ Operation │ Provider plugin   │
├──────────┼────────────────┼───────────┼───────────────────┤
│ Api      │ nextamplified  │ Update    │ awscloudformation │
├──────────┼────────────────┼───────────┼───────────────────┤
│ Hosting  │ amplifyhosting │ No Change │                   │
└──────────┴────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes

⚠️  WARNING: your GraphQL API currently allows public create, read, update, and delete access to all models via an API Key. To configure PRODUCTION-READY authorization rules, review: https://docs.amplify.aws/cli/graphql/authorization-rules

Cognito UserPool configuration
Using service: Cognito, provided by: awscloudformation
 
 The current configured provider is Amazon Cognito. 
 
 Do you want to use the default authentication and security configuration? (Use arrow keys)
❯ Default configuration 
  Default configuration with Social Provider (Federation) 
  Manual configuration 
  I want to learn more. 
```

As you can see, Amplify detected that we needed to add authentication due to our updated @auth directive in the schema and automatically began the process of adding Amazon Cognito as our provider. Select all of the default settings for Cognito.

```bash
 Do you want to use the default authentication and security configuration? Default configuration
 Warning: you will not be able to edit these selections. 
 How do you want users to be able to sign in? Username
 Do you want to configure advanced settings? No, I am done.
✅ Successfully added auth resource nextamplified22eac13d locally

✅ Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud


⚠️  WARNING: your GraphQL API currently allows public create, read, update, and delete access to all models via an API Key. To configure PRODUCTION-READY authorization rules, review: https://docs.amplify.aws/cli/graphql/authorization-rules

✅ GraphQL schema compiled successfully.

Edit your schema at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema.graphql or place .graphql files in a directory at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema

⚠️  WARNING: your GraphQL API currently allows public create, read, update, and delete access to all models via an API Key. To configure PRODUCTION-READY authorization rules, review: https://docs.amplify.aws/cli/graphql/authorization-rules

✅ GraphQL schema compiled successfully.

Edit your schema at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema.graphql or place .graphql files in a directory at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema
⠸ Building resource api/nextamplified
⚠️  WARNING: your GraphQL API currently allows public create, read, update, and delete access to all models via an API Key. To configure PRODUCTION-READY authorization rules, review: https://docs.amplify.aws/cli/graphql/authorization-rules

⠼ Building resource api/nextamplified✅ GraphQL schema compiled successfully.

Edit your schema at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema.graphql or place .graphql files in a directory at /home/ec2-user/environment/next-amplified/amplify/backend/api/nextamplified/schema
? Do you want to update code for your updated GraphQL API Yes
? Do you want to generate GraphQL statements (queries, mutations and subscription) based on your schema types?
This will overwrite your current graphql queries, mutations and subscriptions Yes
⠼ Updating resources in the cloud. This may take a few minutes...
...

```

This will add the authentication backend to our application, but we also need to update the frontend code to provide UI for signup, signin, forgot/reset password, MFA, etc. If you have implemented auth flows in an app before, you know that this can be quite complicated. Fortunately, Amplify makes this extremely easy. In your Cloud9 IDE, update **pages/index.js** with the following:

```javascript
// pages/index.js
import React from 'react';
import { Authenticator } from '@aws-amplify/ui-react';
import { Amplify, API, withSSRContext } from 'aws-amplify';
import Head from 'next/head';
import awsExports from '../src/aws-exports';
import { createPost } from '../src/graphql/mutations';
import { listPosts } from '../src/graphql/queries';
import styles from '../styles/Home.module.css';
import '@aws-amplify/ui-react/styles.css';


Amplify.configure({ ...awsExports, ssr: true });

export async function getServerSideProps({ req }) {
  const SSR = withSSRContext({ req });
  const response = await SSR.API.graphql({ query: listPosts });

  return {
    props: {
      posts: response.data.listPosts.items
    }
  };
}

export default function Home(props) {
  const [posts, setPosts] = React.useState(props.posts || []);
  
  async function handleCreatePost(event) {
    event.preventDefault();
  
    const form = new FormData(event.target);
  
    try {
      const { data } = await API.graphql({
        authMode: 'AMAZON_COGNITO_USER_POOLS',
        query: createPost,
        variables: {
          input: {
            title: form.get('title'),
            content: form.get('content')
          }
        }
      });
      // console.log(data);
      
      setPosts([
        ...posts,
        data.createPost
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
              <p>{post.content}</p>
            </a>
          ))}

          <Authenticator>
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

              <button>Create Post</button>
            </form>
          </div>
          </Authenticator>
        </div>
      </main>
    </div>
  );
}
```

If you look closely, you'll see that we only updated 4 lines of code: Lines 3, 36, 81, and 104. Here's a brief description of each change.
- **Line 3** - We're importing the Authenticator component from Amplify's React UI library. This component provides all of the UI needed for authentication including Sign up, Sign in, and Forgot/Reset Password screens.
- **Line 36** - Here, we're telling our **createPost** GraphQL query to use the newly created Cognito User Pool as the authMode for this request. This is needed because our schema updates from earlier now limit the creation of Posts to authenticated users only.
- **Lines 81 and 104** - These lines wrap our **New Post** form inside our Authenticator component, which prevents non-authenticated users from accessing the form to create Posts.

Let's go ahead and restart our development server, so we can see our changes.

```bash
npm run dev
```

Remember to use the **Preview Running Application** feature to see our application.

![Amplify Authentication](/images/amplify-authentication.png)

