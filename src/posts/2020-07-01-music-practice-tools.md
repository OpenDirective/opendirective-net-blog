---
title: Music Practice Tools
date: '2020-07-01'
tags:
---

<div class="message">
As an adult learner of bass guitar I find there's much to get to grips with. In addition to instrument technique there's a raft of musicality skills to work on. As a confirmed geek I've long thought about various neat musical projects using the web platform which is now very musically capable thanks to the Web Audio API. And then COVID19 hit us and my lessons became remote, requiring distant sharing of information with my teacher. Add to that my aspirations of keeping a musical journal and <a href="https://musicpracticetools.net">musicpracticetools.net</a> was born as a hybrid blog/journal/app type thing.
</div>

## Tools for practice and remote lessons

Fortunately, my bass teacher encourages me to develop all-round skills and a comprehensive daily practice has evolved. I have a task list to work through and while I could do so with a paper checklist, given my handwriting, I liked the idea of something interactive. Plus it could integrate with a timer so each task could be timed with an indication of when to move on to the next.

Every week we have a video call and holding my scruffy transcriptions up to the camera turned out a pain. It'd be much better if I could share nicely formatted "dots" via a web url. And what if the dots could be played so I could check the transcriptions as I made them?

Then there's the minor annoyance of making repeated changes to the metronome bpm and picking random keys or exercise numbers.

