---
layout: post
title:  "SendBird for building React Native messaging apps"
author: sabrine
date:   2018-02-16 10:46:00
categories: react
tags: react react-native es6 javascript sendBird
image: /assets/article_images/2016-05-04-create-your-own-download-manager/e.jpg
---
Sending messages with apps has become a worldwide top use on mobile device and  messaging applications have became more ubiquitous.

In this blog post, we will share with you our experience with messaging applications.

Messaging is no longer a medium nor just a feature, it's a realm of large tech companies. It does beg the question, Should I build my own feature, in-house development or integrate a 3rd party solution ?

Generally, teams prefer to use off-the-shelf messaging SDK solution. It's the best way to reduce time consuming, increase perfermance and focus other core content. This is why we used SendBird, It's really a scalable, powerful and fully-documented chat API. It helps you to quickly add a real-time chat to any app. This API is rich in features. You can find what you really need and more.


let's discover this amazing API and Follow the initial step towards getting how everything fits together.

We'll build a simple messaging app using React Native with SendBird's JavaScript SDK.

First you'll need to create a new SendBird application in the dashboard.

![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/sendbird-dashboard.png)

Installation
---
In your  react-native project folder, you need to run

```npm install sendbird --save```

Connect To SendBird
---
SendBird requires a UserID to connect or to create a new user account. The UserID can be any unique id, such as an email address or a database UID. 

We do not need a separate registration process. SendBird creates for you a new account if you have not been registered  yet, all you need is to specify your unique ID.

The first thing we’re going to need is a screen where the user can enter a userID and a nickname to appear as.

let's start creating a Signin screen

``` javascript
import React, { Component } from 'react';

import { Container, Content, Item, Input, Form, Button, Label, Text } from 'native-base';

type Props = {}

export default class LoginScreen extends Component<Props> {
  state= {
   email: '',
   nickname:'',
  }
  render() {
    return (
      <Container>
        <Content>
          <Form style={{ padding: 16 }}>
            <Input 
              onChangeText={(email) => this.setState({ email })}
              placeholder="email"
              style={{ backgroundColor: '#fff', marginVertical: 16 }}/>
            <Input 
              placeholder="nickname"
              onChangeText={(nickname) => this.setState({ nickname })}
              style={{ backgroundColor: '#fff', marginVertical: 16 }}/>
            <Button 
              full
              onPress={this._connect(this.state.email, this.state.nickname)}
            >
              <Text>Connect</Text>
            </Button>
            </Form>
        </Content>
      </Container>
    );
  }
}
```

In this tutorial we will use email as UserID and nickname to be displayed beside each message we send.

First, we have to import `sendbird` to our component:
 ``` javascript
import SendBird from 'sendbird';
```

Next, we should initialize SendBird using the APP_ID assigned to our SendBird application.
 ``` javascript
const sb = new SendBird({ appId: APP_ID });
```

Let’s write our first function using the SendBird functions `connect()` and `updateCurrentUserInfo()`:

  * **connect()**: it's a connection request and also used to create a new user account if the user has not been registred yet.

  * **updateCurrentUserInfo()**:  is a function Called to update user's nickname and profile image.

 ``` javascript
import SendBird from 'sendbird';

const sb = new SendBird({ appId: APP_ID });
 // some code
export default class LoginScreen extends Component<Props> {
   // some code
  _connect(email, nickname) {
    const self =  this;
    sb.connect(userId, (user, error) => {
        if (error) {
            console.log('SendBird Login Failed.');
        } else {
            sb.updateCurrentUserInfo(nickname, null, (user, error) => {
                if (error) {
                    console.log('Update user Failed!');
                } else {
                  self.props.navigation.navigate('HomeScreen');
                }
            })
        }
    });
  }
  render() {
    return(
      // some code
    );
  }
}
```
`connect()` aims at returning a plain JavaScript object describing the user profile.


![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/sendbird-connection.png)

We are registred now! let's go to the next step.

Join channel
---
All registred users can create or join channel which provides methods to easily send, fetch, and receive messages in real-time.

Sendbird provides you with different types of channel: 

  * **Open channel**: anyone may join and participate in chat. 
  * **Group channel**: user can join the chat only through an invitation. it could be a private channel among multiple users or between tow users. Also, You can send files and costum Data using files transfers.

In our app we will use group channel.

Here we listed some users which are already registred.

![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/friendlist.png)

``` javascript

import React, { Component } from 'react';

import { Container, Content, Text, Right,Body, Header, ListItem, List, Left, Thumbnail, Button } from 'native-base';

type Props = {}
export default class HomeScreen extends Component<Props> {
  render() {
    return (
      <Container>
        <Content>
            <List  style={{padding: 16}}>
               {users.map(user => (
                  <ListItem avatar key={user.id}  style={{padding: 16}}>
                    <Left>
                      <Thumbnail source={{ uri: user.profileUrl }} />
                    </Left>
                    <Body>
                      <Text>{user.name}</Text>
                      <Text note>Online</Text>
                    </Body>
                    <Button
                      transparent
                      onPress={() => this._joinChannel(user.userId)}
                    ><Text>chat</Text></Button>
                </ListItem>
                ))}
            </List>
        </Content>
      </Container>
    );
  }
}

```
let's  write our function _.joinChannel()

``` javascript
 _joinChannel(receiverId) {
    const sb = SendBird.getInstance();
    const self = this;
    sb.GroupChannel.createChannelWithUserIds([receiverId],
      true, (createdChannel, error) => {
        if (error) {
          console.error(error);
        } else {
          self.props.navigation.navigate('Chat', { channel: createdChannel });
        }
    });
  }

```
```javascript 

import React, { Component } from 'react';

import { Container } from 'native-base';

import SendBird from 'sendbird';

type Props = {};

export default class ChatScreen extends Component<Props> {
  state= {
     messageList: [],
  }
  componentDidMount(){
    const { channel }  = this.props.navigation.state.params;
    this.getChannelMetaData(channel);
  }
  getChannelMetaData(channel) {
    if (channel) {
      const self = this;
      const messageListQuery = channel.createPreviousMessageListQuery();
      messageListQuery.load(30, true, (messageList, error) => {
        if (error) {
          console.error(error);
        }
        console.log(messageList)
        this.setState({ 
          messageList,
        });
      });
    }
    return;
  }
  render() {
    return (
      <Container>
         
      </Container>
    );
  }
}

```

By providing its own Event Handlers, the SendBird SDK allows you to track events occuring within channels or devices such as typing , read receipts and more.



Conclusion
---
  