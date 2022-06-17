---
title: Is svelte.js the user-friendly developer framework?
date: '2019-12-19'
tags:
---

<div class="message">
Svelte provides an excellent Developer Experience (DX) whilst supporting a slick User Experience (UX), even on low powered devices. It does this through performing as much work as possible at build-time resulting in tight minimal code that when deployed directly interacts with the web platform in the browser.
</div>

In my [last post on hyperapp](/2019/08/04/hyperapp-micro-framework/) I explained my interest in terms of avoiding user frustration with poor website performance, often due to heavy reliance on a lot of javascript. Hyperapp is a tiny payload and is reasonably fast so can deliver a good UX on many older devices and even with poor connectivity as is often experienced by mobile users.

In this post we look at Svelte which also provides excellent UX performance but it does this by eschewing the currently popular dev friendly runtime JavaScript VDOM techniques. Such reliance on Javascript for all rendering may negatively impact the UX on all but the best devices (which developers alone often have) but not there users. Svelte does this by using a compiler providing build-time generation of 'laser' precise' UI code that acts directly with the browser DOM and BOM APIs.

Additional improvements in speed and reliability can be gained through svelte's use of templated component markup. This transparently supports Progressive Enhancement (PE) of base coding with HTML with CSS which the browser processes with generally "fail safe" results. The  developer enhances this base with additional features, usually with more fragile JavaSript behaviours according to if they are supported by the user's device.

Whatever framework is being used, developers require a good understanding of the web platform in order to create reliable and inclusive UX that is fast. Svelte's compiled template design providing JavaScript-in-HTML makes this a relatively natural approach to follow. This compares to the more common HTML-in-JavaScript technique (with or without JSX) which might even be said to encourage developing deep code abstractions that distance away from the platform. These might be good from a computer science viewpoint but are not necessarily a good match with the highly heterogeneous web platform. HTML and CSS have been designed with almost painful care to be as inclusive as possible and this has to be reproduced in any roll-your own code if it is not to be lost to users detriment.

In other words, svelte supports modern component UI design patterns but with a compiler reducing runtime the loading / processing times seen by users. Its also also allows developers to take direct advantage of the web platform's strengths without requiring  extra framework abstractions (such as React hooks) to be learnt. Of course, abstractions are possible but they can be added at the discretion of the developer, hopefully after a good cost/benefit tradeoff analysis from the user perspective. Svelte does have a preprocessor hook to allow extensions such as CSS tooling but I sense that keeping close to the platform is encouraged.

Svelte does provide a few abstractions of it's own but these are minimal and extremely useful. For example Stores are an minimal approach to cross component data that. They are observable. In fact, stores very closely follow the core APIs of Functional Reactive Programming (FRP) libraries like RxJS allowing trial integration with these. Another set of abstractions make animations and transitions easier to programme. These support both Javascript and CSS approaches too, providing flexibility.

A couple of other abstractions are really overloads of existing JavaScript syntax. While this may a first seem a bad idea in practice they work REALLy well without causing any confusion and allows the source code in svelte component files to be valid JavaScript, HTML and CSS. One such syntax overload is the use of 'export' to declare properties exposed by components to parents. The others add reactivity in addition to the observable Stores. All assignment statements cause a redraw (hence Svelte is arguably more reactive than React and does not relay on immutability either). Mutation methods such as 'array.push' do not but there are simple work-arounds.  Another is the use of a '$:' label to mark expression statements as reactive. Reactive expressions are run before a component redraws when ever any of their dependencies change. Finally, there is a convention of starting a variable name with a $ to generate code that observes the value and also manages automatic subcription / unsubscription to stores.

One concern that might come to mind when considering the use of a compiler is the ease of debugging the output code. In actual fact, I found this to be a non problem. The code is clean and consistent using a few light abstractions that you might well develop yourself when working directly with the DOM. Even the component scope CSS is easy to work with with modern browser tooling, plus you can use global CSS as well.

Svelte is experiencing something of a major surge in interest right now and this is perfectly understandable given version 3.0 bringing together various excellent ideas, hitting a sweet spot for both DX and UK. Personally having been a heavy user of Macro Assemblers, the make tool with scripts and the awesomeness of generics using the C++ processor I fully appreciate the approach of removing work from run time in order to improve both DX and UX.

One final impressive "feature" of svelte well worth mentioning is the excellent learning resources provided by svelte's developer Richard Harris. There's a REPL and the tutorial provide an interactive place to explore which once completed leaves you with a solid understanding of svelte.

Svelte is relatively young but with a well led and fairly small community. Even so, it is poised to have a big impact. In fact, I just saw it prominently featured in the 'State of JavaScript' survey results, taking pride of place next to the big popular frameworks. Watch this space!

I explored much of svelte by developing an inclusive [random number "spinner" app](https://github.com/SteveALee/svelte-donut-spinner) as a fairly non-trivial sample. This might be useful as an example, in addition to the many other resources linked to from [the svelte website](https://svelte.dev/). Svelte has a related project, [sapper](https://sapper.svelte.dev/), which adds features required in a SPA and server side rendering with hydration of the frontend.

Recently, Jason Lengstorf [learnt svelte with Richard Harris](https://www.learnwithjason.dev/let-s-learn-svelte) a process that's fun and informative to watch.
