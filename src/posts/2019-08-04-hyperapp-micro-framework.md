---
title: Hyperapp, the frontend web framework your users might appreciate
date: '2019-08-04'
tags:
---

<div class="message">
Hyperapp is a framework for building web interfaces and ideal for when you don't want a full-blow framework (and perhaps for when you do).
</div>

**[UPDATE 2019/08/07: Added example]**

**[UPDATE 2019/09/06: Added link to Cypress explorer]**

Right now is a "fascinating" time to be a frontend web developer. There's plenty of interesting tools and frameworks to help ease the developer experience while providing rich user experiences. The downside can be the steep developer learning curve with complex configurations.

But your users may not always be happy with their experience of your development stack choice. The main popular frameworks, React, Vue and Angular, make heavy use of JavaScript running in the browser. Large JavaScript payloads need to be transmitted, loaded and parsed by the browser before they even start to do anything the user can see or interact with. Any errors are usually fatal, unlike errors that occur in processing declarative HTML and CSS.

This can frustrate users who experience delays, broken we sites and browser crashes. Worse, this is most likely to happen if they have old or low specification hardware, old web browsers or poor connectivity. Unless the developers are lucky enough to be targeting a controlled homogeneous environment, like an intranet, then at least some of the users are very likely to bfe in this unhappy group. They are also unlikely to remain users for long.

More recent developer tooling address some of these issues through techniques that preload some HTML or reduce the size of Javascript. For example, "server side rendering", "code splitting" and "tree shaking" are common practice. One recent framework, Svelte, reduces runtime size by doing as much work as possible at build time.

However, there is another developer approach to avoiding user frustration. That is to not have a framework at all. Or else, use one with a very small runtime. Remember, frameworks are usually designed to cover as wide a range of requirements as possible, many of which may be irrelevent to the current project. They also often address the problems experienced by large teams working on very large websites. Thus, they may not be the best solution for you current project, even if you already know a specific framework really well.

