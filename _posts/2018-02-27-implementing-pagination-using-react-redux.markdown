---
layout: post
title:  "Server Side Pagination, using React, Redux and Ant Design"
author: oumayma
date:   2018-02-27 11:20:00
categories: react
tags: react redux ant-design
---

![Server Side Pagination, using React, Redux and Ant Design](/assets/article_images/2018-02-27-implementing-pagination-using-react-redux/redux-react-antd-pagination.png)

Let's be straightforward: Fetching a quite long list of users (or other entity) from an API and displaying them in a list view is relatively not useful especially for large sets of data.

The main handicap of this idea would be an unlimited scroll down list which is definitely a bad design. Plus sinking the client with data.

Server side pagination seems useful to solve this matter. It transfers to the client a much smaller data to handle. So it provides the possibility to specify `the number of data sub-sets` to retrieve per page and for a given `page number`.

Although a substantial number of APIs do support serving data in this fashion, there are also a considerable number of them that don't.

In our blog post, we are exposing two different solutions :

- **Server side pagination** assuming an API that supports it.

- **Client side pagination** without pagination, that is full data transfer.

At EMIKETIC, we use [React](https://reactjs.org/) with the [Redux](https://redux.js.org/) architecture combined with [Ant Design](https://ant.design/). We will be showcasing this post with this stack along with a example loading users from a `Restful` API.

Let's dive in!
---

As the Redux mantra prescribes it, all the application state is stored as a single object in a store. First of all, let's define a sub-state we will call `User` ( refers to user component) in order to store rendered data and pagination states.

**User component substate**
``` javascript

const INDEX_PAGE_SIZE_DEFAULT = 50;
const INDEX_PAGE_SIZE_OPTIONS = [5, 10, 20, 30, 50, 100];

state = {
  data: null,
  meta: {
    page: 1,
    pageSize: INDEX_PAGE_SIZE_DEFAULT,
    pageSizeOptions: INDEX_PAGE_SIZE_OPTIONS,
    pageTotal: 1,
    total: 0,
  },
};
```

First Approach: Server-side pagination
---

This approach is used only when the API supports sending pagination parameters to receive a much smaller amount of data.
Using Redux, we have to define action types, action creators and thunks to make it all happen.

**Action Types and Action Creators to load users**

```javascript
// REQUEST LOADING USERS
const USER_INDEX_REQUEST = 'USER_INDEX_REQUEST';
const fetchIndexRequest = () => {
  return {
    type: USER_INDEX_REQUEST,
  };
}

// USERS RETREIVED WITH SUCCESS
const USER_INDEX_SUCCESS = 'USER_INDEX_SUCCESS';
const fetchIndexSuccess = (payload) => {
  return {
    type: USER_INDEX_SUCCESS,
    data: payload,
  };
}

// FAILED TO RETREIVE USERS
const USER_INDEX_FAILURE = 'USER_INDEX_FAILURE';
const fetchIndexFailure = () => {
  return {
    type: USER_INDEX_FAILURE,
  };
}

// THUNK ACTION CREATOR TO FETCH USERS
function $fetchIndex() {
  return (dispatch, getState) => {
    const { meta } = getState().User;

    dispatch(fetchIndexRequest());

    return fetch(
      `${endpoint}/users?${serializeQuery({
        per_page: meta.pageSize,
        page: meta.page - 1,
      })}`)
      .then((result) =>
        dispatch(fetchIndexSuccess({
          data: result.users,
          meta: {
            page: 1 + result.start / result.limit,
            pageSize: result.limit,
            pageTotal: Math.ceil(result.total / result.limit),
            total: result.total,
          },
        })))
      .catch((error) => dispatch(fetchIndexFailure(error)));
  };
}
```
with :

```javascript
function serializeQuery(query) {
  return Object.keys(query)
    .map((key) => `${encodeURIComponent(key)}=${encodeURIComponent(query[key])}`)
    .join('&');
}
```
**Action Type and Action Creators for pagination**

```javascript

// SINGLE ACTION TYPE TO CHANGE META DATA
const USER_INDEX_META = 'USER_INDEX_META';

// ACTIONS CREATORS
function $pageSize(pageSize = INDEX_PAGE_SIZE_DEFAULT) {
  if (pageSize < 1) {
    pageSize = 10;
  }

  if (pageSize > 100) {
    pageSize = 100;
  }

  return {
    type: USER_INDEX_META,
    meta: {
      pageSize,
      page: 1,
    },
  };
}

function $page(page = 1) {
  return (dispatch, getState) => {
    const { meta } = getState()[substate];

    if (page < 1) {
      page = 1;
    }

    if (page > meta.pageTotal) {
      page = meta.pageTotal - 1;
    }

    dispatch({
      type: USER_INDEX_META,
      meta: {
        page,
      },
    });
  };
}
```

**Reducer**
```javascript
function reducer(
  state = {
    data: null,
    meta: {
      page: 1,
      pageSize: INDEX_PAGE_SIZE_DEFAULT,
      pageSizeOptions: INDEX_PAGE_SIZE_OPTIONS,
      pageTotal: 1,
      total: 0,
    },
  },
  action,
) {
  switch (action.type) {
    case USER_INDEX_META:
      return {
        ...state,
        meta: {
          ...state.meta,
          ...action.meta,
        },
      };
    case USER_INDEX_SUCCESS:
      return {
        ...state,
        data: action.data,
        meta: {
          ...state.meta,
          ...action.meta,
        },
      };
    default:
      return state;
  }
}
```
**Wiring to the view**

We used all defined actions in a react component called `UserIndexView` connected to the store thanks to `react-redux` npm module.
In this component, we used the component [Table](https://ant.design/components/table/) from `Ant Design` which contains the configurable property [Pagination](https://ant.design/components/pagination/). The properties of `pagination` responsible of changing the pagination state are:

* **onShowSizeChange** : the action creator `$pageSize` is dispatched once the pageSize is changed.

* **onChange**: the action creator `$page` is dispatched when the page number is changed.

* **pageSizeOptions**: is an array that specifies the sizeChanger options. In our example we have `pageSizeOptions = [5, 10, 20, 30, 50, 100]`.


```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { Table, Avatar, Icon, Input, Button } from 'antd';

import { $pageSize, $page, $fetchIndex } from './state';

// provide shared state and actions as props
const withStore = connect(
  (state) => ({
    data: state.User.data,
    meta: state.User.meta,
  }),
  (dispatch) => ({
    dispatch,
  }),
);

// provides shared state and actions as props
const Connector = (C) => withStore(C);

class UserIndexView extends Component {
  componentWillMount() {
    this.props.dispatch($fetchIndex())));
  }
// PAGINATION OPTIONS
  paginationOptions = {
    showSizeChanger: true,
    showQuickJumper: true,
    onShowSizeChange: (_, pageSize) => {
      this.props.dispatch($pageSize(pageSize));
      this.props.dispatch($fetchIndex())));
    },
    onChange: (page) => {
      this.props.dispatch($page(page));
      this.props.dispatch($fetchIndex())));
    },
    pageSizeOptions: this.props.meta.pageSizeOptions,
    total: this.props.meta.total,
    showTotal: (total, range) => `${range[0]} to ${range[1]} of ${total}`,
  };

  render() {
    const pagination = {
      ...this.paginationOptions,
      total: this.props.meta.total,
      current: this.props.meta.page,
      pageSize: this.props.meta.pageSize,
    };

    return (
      <Table
        style={{ backgroundColor: 'white', flex: 1 }}
        dataSource={this.props.data}
        pagination={pagination}
      >
        <Table.Column
          title="Name"
          key="name"
          render={(record) => (<span>{record.name}</span>)}
        />
        <Table.Column
          title="Email"
          key="email"
          render={(record) => (<span>{record.email}</span>)}
        />
      </Table>
    );
  }
}
export default Connector(UserIndexView);
```


Second Approach:  Client-side pagination
---

In this part, we are going to mention just the trick that differentiates the `Client-side pagination` from `Server-side pagination`.
As the API does not supply the filtering parameters to send from the client and loads instead the whole data, we will need to load all the data and update pagination options based on our stored state and dispatch actions using Redux.

In the main component `UserIndexView` already defined, we definitely have no differences, just as we don't in all actions creators. There are only two small differences in `thunk action creator` and `reducer` which will be mentioned in the two pieces of code:

**Thunk action creator**
```javascript
...
return fetch(`${endpoint}/users`)
  .then((result) =>
    dispatch(fetchIndexSuccess({
      data: result.users,
      meta: {
        page: 1,
        pageSize: meta.pageSize,
        pageTotal: Math.ceil(result.total / meta.pageSize),
        total: result.total,
      },
    })))
...    
```

**Reducer**

```javascript
...
case USER_INDEX_SUCCESS:      
  return {
    ...state,
    data,
    meta: {
      ...state.meta,
      ...action.meta,
      pageTotal: Math.ceil(data.length / state.meta.pageSize),
      total: data.length,
    },
  };
...  
```

Conclusion
---

The purpose of this blog post is to highlight the importance of using pagination even if the restful API we are using does not provide it, and the important role that Redux plays in managing and organizing the whole mechanism.
