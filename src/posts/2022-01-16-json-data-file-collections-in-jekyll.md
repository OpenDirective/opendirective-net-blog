---
title: JSON Data File Collections in Jekyll
date: '2022-01-16'
tags:
---

<div class="message">While Jekyll and liquid provide limited support for collections, it's surprisingly easy to merge a set of JSON files so they can be readily and efficiently accessed on the client.
</div>

## Requirements

We have three similar projects under development at the W3C WAI. Each provides information on a set of items (course, authoring tools and evaluations tools) and users of our Jekyll SSG generated web 'app' wil be able to sort and filter the items.

A form is provided to submit new items that will undergo a review process before eventually being added to the dataset for future access in the app.

So that's only two pages, assuming no pagination.

## Architecture

The need for client side manipulation of the item list view (filter and sort) suggests the JSON data be available to the client.

It would be possible to keep all state in the DOM and manipulate that directly, touh that might be slow. It would also be possible to use the now common pattern of always rendering a view of the data in the client (as made popular by React).

However, as this is a very low complexity app other architectures might also be suitable without a risk of bugs due to complex interactions. For example, using matching data and HTML in the client.

Also, as the content will ony rarely change and Jekyll is a SSG it makes sense to pre-generate as much as possible, both data and HTML. Then custom client side behaviour (ie JavaScript code) can be minimised.

A final observation is that as we are using a SSG, data and HTML can be pre rendered and directly included in pages, rather than being fetched and rendered with JavaScript at runtime in the client.

The need for scalability and pagination might impact these decisions, but initially data sets are small.

This post highlight an approach to the json data SSG rendering that might be non obvious. Options for client side access to that data is the topic for another post.

## Outline

The SSG build time approach is as follows:

- each item has it's own json data file in a folder with the others
- the JSON files have a top level object (hash) s owe can access various fields
- generate a single JSON structure of the items as an array
- Jekyll is used to general a single JSON structure of items
- html for the items can be created in as similar way
- we can use Jekyll `includes` to modularise the code

The data and initial HTML are directly embedded into the HTML files foreasy client access.

So at build time we would like to:

- access the JSON files as a collection
- sort the collection on a specific item field - eg name
- generate a single JSON structure from the files
- iterate over the collection, to generate HTML for example

## Creating the JSON structure

The Jekyll documentation on using [data files](https://jekyllrb.com/docs/datafiles/) is a little lacking and some experimentation is needed. But it turns out to be really easy to generate the JSON from all the files.

### Tools

Liquid has the concept of `arrays`, but provides a very restricted set of `filters` (functions) that create and operate on them. It also has the `[n]` access operator, plus `first` and `last` methods. Jekyll adds a few more useful filters like `push`.

Jekyll also adds `hashes` (objects) and some filters that work on them. Hashes are created for you in standard variables and there is no way to create or manipulate them. The `.` and `[]` operators are provide for getting values.

But we can do what we need to with these limited programming language features.

### Attempt

Given our JSON files live in the `_data` directory, an initial attempt might be as trivial as using the variable `site.data` to access them as a collection. Then we can try things like

```liquid
{%- raw -%}
{{ site.data | sort: 'name' }}
or
{{ site.data.items | jsonify }}
and
{% for item in side.data %} ... {{ item.name }} ... {% endfor %}
{%- endraw -%}
```

But alas, results are not as expected. So with a little use of the `inspect` filter it turns out that:

- `site.data` is a hash keyed by filename and with values that are file contents when parsed as json (based on file extension being .json)
- the `jsonify` filter shows an object (hash) with the same structure
- when iterating using `for` each file item is converted into a 2-tuple of [key, value]
- sorting appears to work using the filename key

### Solution

The solution is straightforward. As we don't care about the filenames we can convert the hash to an array of the values. Then all the operation work as required. Here's the code:

```liquid
{% raw %}
# create an empty array and add the hash item values
# note push is non mutating
{% assign values = "" | split: "," %}
{% for item in site.data -%}
    {% assign values = item[1] %}
    {% assign values = values | push: value %}
{% endfor %}
{% endraw %}
```

Note there's no "return" from includes so the output is any variables we set. In this case it's values.

Now you can perform those operations we tried before:

```liquid
{% raw %}
{{ values | sort: "name" }}
{{ values | jsonify }}
{% for value in values %} ... {{ value.name }} ... {% endfor %}
{% endraw %}
```

As a optimisation you can put the code in an `include`, passing the data folder (you can use subfolders of `_data` to organise things) and a sort key.

```liquid
{% raw %}
{% include sort-data-folder.liquid data=site.data sortKey="name" %}
{% endraw %}
```
