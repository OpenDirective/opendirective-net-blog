---
title: TWAM Proof of Concept app - using Netlify, Eleventy and Google Sheets
date: '2020-08-25'
tags:
---

<div class="message">
A new project with yet a different stack. This time it's a Jamstack static site with a serverless back end using most of Netlify's services. On this platform Eleventy provides templated page build with no front end framework in sight. Google Sheets is used as a database. This post explains some of the technical decisions made and challenges met.
</div>

## Digitising Tools with a Mission's process

Tools with a Mission ([TWAM](https://www.twam.uk/)) recycle and refurbish tools, sewing machines and computers. It then sends them to the developing world for livelihood creation. Their processes are reasonably complex and currently use a mixture of Google Apps tools, including GMail and Sheets. I'm working in a small team to help move more of the process to an integrated 'web app' experience. This will support customer applications for tools, evaluation, approval and other steps through to shipping in containers.

This is a pretty standard setup of an app with forms plus a back office process which defines the  workflow. Thus we looked at the usual candidates for forms and workflow. I personally find such systems to be a quite restrictive, especially if they only use a GUI and cannot be data or code driven. Given TWAM's embedded use of Goggle Apps we also considered Google Action script which is basically a hosted JavaScript environment with some support for creating web apps and APIs.

In the end however, my very positive experience with using Netlify Deployments along with Eleventy for [https://musicpracticetools.net](https://musicpracticetools.net) meant I was keen to try more of the Netlify portfolio of serverless services. However, for the database, rather than the pNetlify partner noSQL Fauna, I decided to try using Google Sheets. Using sheets opens up the possibility of easy access to the data and ad-hoc processing using office software skills rather than full developer skills. In addition, Gmail is used for inter-person messaging and for automatic notifications. Again, why code a new messaging solution when a good one exists and fits in with people's work patterns. So, the stack chosen for the proof of concept is:

* Eleventy with nunjucks templates
* Good ol' HTML, CSS and Javascript - ie no framework
* Google Sheets API for database
* Gmail API for notifications
* Netlify deploys, forms, functions, identity and role based gating
* Netlify dev - not really part of the stack but excellent

This stack supports a representative slice of functionality including user management, form filling with editing and notifications. The proof of content flow is that someone makes an application and can check back later to see if successful Someone else reviews a subset of applications filtered by country and adds evaluation notes and assigns an accept/reject status.

## Tricky areas

I'd already ironed out any small issues with Eleventy and nunjucks in my last project so just used the same config again. Netlify deploys are really smooth to use with github integration and pretty much just work; as do the forms which invoke a function when submitted and also functions which wrap AWS Lamda, making them a "doddle" to use.

The fun and games came with Google Sheets and Netlify identity used in conjunction with the role based gating.

### Google Sheets

Google provide a REST API for Sheets and also the Action Script environment with support for creating HTTP APIs. They also maintain a npm package binding JavaScript to ALL their APIs, though there is a simpler independent package just for sheets. We can most easily call the JavaScript binding from our Netlify Functions rather than poking the REST API via fetch. I decided to use the simpler `google-spreadsheet` package, though it turns out I might switch to the Google monster package as it supports batching API calls for efficiency.

But we soon hit an major issue. For some reason the latest v4 of the Google Sheets API does not support any filtering when fetching sheet cells. And v3 is about to be dropped. That's obviously a scalability problem! We don't want to load all columns and rows into memory for processing. I looked at using Action Script to create an API which does work but there are two issues here. 1) the script have to be edited in Google's in browser editor which is a terrible experience compared to something like VS Code. Perhaps more serious is the fact that in order to expose the API endpoint the Google Apps setting to allow anyone to share items publicly must be enabled. That's a potential security issue and might not be welcome.

After much head scratching I found a work around to this. YOu can use a sheet formula to process the sheet data as required and return the filtered row set. Further more, I decided to reduce coupling and error possibilities, that rather than keeping the formula in the spreadsheet we'd push it from the app and read the results back. In addition, in order to have a have chance of handling multiple accesses at once a new sheet is created for the formula and then deleted. So the process to get a filtered row set is

1. Create a new worksheet in the spreadsheet
2. Push the Formula to a cell so it runs
3. Fetch the results
4. Delete the worksheet

This works well but does seem a little "hacky" and I still need to test with multiple simultaneous updates. It is also slow, taking around 4s. This appears to be purely communication time with so many round trips. Adding a thousand rows to the sheet makes little difference to the timing. The official Google npm package provides batching mode which should speed things up. You need some good sheet formula skills but can develop in the sheet and move to JavaScript once it all works. In order to further reduce coupling and hardcoded values the formula returns the size of the result set and column names as well as the data itself to the app. The app can read that data first and then use it to fetch the correct cells.

### Netlify Identity and Role Based Gating.

Identity along with account management is always a complex topic. Fortunately, Netlify provide a register / login widget that is perfectly serviceable, though nowhere near as flexible or polished as the one from Auth0. I found I could improve the UX to my liking with a little script, though have not yet succeeded with the CSS tweaks I want. Identity is based on stateless JWT tokens as you'd pretty much expect these days. These are flexible and you can pass the user's id_token to your Functions to provide access control and user info. This way, Netlify manages the users and their details are passed around the front and back ends in the tokens.

The feature that I really appreciate is Role Based gating. This means access to specific app page URIs is controlled declaratively right on the CDN edge almost without any code required. When a user registers and logs in a Function is called and this can return the users roles. Then a config file is used to define which roles are required per URI. And it just works.

Well almost. I hit a lot of strange behavours and errors with access, including an apparent failure when logging in on another browser when still logged in. It turned out to be down to my limited cognitive model of how the pieces work together, not helped by very limited documentation. The gating and identity work differently but do get synchronised if you are careful. This is where Netlify's Community forum came to the rescue in the form of one [Jon Sullivan](https://jon.fm). As a volunteer Jon is incredibly knowledgeable and definitely goes the extra mile, and then some. After a long an fascinating discussion covering how authentication and gating interact and how to handle refresh tokens I now have code which is reliable. It just took a very long time to get there. But we did get a 100+ post thread going, FTW.

## Summary

A important point I should make clear is that ALL this functionality from Netlify is available on the *free* plan! Though, Forms do have their own pricing model after 100 submissions. That's very impressive indeed and perfect for a charity with very little cash to spare.

The stack threw me a few big challenges but now it is working well and I think will scale well to the final app.

The code is available at [https://github.com/OpenDirective/TWAM-App](https://github.com/OpenDirective/TWAM-App)
