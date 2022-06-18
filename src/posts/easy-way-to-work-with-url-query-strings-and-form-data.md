---
layout: layouts/post.njk
title: Easy way to work with URL Query Strings and Form Data
date: 2022-06-18T08:53:18.871Z
---
In a recent bite sized #WhiteboardTheWeb twee [Use the URL object to fix your query param soup!](https://twitter.com/BHolmesDev/status/1537781446764535816) Ben Holmes showed how the Browser JavaScript `URL` object can make creating query strings much easer.

However as Aleksandr Hovhannisyan pointed out, [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) makes it even easier. You can pass a plain old JavaScript object (POJO) into the contstructor and use `toString()` to get the encoded query string.

Also, it can be used with HTML form data as that has the same encoding "application/x-www-form-urlencoded". Note that multi-part forms, including those with file attachments, use a more conmplex encoding. The reason thet are the same is the key difference between a POST and GET form submission is if the data is passed in the HTTP body or the URL query string.

Node also provides URLSearchParams letting you use it in the backend. In fact, a use here is to convert a form encoded POST body (from a form submission) into a POJO for eashy access. I recently did this for a Netlify Function handling form submissions for the a new W3C WAI website feature. Here's the code.

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