Often a project requirements fir interaction are mostly met by browser links and forms, or perhaps a custom control or two. Sometimes only part of the site needs to be dynamically updated rather then creating the entire page with JavaScript. In these cases a full framework can be overhead and it's feasible to just code to the DOM. That's the approach I took with a recent project [YouTube Current Time](https://github.com/music-practice-tools/youtube-current-time/blob/master/ytct-es6.js) that shows the playing time of a video while you scroll around a page. Using a framework would have been overkill and introduce more dependencies and complications than necessary.

With this no-framework approach it's usual to develop architectural patterns and structures that are effectively a custom framework that closely fit the project requirements, and no more. In fact, it is common to develop a minimal reusable starter framework that matches developer preferences and best practices. And this is where [hyperapp](https://github.com/jorgebucaran/hyperapp) is useful as a foundation. Especially when a functional programming approach based on the [elm architecture](https://guide.elm-lang.org/architecture/) is preferred.

Version 2 of the tiny Hyperapp framework has almost landed; it's on master in github but the npm package has not yet been published. While the docs are not quite available yet the [source code](https://github.com/jorgebucaran/hyperapp) is small and relatively easy to digest. Of the 500 odd lines of source code, the first 400 are taken up with the [Virtual DOM](https://reactjs.org/docs/faq-internals.html) implementation and can be ignored if you want to concentrate on the framework proper. There is also a set of official [examples](https://github.com/jorgebucaran/hyperapp/blob/master/docs/examples.md), plus [an implementation](https://github.com/zaceno/sevenguis-hyperapp) of [the 7GUIs](https://eugenkiss.github.io/7guis/) sample apps.

Here's an overview of the few elements that make up Hyperapp. They are mostly just functions as you'd expect in a functional programming framework. Here is a simple counter app example example app, also show at the end of this post.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script type="module">
      import { h, app } from "https://unpkg.com/hyperapp"

      app({
        init: 0,
        view: state =>
          h("div", {}, [
            h("h1", {}, state),
            h("button", { onClick: state => state - 1 }, "-"),
            h("button", { onClick: state => state + 1 }, "+")
          ]),
        node: document.getElementById("app")
      })
    </script>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

## App

The entry function that mounts and manages a DOM tree in the given DOM node. Is is passed: initial state, view and optional subscriptions and renders the view passing the state. User and other events are passed to Actions which may update the state or invoke an effect. Changes in state cause the view to be rerendered. It is possible to mount multiple apps if required.

## State

There is a single state tree for the entire app and it is only ever mutated in Actions, never directly.

## VDOM

Hyperapp includes its own Virtual DOM. As usual, this allows the view to describe the entire DOM tree using lightweight virtual DOM nodes. The VDOM ensure only the virtual nodes that have changed are then updated in the actual DOM.

## View

Maps the State to DOM indirectly by using the VDOM `h()` function to build a virtual DOM tree. Abstractions can be used instead of `h()`, including `hyperapp/html` (or equivalent) helper functions for each HTML element, the `JSX` build time parsed HTML-like syntax or `hyperx` ES6 template strings parsed at run time.

## Tuples

Hyperapp uses 2-tuples consisting of a function and a parameter. As JavaScript doesn't have tuples, arrays of 2 elements are used.

## Actions

Actions are functions that are invoked by the framework and passed the State and an optional parameter. They can return a new State or else invoke an Effect and return it's resulting tuple. They are declared to the framework either as DOM node event handlers in the View or in Subscriptions passed to the App. They abstract DOM event handlers with the framework adding and removing handlers and subscriptions as required.

References to an Action can be declared as the function itself (name or inline) or a Tuple containing the function and an argument.

## Effects (fx)

Effects are functions that perform side effects for the App. They can be thought of as "drivers" that perform all the external work such as calling a RESTful API endpoint, giving focus to an element or saving data to local storage. This way, Actions remain pure functions which simplifies testing and reasoning about logic.

Effects are invoked from Actions when they return a Tuple of the effect to perform as a function and an argument. Effects can invoke an Action asynchronously later.

The module [hyperapp-fx](https://github.com/okwolf/hyperapp-fx/blob/HAv2/api.md) contains many useful Effects.

## Subscriptions

Subscriptions alow Actions to be called in response to external events that are not tied to a DOM node in the view. For example, global DOM events or even side effects such as timers. A Subscription is a Tuple of a function to set up the subscription and a props object passed as a parameter to the function. The setup function is also passed a dispatch function that it uses to 'send' the event to hypeapp. In addition, the setup function returns the function that hyperapp can call to teardown the subscription when it is no longer needed.

The array of Subscriptions is passed to the App by a function which is called whenever the State changes. This function is passed the State and returns an array of subscriptions. If subscriptions appear in the array, they are setup and if they are removed they are torn down. Thus, Subscriptions can vary depending on the State.

This may seem a little complex so see the [Clock example](https://github.com/jorgebucaran/hyperapp/blob/master/docs/examples.md#simple-clock) which uses [hyperapp/time](https://github.com/SteveALee/hyperapp/blob/master/lib/time/src/index.js) 'interval' effect.


## Asyncronicity

As Actions remain pure they cannot use `promises` or `async` but these features can always be used in Effects. Effects and Subscriptions effectively use callbacks for handing async events back to the framework. Any logic that manages timing or ordering of events is thus best handled in Effects, though you can modify subscriptions based on State.

## Conclusion

And that's it. Hyperapp's few concepts and the tiny runtime make it a flexible framework, suitable for small to medium applications. The universal use of pure functions with side effects separated out also makes unit, business logic and integration testing managable. The use of Mocks and Stubs are confined to the Effects, making life much simpler. Finally, hyperapp provides a minimal foundation for building on further abstractions and patterns that perfectly model the current project domain.

Try the [examples](https://github.com/jorgebucaran/hyperapp/blob/master/docs/examples.md) and have a play. You only need an editor, a browser and possibly a local server (eg `npx serve`). Just like the "good old days" of web development. Try this...

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script type="module">
      import { h, app } from "https://unpkg.com/hyperapp"

      app({
        init: 0,
        view: state =>
          h("div", {}, [
            h("h1", {}, state),
            h("button", { onClick: state => state - 1 }, "-"),
            h("button", { onClick: state => state + 1 }, "+")
          ]),
        node: document.getElementById("app")
      })
    </script>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

## Exploring with Cypress

The [cypress](https://www.cypress.io/) test platform makes an ideal tool for exploring code that runs in a browser. This [hyperapp explorer](https://github.com/SteveALee/hyperapp2-explore) lets you play with the code and expands on the API description here.
