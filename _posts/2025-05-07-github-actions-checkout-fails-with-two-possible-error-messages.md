---
title: GitHub actions/checkout action fails with two possible error messages
Author: Mickey Gousset
date: 2025-05-07 00:00:00 -0006
description: I talk about the error messages, why and when they occur, and how to fix them.
categories: [GitHub, Actions]
tags: [GitHub, Actions]
img_path: /assets/screenshots/2025-05-07-github-actions-checkout-fails-with-two-possible-error-messages
image:
  path: splash-image.png
  width: 100%
  height: 100%
  alt: "You may get two different error messages when the checkout action fails"
---

If you are using the GitHub [`actions/checkout`](https://github.com/actions/checkout) action in your [GitHub Actions workflow](https://docs.github.com/en/actions/writing-workflows/about-workflows), you may have run into one of two error messages:

1. **repository 'https://github.com/YourOrgName/YourRepoName/' not found**
2. **could not read Username for 'https://github.com': terminal prompts disabled**

These errors can be frustrating, especially if you are not sure what is causing them. It can be even more frustrating when it starts to happen in a workflow that was previously working. In this post, I will explain the two error messages, why they occur, and how to fix them. But first, we need to talk a little bit about workflow permissions.

{% include embed/youtube.html id='ZZxhzr3GGlY' %}

## Workflow Permissions

The [GITHUB_TOKEN](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication) has certain default permissions when running workflows in a repository:

- **Read and write permissions** - workflows have read and write permissions in the repository for all the scopes
- **Read repository contents and packages permissions** - workflows have read permissions in the repository for the contents and packages scopes only

These can be set at the repository, organization, or enterprise level. You can find this settings by going to **Settings > Actions > General** and scrolling down to the **Workflow permissions** section, shown in **Figure 1**.

![Figure 1. Workflow permissions settings](figure-01-workflow-permissions-settings.png){: .shadow }
_Figure 1. Workflow permissions settings_

Notice that it talks about "scopes". Scopes are the permissions that are granted to the GITHUB_TOKEN. You can find out more about the scopes in the [GitHub documentation](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token).  Here is a quick list of all the possible scopes:

- actions
- attestations
- checks
- contents
- deployments
- discussions
- id-token
- issues
- metadata
- models
- packages
- pages	
- pull-requests	
- security-events	
- statuses

Now a scope can be set to one of [three values](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#defining-access-for-the-github_token-permissions): `read`, `write`, or `none`. So, in the above settings, if we select **Read and write permissions**, then we are basically giving the GITHUB_TOKEN all the scopes with `read` and `write` permissions. If we select **Read repository contents and packages permissions**, then we are giving the GITHUB_TOKEN `read` permissions on the `contents` and `packages` scopes, and `none` on all the other scopes.

At this point, you may be asking yourself, "What does this have to do with the `actions/checkout` action?" The answer is that the `actions/checkout` action uses the GITHUB_TOKEN to authenticate with GitHub when checking out the repository. If the GITHUB_TOKEN does not have the correct permissions, then you will get one of the two error messages mentioned above.  But you are only going to potentially encounter this if you are modifying your permissions in the workflow file itself.

## Using the `permissions` key in the workflow file

You can set the permissions for the GITHUB_TOKEN in your workflow file using the [`permissions` key](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#overview). This key can be set at the workflow level or at the job level. Here is an example of how to set it at the workflow level:

```yaml
name: CI

on:
  workflow_dispatch:
  
permissions:
  issues: write
  pull-requests: write

jobs:
  demo-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
...
```

In this example, we are giving the GITHUB_TOKEN `write` permissions on the `issues` and `pull-requests` scopes. BUT, here is the catch: if you do not specify a permission for a scope, then it will default to `none`. This overrides the default permissions we talked about above.

So, if you do not specify a permission for the `contents` scope, then it will default to `none`, and you will get one of the two error messages mentioned above, because you no longer have permission to read the repository, and therefor the actions/checkout action cannot check out the repository.

Oh yeah, this does NOT apply to `public` repositories, it only applies to `internal` and `private` repositories. Public repositories always have `read` permissions for the `contents` scope.

Let's see how this relates to the two error messages above.

## Error Message: repository 'https://github.com/YourOrgName/YourRepoName/' not found

If you are using [GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud) and you run the workflow listed above, then you will get this error message, shown in **Figure 2**.

![Figure 2. Repository not found error message](figure-02-repository-not-found.png){: .shadow }
_Figure 2. Repository not found error message_

## Error Message: could not read Username for 'https://github.com': terminal prompts disabled

If you are using [GitHub Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users) and you run the workflow listed above, then you will get this error message, shown in **Figure 3**.

![Figure 3. Could not read Username for 'https://github.com'](figure-03-could-not-read-username.png){: .shadow }
_Figure 3. Could not read Username for 'https://github.com'_

Now, as to why you are getting one error message over the other, I'm not completely sure, but I suspect it has to do with the slightly different authentication methods used by GitHub Enterprise Cloud and GitHub Enterprise Managed Users.

## Fixing the error messages

Fixing this is easy. Just add the `contents` scope to the `permissions` key in your workflow file, and set it to `read`. Here is an example:

```yaml
name: CI

on:
  workflow_dispatch:
  
permissions:    
  contents: read
  issues: write
  pull-requests: write

jobs:
  demo-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
...
```

This will give the GITHUB_TOKEN `read` permissions on the `contents` scope, and you will no longer get the error messages.

## Summary

In this post, we talked about the two error messages you may get when using the `actions/checkout` action in your GitHub Actions workflow. We also talked about the permissions for the GITHUB_TOKEN and how to set them in your workflow file. Finally, we showed you how to fix the error messages by adding the `contents` scope to the `permissions` key in your workflow file.

If you are still having issues, please feel free to reach out to me on [X](https://x.com/mickey_gousset) or [LinkedIn](https://www.linkedin.com/in/mickeygousset/).

Thanks for reading!
