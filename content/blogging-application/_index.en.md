+++
title = "Blogging Application"
date = 2022-07-20T22:40:54-06:00
weight = 20
+++

Now that you’ve created and configured a Next.js app and initialized a new Amplify project, we can begin building our blogging application. We'll start by adding a GraphQL API to our application, in order to provide basic CRUD functionality.

The Amplify CLI supports creating and interacting with two types of API categories: REST and GraphQL.

The API you will be creating in this step is a GraphQL API using AWS AppSync (a managed GraphQL service) and the database will be Amazon DynamoDB (a NoSQL database).