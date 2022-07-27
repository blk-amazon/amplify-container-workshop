+++
title = "Install Prerequisites"
date = 2022-07-20T22:39:58-06:00
weight = 12
+++

## Configure Cloud9 Environment

In your Cloud9 environment, run the following commands in the terminal to install the pre-requisites (this may take a few minutes).

```bash
# Expand hard drive
curl -O https://gist.githubusercontent.com/blk-amazon/6fadc5a741d2683caaa6c3713cfdc221/raw/9df629dcff2847502eecd1ad5642d32d509f31a4/resize.sh
sh resize.sh 50
```
By default, Cloud9 is setup to use the latest NodeJS version. We need to switch to a previous version, in order to support all libraries and toolchains needed for this workshop.

```bash
nvm install lts/fermium
```

Next, we need to install the Amplify CLI - this is our toolchain for working with AWS services.

```bash
npm install -g @aws-amplify/cli
```

These commands will take a few minutes to complete. **You may receive some warnings and/or error messages, that is typically ok.**

Now it's time to setup the Amplify CLI. Configure Amplify by running the following command:
```bash
amplify configure
```

The ```amplify configure``` command will ask you to specify the region and user name to use when working with your Amplify project. For region, make sure that **us-east-1** is selected and for user name, you can leave the default value. Once you have configured those two values, the configure tool should display a link to continue to the process of creating your user. Open that link in your browser by click the link and selecting the the **Open** option.

![Amplify Configure](/images/amplify_configure.png)

This will open the AWS Console to the Identity and Access Management (IAM) service page with your new Amplify user partially configured. Click the blue **Next:** buttons at the bottom of the screen to advance through the wizard, making sure to add the **AdministratorAccess** Policy on the permissions screen. The wizard should pre-select the **AdministratorAccess-Amplify** Policy, so make sure you have both selected.

![IAM Set Permissions](/images/iam-set-permissions.png)

Click the blue **Create User** button to create your new Amplify user, making sure to note the **Access key ID** and  **Secret access key** values on the next screen, as you will need them in the final step.

![IAM Access Keys](/images/iam-access-keys.png)

Lastly, you'll need to copy/paste the two access key values that you noted in the prior step back into your Cloud9 environment terminal. This will create an AWS profile named default that Amplify will use to interact with the AWS services in your account.

![Cloud9 Access Keys](/images/cloud9-access-keys.png)

