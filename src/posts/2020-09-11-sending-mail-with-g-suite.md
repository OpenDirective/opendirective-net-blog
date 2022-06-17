---
title: Sending emails with the G Suite Gmail API
date: '2020-09-11'
tags:
---

<div class="message">
If you have a G Suite instance then sending emails from a backend using Gmail turns out to be really simple. Using the Gmail API rather than SMTP and enabling G Suite global delegation means you can simply send email as a G Suite user. Here's how I did it with Netlify Functions.
</div>

## Email options for backend

In my previous post on the Tools with a Mission ([TWAM](https://www.twam.uk/)) app I mentioned the need to send email notifications of various events. TWAM is a non profit usr of Google's [G Suite](https://gsuite.google.com/) and all users requiring notifications have appropriate G Suite Gmail addresses. The app is using Google Sheets as the backend database and it makes sense to also use Gmail to send emails, rather than using yet another service.

It might seem obvious to just use the same SMTP API that email clients use to send and receive via gmail. However, a little research indicates this approach has issues, largely as Google are aggressive in disabling accounts which seem to be suspect, with little recourse to restoration. We obviously can't risk that for a mission critical app.

As the TWAM app has a backend of Netlify Functions (which simplify AWS Lambda) running on nodejs, another sensible choice is using [nodemailer](https://nodemailer.com/about/) which has a facade for Gmail. Again however, the documentation indicates problems with Gmail and it's a heck of a lot of abstraction on top of the the Gmail APIs, with support for many other transports.

Thus direct Gmail use is my preferred solution, assuming it can be made to work with a reasonable "pain threshold". As usual with Google APIs, the documentation is poor, confusing and often downright misleading due to multiple versions being around on the interwebs. Worse, when trying to find solutions in Stack overflow etc, well, "there be Dragons" as they used to say on old maps for unknown dangers.

But in the end after many, many, failed and very frustrating attempts the solution using the Gmail REST API directly is pleasantly simple. You just need to get all the parts lined up "just so". Google provide a JavaScript API for node , but that is a monster monolith covering ALL Google's APIs. I don't expect so much deployed code will run up big costs on Netlify Serverless Functions, but the thought of using such a "Swiss Army Knife" binding leaves a nasty taste in my mouth. REST will do just fine, thank you; after all, it's the way of the web.

## Code

Reading up on the subject of using the Gmail API throws up several complexities that it turns out are just not required:

### No need to get the user to authorise using an OAuth/OpenID flow

This is just as well because we want to send emails as the app backend, not a specific user who is logged into the app. To do so, we can set up a Google Service. This actually uses OAuth JWTs "under the bonnet" but it's much simpler for us to use than the usual flows. The service use an email address for identification and a private key for security. We can use the individual `google-auth-library` package that is part of the Google JavaScript APIs. The final piece of the puzzle is to use a G Suite 'Global Delegation' setting to allow the Service to send emails as a user. This way, the app calls the REST API and Gmail sends the a email as the specified Gmail user. To set this up See the [Google documentation](https://developers.google.com/admin-sdk/directory/v1/guides/delegation) and ensure the correct scopes are enabled for sending emails (the used value of 'https://mail.google.com/' seems too course and can probably be refined).

### No need to BASE64 encode content

While nearly all examples show the HTTP message body for the email being BASE64 encoded this appears uneccessary. You only need to specify the correct `Content-Type` header. Anyway, BASE64 encoding is certainly not part of the RFC 822 email specification. Perhaps we'll hit some content that requires it but for now I'm only sending single part text content (and unicode works just fine).

### No need to use NodeFetch or NAXIOS

`google-auth-library` includes a dependency on Gaxios, a version of AXIOS that works just fine for our purposes. It's called via a wrapper that retries under some auth error circumstances.

### The email functions

`sendRawEmail` calls the Gmail REST endpoint with appropriate security provided via Netlify variables in deployment or a `.env` shell environment configuration file during development.

`sendEmail` builds up a RFC 822 message body for common email fields and calls `sendRawEmail`.

```javascript
// file - functions/_gmail.js

if (!process.env.NETLIFY) {y
  // use .enc file for local dev and assume netlify variables in CI
  require('dotenv').config()
}

if (!process.env.GOOGLE_SERVICE_ACCOUNT_EMAIL)
  throw new Error('no GOOGLE_SERVICE_ACCOUNT_EMAIL env var set')
if (!process.env.GOOGLE_PRIVATE_KEY)
  throw new Error('no GOOGLE_PRIVATE_KEY env var set')
if (!process.env.GMAIL_SENDING_USER)
  throw new Error('no GMAIL_SENDING_USER env var set')

const { JWT } = require('google-auth-library')

// For this to work you must create a service and enable Domain wide delegation for the service
// https://developers.google.com/admin-sdk/directory/v1/guides/delegation
// Set env vars for the service key in GOOGLE_SERVICE_ACCOUNT_EMAIL & GOOGLE_PRIVATE_KEY
// GMAIL_SENDING_USER is the email address that the service delegates for
// ensure you set the used scopes here when enabling the global delegation
async function initServiceClient() {
  return new JWT({
    email: process.env.GOOGLE_SERVICE_ACCOUNT_EMAIL,
    key: process.env.GOOGLE_PRIVATE_KEY.replace(/\\n/g, '\n'),
    scopes: ['https://mail.google.com/'], // better to pick narrower scopes
    subject: process.env.GMAIL_SENDING_USER,
  })
}

// email body is to rfc822. (From: is ignored and GMAIL_SENDING_USER used ) eg
// To: someone_else@example.com
// Subject: An RFC 822 formatted message
//
// This is the plain text body of the message. Note the blank line
// between the header information and the body of the message.
exports.sendRawEmail = async function (emailBody) {
  const client = await initServiceClient()

  // see https://github.com/googleapis/gaxios
  const options = {
    method: 'POST',
    url: `https://gmail.googleapis.com/upload/gmail/v1/users/me/messages/send`,
    headers: {
      'Content-Type': 'message/rfc822',
    },
    body: emailBody,
  }
  return await client.request(options)
}

const field = (f, v) => (v ? f + ': ' + v + '\r\n' : '')

exports.sendEmail = async function ({
  to = undefined,
  cc = undefined,
  bcc = undefined,
  subject = '',
  message = '',
} = {}) {
  if (!to) {
    throw new Error('"To" field is required')
  }

  const email = `${field('Subject', subject)}${field('To', to)}${field(
    'cc',
    cc,
  )}${field('bcc', bcc)}\r\n${message}`
  return exports.sendRawEmail(email)
}
```

## Extra - templates in 3 lines

I also required a simple template for the emails. Something like mustache (or handlebars) is just fine. Lodash provides the necessary.

{% raw %}
```javascript
const template = require('lodash.template')

function parseTemplate(templ, data) {
  const options = { interpolate: /{{([\s\S]+?)}}/g } // mustache style
  const compiled = template(templ, options)
  return compiled(data)
}

 const notification = {
      to: recipient.email,
      subject: parseTemplate(email.subject, templData),
      message: parseTemplate(email.message, templData),
    }
  sendEmail(notification)
```
{% endraw %}

