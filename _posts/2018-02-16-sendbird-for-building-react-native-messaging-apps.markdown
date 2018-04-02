---
layout: post
title:  "SendBird for building React Native messaging apps"
author: sabrine
date:   2018-02-16 10:46:00
categories: react
tags: react react-native es6 javascript sendBird
image: /assets/article_images/2016-05-04-create-your-own-download-manager/e.jpg
---
![SendBird for building React Native messaging apps](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/react-native-sendbird.png)


Sending messages with apps has become a worldwide top use on the mobile device and messaging applications have become more ubiquitous.

In this blog post, we will share with you our experience with messaging applications.

Messaging is no longer a medium nor just a feature, it's a realm of large tech companies. It does beg the question, Shall I build my own feature, in-house development or integrate a 3rd party solution?

Generally, teams prefer to use off-the-shelf messaging SDK solution. It's the best way to reduce time-consuming, increase performance and focus other core content. This is why we used SendBird, It's really a scalable, powerful and fully-documented chat API. It helps you to quickly add a real-time chat to any app. 

This API is rich in features. You can find what you really need and more. Let's discover this amazing API and Follow the initial step towards getting how everything fits together.

We'll build a simple messaging app using React Native with SendBird's JavaScript SDK.

First, you'll need to create a new SendBird application in the dashboard.

![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/sendbird-dashboard.png)

Installation
---
In your React Native project folder, you need to run

```npm install sendbird --save```

Connect To SendBird
---
SendBird requires a UserID to connect or to create a new user account. The UserID can be any unique id, such as an email address or a database UID. 

We do not need a separate registration process. SendBird creates for you a new account if you have not been registered yet, all you need is to specify your unique ID.

The first thing we’re going to need is a screen where the user can enter a userID and a nickname to appear as.

let's start creating a Sign in screen:

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

In this tutorial, we will use email as UserID and nickname to be displayed beside each message we send.

First, we have to import `sendbird` to our component:
 ``` javascript
import SendBird from 'sendbird';
```

Next, we should initialize SendBird using the APP_ID assigned to our SendBird application.
 ``` javascript
const sb = new SendBird({ appId: APP_ID });
```

Let’s write our first function using the SendBird functions `connect()` and `updateCurrentUserInfo()`:

  * **connect()**: it's a connection request and also used to create a new user account if the user has not been registered yet.

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
                  self.props.navigation.navigate('Home');
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

We are registered now! let's go to the next step.

Join channel
---
All regisetred users can create or join channel which provides methods to easily send, fetch, and receive messages in real-time.

SendBird provides you with different types of channel: 

  * **Open channel**: anyone may join and participate in the chat. 
  * **Group channel**: the user can join the chat only through an invitation. it could be a private channel among multiple users or between tow users. Also, You can send files and custom Data using files transfers.

In our app we will use group channel.


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
Here we listed some users which are already registered.

![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/friendlist.jpg)


Let’s define our function `joinChannel()` in order to create a channel with two members (1-to-1 messaging).

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
* **createChannelWithUserIds()**: is a function Called to create a new channel. You should pass two user IDs or your opponent ID. It aims at returning a plain JavaScript object describing the channel.

The best part about this is that you can pass information by passing additional arguments such as name, coverUrl,
data, customType...


See [SendBird Docs](https://docs.sendbird.com/javascript#group_channel_one_to_one_chat) for more informations.

Let's chat!
---
Now for the main dish, we're going to create our chat room.

The first step, we need to create a chat UI component.

In general, we can create our own chat UI our selves using simply React Native library. Also, there are a quite large amount of libraries dedicated to solving this issue themselves. For example here, we used [react-native-gifted-chat](https://github.com/FaridSafi/react-native-gifted-chat) for our project.
Nevertheless, using this library would be more useful, it might also cause some recoverable problems UI and so on ...

```javascript

import React, { Component } from 'react';
import { View }  from  'react-native';
import { Container } from 'native-base';
import { GiftedChat, Bubble, LoadEarlier } from 'react-native-gifted-chat';
import SendBird from 'sendbird';

import styles from './chatScreen.css';

type Props = {};
export default class ChatScreen extends Component<Props> {
  state= {
     messages: [],
  }
  render() {
    const bubble = (props) => (
      <Bubble
        {...props}
        wrapperStyle={styles.wrapperStyles}
      />
    );
    const composer = (props) => (
      <Composer
        {...props}
        multiline
        textInputStyle={styles.composerStyles}
      />
    );
    const loadEarlier = ((props) => {
      if (_.isEmpty(this.state.messages)) {
        return (
          <LoadEarlier
            {...props}
            label="Start a conversation"
          />
        );
      }
      return null;
    });
    return (
      <Container>
          <GiftedChat
            messages={this.state.messages}
            renderBubble={bubble}
            loadEarlier
            renderLoadEarlier={loadEarlier}
            isAnimated
            keyboardShouldPersistTaps="never"
            onSend={(messages) => this.onSend(messages)}
            user={{
              _id: currentUser.userId,
              avatar: currentUser.avatar,
            }}
            showUserAvatar
          />
      </Container>
    );
  }
}
```
Our chat UI component is ready now!

![Image of SendBird Dashboard](/assets/article_images/2018-02-16-sendbird-for-building-react-native-messaging-apps/chatBox.png)

At this stage, we are ready to sent messages and load created conversations.

Let's send our first message!

SendBird SDK provides you with methods to send messages in an entered channel such as `sendUserMessage()` and `sendFileMessage()`.

* **sendUserMessage()**: used to send text messages.
* **sendFileMessage()**: used to send binary messages.

```javascript 
onSend(messages = []) {
  const handle = this;
  const sb  = SendBird.getInstance();
  const channel =  this.props.state.params.channel;
  this.setState((previousState) => {
    channel.sendUserMessage(messages[0].text, (response, error) => {
      if (!error) {
        // handle.getChannelMetaData(channel);
      }
    });
    return { messages: GiftedChat.append(previousState.messages, messages) };
  });
}
```
You could load conversation by creating a query using `createPreviousMessageListQuery()`, this instance loads the most recent n messages.

**Note:**  n is a fixed number used to query a set number of previous messages.

```javascript 

import React, { Component } from 'react';

import { Container } from 'native-base';

import SendBird from 'sendbird';

type Props = {};

export default class ChatScreen extends Component<Props> {
  state= {
     messages: [],
  }
  componentDidMount(){
    const { channel }  = this.props.navigation.state.params;
    this.getChannelMetaData(channel);
  }
  getChannelMetaData(channel) {
    if (channel) {
      const self = this;
      const messagesQuery = channel.createPreviousMessageListQuery();
    
      messagesQuery.load(50, true, (messages, error) => {
        if (error) {
          console.error(error);
        }
        this.setState({ 
          messages,
        });
      });
    }
    return;
  }
  render() {
    return (
      //some code
    );
  }
}

```

By providing its own Event Handlers, the SendBird SDK allows you to track events occuring within channels or devices such as typing, read receipts and more. It is highly advisable you walk the extra mile and discover what this rich SDK has to offer yet.

Learn more about [SendBird Event Handler](https://docs.sendbird.com/javascript#event_handler).

 
Conclusion
---
This post guides you through the preliminary steps of setting up SendBird in your own React Native app.
I hope you found this article useful to kick off learning how to add real-time chat to any React Native app with speed and efficiency.

Finally, if you have a super robust SDK, you don't need to reinvent the wheel! 

