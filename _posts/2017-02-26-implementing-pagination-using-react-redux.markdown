---
layout: post
title:  "Pagination with React & Redux using Ant Design"
author: oumayma
date:   2017-02-26 11:20:00
categories: react
tags: react redux ant-design
image: /assets/article_images/2017-02-26-implementing-pagination-using-react-redux/antd-react-redux.jpg
---

![Pagination with React & Redux using Ant Design](/assets/article_images/2017-02-26-implementing-pagination-using-react-redux/antd-react-redux.jpg)

Let's give the fact that fetching a quite long list of users from an API and displaying them in a list view is relatively not useful especially for a large amount of data.

The main handicap of this idea would be an unlimited scroll down list which is definitely a bad design.

Using [React](https://reactjs.org/) combined with [Redux](https://redux.js.org/) allow us to fully customize our list of users with an attached pagination component of [Ant Design](https://ant.design/) (React UI library rich with height quality components for interactive user interfaces).

Briefly, in our blog post, we are counting on explaining our adopted approach on how to create a custom page to display a list of users with an attached pagination using Ant Design and the power of Redux.

Let's dive in !
---

Step 1: Designing the Substate Shape
---
In Redux, all the application state is stored as a single object. So in this part, we write the pagination state in `User` substate.

Step 2: Defining Action Types & Action creators
---
* **TO FETCH USERS**

* **TO UPDATE PAGINATION**

Step 3: Define reducer
---

Step 4: Dispatching actions
---
* **TO LIST ALL USERS**

* **TO UPDATE PAGINATION**
