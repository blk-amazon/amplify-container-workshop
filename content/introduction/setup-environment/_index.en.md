+++
title = "Setup Dev Environment"
date = 2022-07-20T22:40:09-06:00
weight = 11
+++

AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It includes a code editor, debugger, and terminal. Cloud9 comes prepackaged with essential tools for popular programming languages, including JavaScript, Python, PHP, and more, so you don’t need to install files or configure your development machine to start new projects.

{{% notice warning %}}
Ad blockers, JavaScript disablers, and tracking blockers should be disabled for the cloud9 domain, otherwise connecting to the workspace might be impacted.
{{% /notice %}}

## Create a new environment
1. Go to the Cloud9 web console.
2. At the top right corner of the console, make sure you’re using this region: Virginia (us-east-1)
3. Select Create environment
4. Name it AmplifyWorkshop, and go to the Next step
5. Select Create a new instance for environment (EC2) and pick m5.large
6. Change Cost-saving setting to After a week
7. Leave all of the other environment settings as they are, and click on Next step to proceed to the Review page
8. If everything looks good, click Create environment

## Access newly created environment

If everything completed successfully, when the environment comes up, you should see a Welcome tab. You can optionally configure the theme for your IDE, otherwise, close the tab and you’re ready to begin working!

![Cloud9 Complete](/images/cloud9_complete.png)
