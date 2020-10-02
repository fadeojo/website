---
title: Reusable react state
template: post
slug: Reuseable-react-state
draft: false
featured: true
date: '2019-01-04T15:00:00.000Z'
description: Reuse UI application state to simplify workflows
category: Frontend
cover: ./reusable-react-state-cover.png
tags:
  - React
  - Javascript
  - Frontend
---
In January of 2018 my team was exploring the possibility of taking advantage of using javascript to target web, mobile and desktop. We already had experience building react applications and wanted to reuse as much of the previous code as possible to build desktop and mobile applications using javascript.

I remember a meeting where my manager and director shared their vision of what this new world should look like and the theme was “Code Reuse”. How can we reuse as much as possible? I was tasked with researching possible solutions and estimating the cost to the engineering organization.

I like to make diagrams of my thoughts so I made the following;
![Diagram of my thoughts (this is a living document)](/reusable-react-state-cover.png)

## Code Reuse

After days of prototyping, I realized that the “write once deploy to many platforms” approach was not efficient or effective and left me even less confident about the end result of my research. Some technologies worth mentioning are;

-   React-native
-   React-native-web
-   ReactXP

I started talking to friends in the javascript community to try to learn from what others in my shoes have learned and I also didn’t want to repeat mistakes that could be avoided. The javascript community was tremendously helpful. I posted messages on slack channels, discord channels and more. I bugged as many online and offline friends that would give me their audience. In the end, it was clear that there were two main areas that were worth exploring;

-   UI components.
-   Application logic.

I am going to focus on the **application** logic part of my research from this point on.

## Application logic

Now I know that application logic is a broad term but for the purpose of this post, it represents logic that can be stored in a separate file, to be imported by other parts of our application that can take advantage of it. In our case this would be the bulk of the logic that resides in our state management library of choice. My test environment was made up of a react web app (Web), react electron app (Desktop) and a react native application.

## Redux

We used redux in the past so I tried to create a local stand alone package that held the redux store, actions and reducers. A simple example can be seen in the code below.

```js
Import { createStore } from 'redux'
/**
 * This is a reducer, a pure function with (state, action) => state signature.
 * It describes how an action transforms the state into the next state.
 *
 * The shape of the state is up to you: it can be a primitive, an array, an object,
 * or even an Immutable.js data structure. The only important part is that you should
 * not mutate the state object, but return a new object if the state changes.
 *
 * In this example, we use a `switch` statement and strings, but you can use a helper that
 * follows a different convention (such as function maps) if it makes sense for your
 * project.
 */
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter)
// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However it can also be handy to persist the current state in the localStorage.
store.subscribe(() => console.log(store.getState()))
// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

I found it interesting that our entire application logic could be decoupled from the UI. After all, we have an initial state of ‘0’ and can dispatch an action to increment our state to ‘1’ and so on. I got all excited and added more logic. I could handle synchronous and asynchronous (api calls) without coupling the global state to any ui. I used [Quokka](https://quokkajs.com/) to evaluate state in my text editor.

The example code above is pure javascript. It can be used in any javascript environment provided the **redux** package is available. A command line, server-side or client-side app looking to implement a counter could import the above code from a package and use it.

## Pain

When the use case was simple redux was a delight but that started to change as my application grew. I was struggling to keep up with all the boilerplate code and I didn’t know any better so I just powered through it. There had to be a better way to do this I thought as I piled middleware on middleware until…

## mobx-state-tree (MST)

[Gant Laborde](https://medium.com/u/6ca0fe37eac1) mentioned [mobx-state-tree](https://github.com/mobxjs/mobx-state-tree) (MST) while we were discussing state management for React applications. Gant Laborde mentioned that [infinitered](https://infinite.red/) was evaluating it for the next iteration of the Ignite boilerplate codenamed “Bowser”. [MST](https://github.com/mobxjs/mobx-state-tree) was built on [mobx](https://github.com/mobxjs/mobx) and I had used mobx in the past so I decided to try it out.

As defined on the github repository, MST is an Opinionated, transactional, MobX powered state container combining the best features of the immutable and mutable world for an optimal DX. There is a free egghead tutorial on it, all you need is to provide an email so that an authenticated link of the tutorial can be sent to you. it’s the best way to get started with MST.

The code below is an example of a counter implemented using MST.

```js
import { types } from "mobx-state-tree"
// Create store definition
const CounterStore = types.model("CounterStore", {
    counter: types.number
}).actions(self => {
    return {
        increment() {
            self.counter ++
        },
        decrement() {
            self.counter --
        }
    }
})
// initialize the store
const counter = CounterStore.create({
    counter: 0
})
counter.increment()
console.log(counter.counter) // 1
counter.increment()
console.log(counter.counter) // 2
counter.decrement()
console.log(counter.counter) // 1
```

The example code above is also pure javascript. It can be used in any javascript environment provided the **MST** package is available. A command line, server-side or client-side app looking to implement a counter could import the above code from an npm package and use it.

Handling large amounts of logic was also very simple as I kept each model in it’s own file. The amount of boilerplate was not as much as redux and features of mobx and MST like observable, actions, views, volatile state, tree structure, patches and flow made it easy to model complex logic.

we used lerna to create a mono repo that held the ui repository and our app logic repository. This ensured we could import the application state as though it was in the same repository as our ui.

There were cases where we had a very large initial state and could hydrate our state in a nodejs Lambda and send the reconciled state to the UI so we did very little computation on the browser. This was only possible because our state was a separate package that could be imported into any javascript environment.

![Diagram of my thoughts (this is a living document)](/mst-vs-react.png)

In the end we were able to build a good part of our app logic and test it without a UI. The exercise has changed how I think of application state. Our UX team was busy on another project and this method bought the UX team time to catch up with us. I believe this method would also be great for prototyping. Imagine being able to connect the application logic package with a tool like [FramerX](https://framer.com/) or [Invision Studio](https://www.invisionapp.com/studio). The possibilities ………….

In the end, we chose to share the application logic across all platforms, build the UI for each platform and that was enough!
