---
title: Easy way to work with URL Query Strings and Form Data
date: 2022-06-18
tags:
---
In a recent bite sized #WhiteboardTheWeb twee [Use the URL object to fix your query param soup!](https://twitter.com/BHolmesDev/status/1537781446764535816) Ben Holmes showed how the Browser JavaScript `URL` object can make creating query strings much easer.

However as Aleksandr Hovhannisyan pointed out, [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) makes it even easier. You can pass a plain old JavaScript object (POJO) into the constructor and use `toString()` to get the encoded query string.

Here is Ben's example

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

Note that correct URL character encoding/decoding is provided. For example, `?` maps to/from `%3f` and a space to/from `+`.

## Using with Form Data

URLSearchParams can also be used with HTML form data as that has the same encoding "application/x-www-form-urlencoded". Note that multi-part forms, including those with file attachments, use a more conmplex encoding. The reason that are the same is that a key difference between a POST and GET form submission is the data is passed in the HTTP body for a POST and the URL query string for a GET.

Node also provides URLSearchParams letting you use it in the backend. In fact, a use here is to convert a form encoded POST body (from a form submission) into a POJO for easy code access. This is effectively the reverse of passing a POJO to the constructor. I recently did this for a Netlify Function handling form submissions for the a new W3C WAI website feature. Here's the code.

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

This is a little more complex as it implements the old `php` convention of supporting `[]` in the form element name to make the item in the POJO an array. This simplifies data handling. For example, with multiple checkboxes with the same name if there is only a single item checked the default form encoding provides a string value, but if many are check it provides an array. This extended handling ensure the often missed encoding gotcha is easyily handled by always having an array in the POJO.

Here's an example markup using this feature

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

And the resulting POJO when option A only is checked

```javascript
{
  options: [ "option-a" ]
}
```

Whereas, without the `[]` you would get

```javascript
{
  options: "option-a"
}
```
