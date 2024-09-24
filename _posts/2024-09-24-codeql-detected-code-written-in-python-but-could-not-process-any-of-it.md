---
title: CodeQL detected code written in Python but could not process any of it
Author: Mickey Gousset
date: 2024-09-24 14:00:00 -0006
description: I talk about an error users may encounter with CodeQL when they have Python, and other languages, in hidden "." directories.
categories: [GitHub, GHAS]
tags: [GitHub, GHAS, CodeQL, Security, Code Scanning]
img_path: /assets/screenshots/2024-09-25-codeql-detected-code-written-in-python-but-could-not-process-any-of-it
image:
  path: splash-image.png
  width: 100%
  height: 100%
  alt: CodeQL detected code written in Python but could not process any of it
---

One of the features of [GitHub Advanced Security](https://docs.github.com/en/enterprise-cloud@latest/get-started/learning-about-github/about-github-advanced-security) is [code scanning](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning), specifically using [CodeQL](https://codeql.github.com/). In this post, I'm going to talk about an error you may encounter when you have code in a hidden folder, such as a `.` directory, and you are running CodeQL code scanning against it.

> The example in this post is related to python, but you can potentially see this same issue with other languages.

## The Error

When you run CodeQL code scanning against a repository, you may encounter an error that looks like the following:

```plaintext
Encountered a fatal error while running "/opt/hostedtoolcache/CodeQL/2.19.0/x64/codeql/codeql database finalize --finalize-dataset --threads=2 --ram=6914 /home/runner/work/_temp/codeql_databases/python". Exit code was 32 and last log line was: CodeQL detected code written in Python but could not process any of it. Check the workflow run logs (see https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs). For more information, review our troubleshooting guide at https://gh.io/troubleshooting-code-scanning/no-source-code-seen-during-build . See the logs for more details.
```

## The Explanation

More than likely, the problem is going to be that you have python code in a hidden directory, also referred to as a `.` (dot) directory. As far as I can tell, what is happening is the "Detection" of the languages doesn't seem to care if a file is in a hidden folder. However, the actual extraction of the code into the CodeQL database DOES care if the file is in a hidden folder. So, the detection of the language is successful, but the extraction of the code is not.  

## The Solution

The solution is to specify a [configuration file](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning#using-a-custom-configuration-file) that intentionally [tells CodeQL the directories to scan](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning#specifying-directories-to-scan).

Let's take this example file/folder structure:

```plaintext
.
├── main.py
└── .super-secret
    └── my-secret-file.py
```

If I ran CodeQL code scanning against this, I would get the error I showed above. To fix this, I would create a `codeql-configuration.yml` file and add the following:

```yaml
paths:
  - ./
  - .super-secret/
```

When I run my CodeQL scan, I give it the path to the configuration file. In the configuration file, I have specified that I want to scan the `./` directory and the `.super-secret/` directory. This ensures the CodeQL will extract and scan the code in the hidden directory.

## Conclusion

There ya go. Unfortunately, if you are using default code scanning, you can't specify a configuration file. You would have to use a custom CodeQL workflow (or be running scanning from the command line using [CodeQL CLI](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/about-the-codeql-cli)) so you can specify a configuration file.

Let me know what you think of this post by leaving a comment, hitting me up on Twitter/X, or send me an email.  Thanks for reading!
