+++
title = "API with Server-Side Rendering (SSR)"
date = 2022-07-20T22:41:46-06:00
weight = 22
+++

In this section you will create a way to list and create posts from the Next.js application. To do this, you will fetch & render the latest posts from the server as well as create a new post on the client.

Open pages/index.js and replace it with the following code:

```javascript
// pages/index.js
import React from 'react';
import { Amplify, API, withSSRContext } from 'aws-amplify';
import Head from 'next/head';
import awsExports from '../src/aws-exports';
import { createPost } from '../src/graphql/mutations';
import { listPosts } from '../src/graphql/queries';
import styles from '../styles/Home.module.css';

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
        </div>
      </main>
    </div>
  );
}
```

Let's walk through some of this file:

- **Amplify.configure** – For authenticated requests to work on the server, our client has to be configured with ssr: true to make credentials available on subsequent requests.

- **getServerSideProps** – For each request (req) on the server, we create a copy of Amplify (called SSR here from withSSRContext({ req })) that scopes credentials, data, and storage to just one request. API.graphql queries for a list of posts, and returns them as the posts prop for the Home component.

- **handleCreatePost** – This function is called when a logged-in user submits our "New Post" form. API.graphql is called to create the new post's title and content.

- **Home** – This is a functional component that renders the posts provided by getServerSideProps.

## Testing SSR

Next, run the app and you should see a landing page with 0 posts with a login screen:

```bash
npm run dev
```

You can now use the **Preview Running Application** feature that we saw earlier to see your updated application.

In the application window, you should see a form titled **New Post** that will enable you to begin authoring new blog posts. After you've created some entries, you may have noticed that clicking on one of the blog cards displays a 404 error. That's because, thus far, we've been working strictly with statically generated content. We'll see how to address this issue in the next section.
