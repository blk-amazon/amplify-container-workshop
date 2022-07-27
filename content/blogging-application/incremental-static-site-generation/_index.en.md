+++
title = "API with Incremental Static Site Generation (SSG)"
date = 2022-07-20T22:41:35-06:00
weight = 23
+++

Statically generating pages during the build process improves performance. But, dynamically created posts still need to not 404.

To solve this, create **pages/posts/[id].js** and paste in the following content:

```javascript
import { Amplify, API, withSSRContext } from 'aws-amplify';
import Head from 'next/head';
import { useRouter } from 'next/router';
import awsExports from '../../src/aws-exports';
import { deletePost } from '../../src/graphql/mutations';
import { getPost, listPosts } from '../../src/graphql/queries';
import styles from '../../styles/Home.module.css';

Amplify.configure({ ...awsExports, ssr: true });

export async function getStaticPaths() {
  const SSR = withSSRContext();
  const { data } = await SSR.API.graphql({ query: listPosts });
  const paths = data.listPosts.items.map((post) => ({
    params: { id: post.id }
  }));

  return {
    fallback: true,
    paths
  };
}

export async function getStaticProps({ params }) {
  const SSR = withSSRContext();
  const { data } = await SSR.API.graphql({
    query: getPost,
    variables: {
      id: params.id
    }
  });

  return {
    props: {
      post: data.getPost
    }
  };
}

export default function Post({ post }) {
  const router = useRouter();

  if (router.isFallback) {
    return (
      <div className={styles.container}>
        <h1 className={styles.title}>Loading&hellip;</h1>
      </div>
    );
  }

  async function handleDelete() {
    try {
      await API.graphql({
        query: deletePost,
        variables: {
          input: { id: post.id }
        }
      });

      window.location.href = '/';
    } catch ({ errors }) {
      console.error(...errors);
      throw new Error(errors[0].message);
    }
  }

  return (
    <div className={styles.container}>
      <Head>
        <title>{`${post.title} – Amplify + Next.js`}</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main}>
        <h1 className={styles.title}>{post.title}</h1>

        <p className={styles.description}>{post.content}</p>
      </main>

      <footer className={styles.footer}>
        <button onClick={() => { 
            window.location.href = '/';
        }}>Home</button>&nbsp;
        <button onClick={handleDelete}>💥 Delete post</button>
      </footer>
    </div>
  );
}
```

Let's walk through some of this file:

- **Amplify.configure** – For authenticated requests to work on the server, our client has to be configured with ssr: true to make credentials available on subsequent requests.
- **getStaticPaths** – Because the value of [id] isn't known at build-time, we need to provide all possible paths for Next.js to render pages for. We do this by querying all posts with API.graphql, and mapping each post into { params: { id: post.id }}. These params are passed to getStaticProps next. Note that we return fallback: true. If this value were false, then any new posts would return a 404 because they didn't exist at build-time. With true, static pages are returned quickly, while any new ids are looked up once.
- **getStaticProps** – Because the Post components props aren't dynamic per-request, they're provided statically from getStaticPaths as params. Here, we use params.id to query for that specific post with API.graphql.
- **Post** – Post is a functional component that renders the provided prop post. Because fallback: true was specified above, we have to account for a "loading" state while a new post may be fetched.
- **handleDelete** – This function is called when the "Delete post" button is clicked. Based on our configuration when we ran amplify add api, this value is defaulting to API_KEY.

For each request (req) on the server, we create a copy of Amplify (called SSR here from withSSRContext({ req })) that scopes credentials, data, and storage to just one request. API.graphql queries for a list of posts, and returns them as the posts prop for the Home component.
- **handleCreatePost** – This function is called when a logged-in user submits our "New Post" form. API.graphql is called to create the new post's title and content. Once created, we redirect to /posts/${post.id}.
- **Home** – This is a functional component that renders the posts provided by getServerSideProps.

With your server running (`npm run dev`), refresh the page (or relaunch using the **Preview Running Application** feature) and you'll see a page for this post!

🙌 Great job! You have successfully deployed your API and connected it with your app! Next, we'll deploy and host our application, so that it's available to the world!
