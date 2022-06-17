---
title: React, JAMstack and React Static
date: '2019-01-13'
tags:
---

<div class="message">
A simpler development experience, deployment and fast UX with React, JAMstack and React Static.
</div>

When developing [AlwaysInMind](https://github.com/AlwaysInMind/aim-web-app) I made the concious decision to use [React](https://github.com/facebook/react/) but avoided the more sophisticated and opinionated [Nextjs](https://github.com/zeit/next.js). I generally feel it is better to use minimal dependencies and develop your own abstractions according to the problem at hand. This is especially true when you are learning both the technology and the application architecture as you explore your problem domain. I find this working "bottom up" approach to be much more productive and less frustrating than starting with a highly opinionated framework such as Angular. YMMV :)

I finally decided on [Create React App](https://github.com/facebook/create-react-app) (CRA) as providing a good starting point that neatly manages the dev and build configuration yet allows much flexibility in architectural style. This proved to be an excellent choice as I was able to concentrate on my application artefacts without CRA getting in the way.

AlwaysInMind evolved into a rich React SPA coupled with a NodeJS + Ziet [micro](https://github.com/zeit/micro) powered API server that provides a facade to various 3rd party SaaS APIs including Auth0, Google Photos and YouTube. This is a minor variation of the so called '[JAMstack](https://jamstack.org/)' architecture which in many cases provides an excellent developer experience and scalability.

In the JAMstack, HTML artefacts are created at build time, deployed to cheap static hosting, and Javascript provides enhanced behaviour including calls to HTTP APIs. This compares very favourably to the complex DevOps configuration and tighter client / server run time coupling of SSR solutions like NextJS. The React rendering can be performed at build time on the dev machine or on hosting rather than at run time on the server. JAMstack also provides a measure of Progressive Enhancement as a static HTML file is served and can enhanced Javascript behaviour, including React components. This provides a way for features such as user specific content displayed on a static 'template' that can be found search engines (good for SEO).

Recently, NextJS added a static export option that can provide a JAMstack like architecture. But to be honest it is backwards in approach. It takes a full SRR design and scans all the links in an attempt to determine a correct set of static files. That is pretty fragile and anyway, it is a post process to designing in an environment that is all about SSR and so likely to lead to decisions that don't map well.

Two features stand out when analysing what Next offers when exporting to static. The first is the ability to put SPA page components in a folder and have routes automatically provided. This is minor but still useful convenience and in fact it's fairly common to want explicitly declared dynamic child routes of an automatically mapped page.

The other useful feature is the provision of a consistent model for asynchronous data fetching in pages using getInitialProps(). Actually, a JAMStack app is likely to collect data from many sources. More importantly, some of the data can be determined at build time, not left till run time. This could be static data or data fetched from a remote service.

While these features are relatively easy to develop I recently discovered the light weight [React Static](https://github.com/nozzle/react-static/tree/master/) framework. This has a nice clean approach to data provision, separating out the build time determined data into a single configuration file and injecting it page properties. Doing this helps clarify thinking about where and when data is captured, leading to a better UX. As for pages, they can be automatically or explicitly routed as required.

With a few [Core Concepts](https://github.com/nozzle/react-static/blob/master/docs/concepts.md), React Static has very few opinions on top of basic ReactJS. The source code is clean which is an advantage for such young, sole-developer project. Having said that, Tanner Linsley, the developer, is friendly and responsive.

I'm looking forward to exploring React Static for my new project. So far so good.
