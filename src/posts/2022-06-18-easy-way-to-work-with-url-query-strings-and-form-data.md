---
title: Easy way to work with URL Query Strings and Form Data
date: 2022-06-18
tags: null
---
<strong>"Rather than catenating encoded strings to make URL query strings and decoding HTML form data with bespoke logic, the `URLSearchParams` object makes it easy to work with these encoded data formats.</strong>

In a bite sized [#WhiteboardTheWeb tweet](https://twitter.com/BHolmesDev/status/1537781446764535816) titled "Use the URL object to fix your query param soup!", Ben Holmes showed how the Browser JavaScript `URL` object can make creating query strings easer.

However, as Aleksandr Hovhannisyan pointed out, [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) makes it even easier. You can pass a plain old JavaScript object (POJO) into the constructor and use `toString()` to get the encoded query string.

Here is Ben's example:

```javascript

const url = new URL(...)
url.search = new URLSearchParams({
  one: 1,
  two: 2,
}).toString()

```

and the resulting URL is: 

``` javascript
"...?one=1&two=2"
```

Note that correct URL character encoding/decoding is also provided. For example, `?` maps to/from `%3f` and a space to/from `+`.

## Decoding Form Data - ideal for serverless functions

URLSearchParams can also be used with HTML form data as that has the same encoding "application/x-www-form-urlencoded". Note that multi-part forms, including those with file attachments, use a more complex encoding. The reason for the same encoding is that the encoded data is passed in the HTTP body for a POST submission and the URL query string for a GET submission. 

As well as browsers, Node also provides URLSearchParams letting you use it in the back-end. This provides an concise way to convert a form encoded POST body from a form submission into a POJO for easy code access. That is effectively the reverse of passing a POJO to the constructor and using toString as show above. 

I recently did tjust his for a Netlify Function [handling form submissions](https://github.com/w3c/wai-website/blob/master/_functions/list-submission.js) for a new W3C WAI website [course submission feature](https://www.w3.org/WAI/courses/submission/). Here's the code:

```javascript
function formEncodedToPOJO(formEncoded) {
  const form = new URLSearchParams(formEncoded)
  return Array.from(form.keys()).reduce((result, key) => {
    const isArrayKey = key.endsWith('[]')
    const targetKey = isArrayKey ? key.slice(0, -2) : key
    result[targetKey] = ((result[targetKey] && !Array.isArray(result[targetKey]))) ? // 2nd checkbox with this key
      form.getAll(targetKey) :
      (isArrayKey) ? form.getAll(key) : form.get(key)
    return result
  }, {})
}
```

This is a little more complex than a basic conversion as it implements the `php` convention of supporting `[]` in the form element name in order to create an array in POJO. This greatly simplifies data handling when using multiple checkboxes with the same name. In this case, if there is only a single item checked the default form encoding provides a string value, but if many are checked it provides an array. Support `[]` ensures this often missed encoding gotcha is easily handled by always having an array in the POJO when the item name specifies an array.

Here's an example markup using this feature:

```html
<fieldset>
  <legend>Options</legend>
  <label>Option A
    <input type="checkbox" name="options[]" value="option-a">
  </label>
  <label>Option B
    <input type="checkbox" name="options[]" value="option-b">
  </label>
</fieldset>
```

And the resulting POJO when option A only is checked:

```javascript
{
  options: [ "option-a" ]
}
```

whereas, without the `[]` you would get:

```javascript
{
  options: "option-a"
}
```

which can be quite a pain to handle in generic code.