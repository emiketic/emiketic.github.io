---
layout: post
title:  "React DnD for complex drag and drop interfaces"
author: sabrine
date:   2017-03-29 10:46:00
categories: react
tags: react react-dnd es6 javascript
image: /assets/article_images/2016-05-04-create-your-own-download-manager/e.jpg
---

React DnD is a performant drag and drop library for React, that allows you to create drag and drop interfaces
keeping your components separated.

First let's take a look at React DnD concepts:

  React Dnd implemented the **HTML5 drag and drop API**  and Data is the only source of truth for it. This library supports applying different functionalities to different sets of data.

  In order to drag and drop, there are a few functions and properties to be declared:

  * **Item**: A plain JavaScript object describing what has been dragged.
  * **Type**: A type is a string or Symbole that should be unique throughout the whole application.
  * **Monitors**: Allows you to specify your drag and drop behavior.
  * **Connectors**: Used to assign roles to the DOM Node such as `DragSource`, `DragPreview` and `DragTarget`.

This post is an initial step towards getting how everything fits together.

The best way to illustrate this is by example. We're going to drag boxes and drop them into the bin.


Installation
---
We need to install 2 packages:

**npm install react-dnd**

**npm install react-dnd-htlm5-backend**

Setting up our app
---
We're going to use the the HTML5 backend in our app, so we should add `DragDropContext` in the top level component in our app.

``` javascript
import React, { Component } from 'react';

import { DragDropContext } from 'react-dnd';
import HTML5Backend from 'react-dnd-html5-backend';

import './App.css';


 export class App extends Component {
   render() {
     return (
       <Grid className="App-container" fluid>
        {
          // write your code here
        }
       </Grid>
     );
   }
 }
export default DragDropContext(HTML5Backend)(App);
```
This example will have two major components:

* **Box**: component can be dragged and dropped.

* **Bin**: component where Boxes will be dropped.

Let’s write a constant for the draggable item's type.

``` javascript
const Types = {
  BOX: 'box'
};
```

Step 1: Make component draggable
---
First, we define the Box component.

``` javascript
import React, { Component, PropTypes } from 'react';

export class Box extends Component {
  render() {
    const { color } = this.props;
    const style = {
      backgroundColor: color,
      height: 10vh;
    };
    return (
      <Col md={3}>
       <div style={style}>
         {color}  
       </div>
      </Col>
    );
  }
}

Box.propTypes = {
  color: PropTypes.string.isRequired,
};
```
Then, we wrap the Box component with ```DragSource``` to make it draggable.

``` javascript
import { DragSource } from 'react-dnd';
/**
  code
*/
const boxSource = {
  beginDrag(props) {
    return {
      color: props.color,
    };
  },
};
export class Box extends Component {
  render() {
    /**
      code
    */
    return connectDragSource(
      <Col md={3}>
       <div style={style}>
         {color}  
       </div>
      </Col>
    );
  }
}
Box.propTypes = {
  color: PropTypes.string.isRequired,
  connectDragSource: PropTypes.func.isRequired,
};
export default DragSource(Types.BOX, boxSource, connect => ({
  connectDragSource: connect.dragSource(),
}))(Box);

```
These are functionalities  that help you handle your component and identify its current state:

 **beginDrag** aims at returning a plain JavaScript object describing the data being dragged.

 **connectDragSource** tells the react-dnd that the DOM Node is a valid dragSource.

Step 2: Make a drop target
---

Now our component is ready to be dragged. Let's Move to define the component we can drop Boxes into.

We’re keeping things simple and just using ```DropTarget``` to make Bin as a drop target.

``` javascript
import React, { Component, PropTypes } from 'react';
import { DropTarget } from 'react-dnd';

const boxTarget = {
  drop(targetProps, monitor) {
    return {
      box: monitor.getItem().color
    };
  },
};

export class Bin extends Component {
  render() {
    const { isOver, connectDropTarget } = this.props;
    const style = {
      backgroundColor: isOver ? 'grey': 'white';
    };
    return connectDropTarget(
      <Col>
         <div style={style}>
            Drag your box here
         </div>
      </Col>,
    );
  }
}
export default DropTarget(Types.BOX, boxTarget, (connect, monitor) => ({
  connectDropTarget: connect.dropTarget(),
  isOver: monitor.isOver(),
}))(Bin);

```
**drop()** is a function Called when a compatible item is dropped on the Bin.

**connectDropTarget()** marks the react element as the droppable node.

You can ask the monitor about the current drop state by calling monitor methods such us **monitor.isOver()**,
**monitor.canDrop()**, **monitor.getItemType()**...
The best part  about this, is that such information allow you to manipulate the component style as illustrated above.
the same options are applicable concerning```DragSource``` and results are guaranteed.

Finally, we import Boxes and Bin components in the top level component in the app.

Conclusion
---
This post carried out the objective of familiarizing developers with react DnD. It can surely help you build, handle and manipulate complicated drag and drop interfaces with a fairly small amount of effort.
It is highly advisable you walk the extra mile and discover what this rich library React DnD has to offer yet.  