My existing [Youtube current time](https://github.com/music-practice-tools/youtube-current-time) widget will be really useful when practicing music from that source. And a series of links to the music sections would provide a cool way to study song structure.

Other gadgets like a tuner seem unnecessary, though perfectly possible and would make interesting projects. however the solutions I have already are fine. That's solutions like, er, a tuner. Though a single A 440 note would be a useful addition.

Finally, there's that idea of keeping a journal, with notes, ideas, thoughts, a collections of tunes I've work on or would like to learn, and more.

## But what is it exactly

So while this could be some form of web app I had something a bit different in mind. Basically a blog but enriched with on-page widgets to provide interactive tools.

Unlike web apps which are interactive with settings to adjust features, a blog has a definite "write and publish" life cycle. Interactive elements would be configured at write time but any changes will probably still need to be persisted. That might become a problem but when it's just for myself that's really not an issue. I'm happy to hack templated markdown in an editor (VS code these days) and deploy to publish. An app does offer the lucrative advantage of an easy route to monetization via subscripted SaaS, but eventually I decided to make it open source in case others find it useful during these difficult times. The route to using is a little more complex though than an web app, though a simple "deploy to XXX" button on git hub would be ok for technical users, as will hacking on templated markdown.

## The Technical Stack

The metronome key requirement is it has to be easy to create and publish pages and also to include or develop new widgets. That means some sort of publishing framework that also supports client side components, ideal with custom templates rather than partial includes as they are a little prettier. While WordPress is an established solution I'd much prefer static hosting to n-tier dynamic architectures, for quite a few reasons. And these days here's plenty of static site generators to choose from and even the static generation options for component frameworks like Svelte, Next and Nuxt. In the end [11ty](https://www.11ty.dev/) won me over with it's light weight simplicity and flexibility. While it works with many template languages for creating pages, Mozilla's [nunjucks](https://mozilla.github.io/nunjucks/) offers most functionality in 11ty.

For client side we need to create widgets that enhance the static content. I'm happy as a "pig in muck" to craft good ol' DOM manipulation code myself but it does tend to quickly get tedious and "boiler-platey". So something is called for that helps without imposing the full overkill weight and fixed abstractions of a runtime framework like [insert your favourite here]. I'm was keen to use svelte as I think it gets so much right, including minimal load and runtime impact. However, it is usually used for building complete apps with a single component tree and that's not what is wanted here. Svelte will build web components and that certainly a possibility, though at the expense of complicating the build process. There are certainly other interesting lightweight client runtime-only possibilities including [Hyperapp](https://hyperapp.dev/) and [lit-element](https://lit-element.polymer-project.org/) but in the end I stumbled upon [alpinejs](https://github.com/alpinejs/alpine) via [Tailwind UI](https://tailwindui.com/) and it really hit's the sweet spot for this application.

Now what about the creating the musical widgets using the web platform? Many moons ago when I was involved in Mozilla I was mightily impressed by the work at Seneca College led by David Humpreys. That work included a new audio API they were working on. Eventually that led the incredible [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) that we now have to play with. However this is no easy-peasy high level musical tool like Microsoft DirectPlay (now depreciated). The Web Audio API requires a lot of bit twiddling so it's wonderful to have the excellent [ToneJS](https://tonejs.github.io/) library to make it much easier to grapple with. For managing abstract musical concepts like scales we have [tonal](https://github.com/tonaljs/tonal).

And then there's [abcjs](https://paulrosen.github.io/abcjs/) based on the semi-standard musical score text language [abc](http://abcnotation.com/wiki/abc:standard:v2.1#first_and_second_repeats). This adds the icing on the cake providing SVG "dots" that can also be played. The only wrinkle is there is no WYSIWYG editor but rather you describe the notes in the abc language. But that's just fine with me and works well in a publishing workflow.

Finally I had to figure persistence of widget state. That should be minimal but things like timer time need to work as you switch pages (ie across page loads in jamstack). We dont' have cookies or invisible field type hacks as there's no server. This isn't a SPA keeping the top of the DOM tree in place between psuedo page updates. We could similar approach like turbolinks which keeps the Window object and swaps the document body for each page load. In the end I decided to use browser local storage. This requires careful lifecycle management to avoid problems due to lack of a single source of truth. We'll also need to manual manage ids to void problems with page changes and may even need a form of garbage collection. Oh and I just figured there will be problems with multiple tabs. Plus the user will need reset options (or else we tell them to use the browser F12 tools). It's not so surprising this is not a common approach. I may move on fairly soon.

## Making it

Dev and build is handled by Nodejs (what else these days) but only for server side 11ty. The client side is pure un-built un-bundled un-munged HTML, CSS and JavaScript (like your grandma used to make), using global scripts. And why not? It works just fine and I'll bundle etc if and when I need to. Code spliting is easy if required to reduce Javascript load time. I did look at browser modules but there's a few quirks to solve (like modules are deferred which messes up code in the main html file)

And what about deployment? Well, while GitHub pages can be bent to work without Jekyll, they have limitations. GitHub now has Actions to provide Continuous Deployment (CD) etc. and there's Zeit (I mean Vercel). But [Netlify](https://www.netlify.com/) really seem to call me with their good range of services, free pricing tier and community vibe. They also provide Serverless Functions for if/when we do need backend functionality. Netlify have a headless CMS using GitHub for storage that would allow for an alternative in-browser editing / admin UI if that is ever required.

## Outro

So there we have it. A perfectly serviceable stack for a slightly left-field blog/journal/app thing. So far it has  proven more than adequate. I create widgets using client side Javascript, sometimes with Alpine, and 11ty shortcodes to make the markdown page editing experience quick and clean. Local dev of shortcodes with automatic browser update required a little trick with nodemon and browsersync but it works really well.

Here's an example daily [practice page](https://github.com/music-practice-tools/music-practice-tools/blob/7c5b1ad3ec845e859e543f5fabd786c39a16338c/pages/activities/daily.md) as created and a [tune page](https://github.com/music-practice-tools/music-practice-tools/blob/7c5b1ad3ec845e859e543f5fabd786c39a16338c/pages/tunes/the-thrill-is-gone.md).

We're really lucky to have so much great open source software to build on. Hopefully this adds a little useful something to commons.

Right, now I can concentrate much more on developing my bass chops!

Oh, wait, I just thought of a really cool new feature to add.....
