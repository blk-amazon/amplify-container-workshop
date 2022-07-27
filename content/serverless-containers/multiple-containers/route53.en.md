+++
title = "Hosted Zone with Route53"
date = 2022-07-25T14:24:35-06:00
weight = 5
+++

As mentioned on the previous page, since our hosting container will be fronted by an Application Load Balancer, we will need to provide a public domain name when we add hosting to our application. To do this, we're going to create a hosted zone that will be a subdomain of an existing public domain, so that you don't need to buy or use one of your own domains. To begin, navigate to the Hosted Zone section of the **Route53** page in the AWS Console and click the orange **Create hosted zone** button.

![Create Hosted Zone](/images/create-hosted-zone.png)

{{% notice info %}}
When using sandbox accounts from Event Engine, some access permissions are disabled, so we may occasionally see errors and warnings in the AWS Console. For the purpose of this workshop, these can be safely ignored.
{{% /notice %}}