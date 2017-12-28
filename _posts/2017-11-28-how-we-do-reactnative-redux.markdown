---
layout: post
title:  "How we do React/Redux"
author: chourouk
date:   2017-11-28 10:27:00
categories: react
tags: react-native redux
image: /assets/article_images/2016-05-04-create-your-own-download-manager/e.jpg
---

Before launching any project a several of questions get in you mind, what i’m going to use? why this? How can i make a better architecture in the application? Those are the main questions that a developer of any level has to ask them, and that’s why in this blog post i’ll expose the way i make it in a react-native/redux app .

What is the best way to create a react-native application?
---

The first question i had in my mind when launching a project based on react-native. With which command i will created my app? **react-native init** or **react-native create-app** ?
So i want to compare them and see what the difference .

* **react-native init**:

Running this command will create an android and an ios folders. The existence of those two files provide you the ability to add modifications related to the native platform. And install all necessary components to start the development. 

* **react-native create-app**:

Running this command, will generate something entirely different from the first command . 
The created application will have an architecture based on the boilerplate create by Dan Abramov (the Redux creator), no android neither an ios folder but an [expo](https://expo.io/) folder.

=> So if you are new to the react-native use the second command you will not have to deal with the configuration. But you will use the ability of adding some modifications in the native platform and what embarrass me really is the “black-box” that the expo add it in the application. 

Within that you will certainly understand that i prefer using the first command because it’s more the native way to code in react-native .

What we are going to use as a library for the UI?
---

After a deep search, i found two library that are useful for the UI development the first one is [nativeBase](https://docs.nativebase.io/#Introduction) and the second is 
[react-native-elements](https://react-native-training.github.io/react-native-elements/). So it’s time to compare them and see what is the best of them not only on term of ui but also in term of flexibility and integration.

* **nativeBase**: As mentioned in their introduction, nativeBase “ is a free and open source UI component library for react-native to build native mobile apps for iOS and Android platforms.”

This library is rich of components you can find what you really need and more. But i was really impressed is his flexibility of use with [react-navigation](https://github.com/react-navigation/react-navigation)  and many other [library](https://docs.nativebase.io/docs/examples/Examples.html)

* **react-native-elements**: is cross platform UI toolkit. This library limit its components in only the needed ones so you can find what it’s the more used.

=> The choice of what you will use as a library depends in what is the dimension of the app . if you are going to create a simple application that don’t use many components i recommend you to use the second one. But if you are going to create an application full of components and that needs to be integrated with some other library like redux-form/ react-navigation … i think that nativeBase is more suitable to this.

What is the best architecture to put it in place when using redux ?
---

I think that's the most nerf point in all the process of starting the application. What is the most suitable architecture for my app? Is this the best way to do things?

So i will juste propose you in this section my way to see things. You have just nedd to ask yourself this question: **what is the dimension of my application?**. And based on your response you can do it with one of this ways:
* **small application**: You can use the simple way of putting in place a redux application as used in the redux documentation.

* **large application**: I propose to separte things as topics, using this way you will have a clean , re-usable and maintainable architecture.

=> So what i really recommand you in this point is to take a space before start to develop. And think twice what is the best way to do it because it influence in all the application.