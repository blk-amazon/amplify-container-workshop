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

Our pipeline is now ready to use our newly connected repository. Let's update our application and push our changes to our CodeCommit repository.

In your Cloud9 IDE, open the file **pages/index.js** and scroll down to line 16. For now, we're just going to make a simple change to test that our pipeline is able to detect the push to the repository and properly build and deploy our application. At the end of line 16, after the closing anchor tag (</a>) add a comma, followed by your name. For example:

```javascript
Welcome to <a href="https://nextjs.org">Next.js</a>, Brian
```

Now, commit your changes and push them to your CodeCommit repository.

```bash
git add .
git commit -m "updating application"
git push --set-upstream origin main
```

Return to your application pipeline in CodePipeline and you should soon see the source status update to "In progress".

![Published Application](/images/codepipeline-codecommit.png)

Once the pipeline completes, reload your application's URL and confirm that you can see that your updates have been published.

![Published Application](/images/published-application2.png)

Now that we have our application running through our pipeline on commits and then building and deploying in a serverless container, let's add some additional containers to make our application a little more interesting.

