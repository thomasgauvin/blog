---
layout: post
title: How to store environment variables for local development
permalink: how-to-store-env-variables-for-local-development
date: 2021-11-12 01:07:00 +0000
description: Storing environment variables locally rather than in the code can prevent an unsecure exposure of your access keys, and it's quite easy to do! Here's how to store and use environment variables locally.

---
Storing environment variables locally rather than in the code can prevent an unsecure exposure of your access keys, and it's quite easy to do!

Have you ever received an alert from GitHub indicating that you've uploaded a secret to your public repository? Usually, this happens when we are developing an application and push our code while our secrets are still in the code.

![Managing alerts from secret scanning - GitHub Docs](https://docs.github.com/assets/images/help/repository/secret-scanning-resolve-alert-ghe.png "GitHub secret scanning warning")

When we push secrets to our public repositories, these secrets become accessible by anyone in the world! Anyone could decide to steal these keys and access your resources. This could result in them accessing your private data or unauthorized utilization of your resources.

In order to avoid this, it's best to start using environment variables early on in the development process. Turns out, it's very easy to store and use environment variables locally.

## Storing Environment Variables Locally

### Windows

In cmd, paste the following, replacing NAME-OF-ENV-VARIABLE with a name for the environment variable and VALUE-OF-ENV-VARIABLE with the value of your environment variable (such as an access key).

    set NAME-OF-ENV-VARIABLE=VALUE-OF-ENV-VARIABLE

You could use keyword 'setx' instead of 'setx'. As per [this StackOverflow response](https://superuser.com/questions/916649/what-is-the-difference-between-setx-and-set-in-environment-variables-in-windows "What is the difference between SETX and SET in environment variables in Windows"), 'setx' will set the environment variable permanently, whereas 'set' will modify for the current session.

### Linux/macOS

In bash/terminal, paste the following, replacing NAME-OF-ENV-VARIABLE with a name for the environment variable and VALUE-OF-ENV-VARIABLE with the value of your environment variable (such as an access key).

    export NAME-OF-ENV-VARIABLE="VALUE-OF-ENV-VARIABLE"

## Accessing the Environment Variables in your code

It's quite easy to access the environment variables in your code after that! It'll depend on the language that your are coding is, so your best bet is to check the official documentation.

In my case, I'm working in Python at the moment so I can access the environment variable this way:

    env_variable = os.getenv('NAME-OF-ENV-VARIABLE')

All in all, using environment variables earlier in your development process is very easy, and you won't have to change your code once you deploy your application if you plan on using environment variables.