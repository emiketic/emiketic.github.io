---
layout: post
title: 'React Native Continuous Delivery with GitLab CI/CD and Appetize'
author: nader
date: 2018-09-19 00:00:00
categories: react-native
tags: react-native continuous-delivery tooling
draft: true
---

At EMIKETIC, it is important for us to provide results as fast as possible and in a convenient and easy-to-use format.

Also as a service company, we're required to swiftly validate changes by fellow developers, QA officers, and/or clients.

Automated and continuous delivery is therefore a critical component for us. Is is very feasible for web apps, but might be more challenging for mobile apps.

## Our Approach to CI/CD

On most of our projects, our current approach to automated delivery is as follows:

- use [GitLab CI/CD](https://docs.gitlab.com/ee/ci/), a powerful continuous integration/delivery engine, to validate, test, build, and deploy any changes committed to codebase.
- use [Appetize](https://appetize.io/), an "online web based iOS Simulators and Android Emulators", to demonstrate changed within the comfort of web browser.
- provide a link to downloadable build for users to install and run on their devices.

This approach is better demonstrated by our [React Native Starter
](https://github.com/emiketic/emiketic-starter-react-native).

![Image of SendBird Dashboard](/assets/article_images/2018-09-19-react-native-ci-cd-with-gitlab-and-appetize/gitlab-ci-cd-pipeline.png)

### GitLab CI/CD

[GitLab CI/CD](https://docs.gitlab.com/ee/ci/) is probably one of most capable CI/CD engines out there, an it is a personal favorite.

We use GitLab CI/CD descriptor (`.gitlab-ci.yml`) combined with a number of custom scripts and [Docker images](https://hub.docker.com/r/emiketic/)

Having your code at GitLab is not a requirement since it is possible to mirror repository from other sources.

### Appetize

[Appetize](https://appetize.io/) is very convenient tool for actually running a mobile application without having to install it on a device. It supports multiple iOS and Android devices and versions.

<iframe src="https://appetize.io/embed/3xvgukkq4gqjyjn1ztrzq6czwr?device=nexus5&scale=75&autoplay=false&orientation=portrait&deviceColor=black" width="300px" height="597px" frameborder="0" scrolling="no"></iframe>

<iframe src="https://appetize.io/embed/nkn34mhpchnx172e67ptmjypdm?device=iphonex&scale=75&autoplay=false&orientation=portrait&deviceColor=black" width="307px" height="634px" frameborder="0" scrolling="no"></iframe>

<br>

We use Appetize' HTTP [API](https://appetize.io/docs#api-overview) to automatically upload available builds. This is done mainly the following command:

```sh
curl "https://${APPETIZE_TOKEN}@api.appetize.io/v1/apps/${APPETIZE_ID}" -F "file=@${TARGET}" -F "platform=${PLATFORM}" -F "note=${CI_COMMIT_SHA}"
```

Where `CI_COMMIT_SHA` is a git commit reference provided by GitLab CI/CD.

### Downloadable Build

Providing the ability to download and install the app enables involved parties to properly validate the application in real world usage.

As for now this is only valid for Android (APK).

For this, we use a simplistic solution which is a shared hosting that we upload builds to using FTP, mainly with following command:

```sh
curl -u "${FTP_USERNAME}:${FTP_PASSWORD}" "ftp://${FTP_DOMAIN}${FTP_PATH}/${CI_COMMIT_REF_SLUG}/" -T $TARGET
```

Where `CI_COMMIT_REF_SLUG` is a git reference (tag or branch) provided by GitLab CI/CD.

This solution good enough for us even when security/access is a constraint (we use `.htaccess`).

For more fine-grained control, we're looking into Google Drive among other suitable candidates like Dropbox, Amazon S3, ...

## What's next?

Automated delivery is not complete while actual deployment to stores remain manual. To accomplish this we're looking into [fastlane](https://fastlane.tools/) .. a subject for another blog post.
