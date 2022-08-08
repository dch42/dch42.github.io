---
layout: post
title: Generating Linked Tag List With Counts In Jekyll With Liquid
subtitle:
author: CD
categories: jekyll
tags: liquid jekyll html css
sidebar: []
---

This post will go over the process of creating a reusable snippet that displays unique tags and tag counts for Jekyll blog posts, with each tag linking to a list of posts containing the tag.

## Preparing Our Div

Our tag list will need a home, so first let's make a tidy `<div>` to contain them. We'll assign a `class` 'tag-list' to style it.

```html
<div class="tag-list">
<h4>Tags:</h4>
</div>
```

I decided to style mine like so:
```css
  .tag-list {
    border: 3px solid gray;
    padding: 7px;
    margin: 7px;
    text-align: center
  }
```

## Getting Tags

In Jekyll, the tags that are assigned to posts are available via `site.tags`. However, `tag` is not simply the tag string. 

For example, the following for loop would return first the name of the tag, followed by each post containing the tag, for each tag on the site:


{% raw %}```liquid
{% for tag in site.tags %}
    {{ tag }}
{% endfor %}
```{% endraw %}


Therefore, `tag[0]` is the name of the tag, and `tag[1]` is an array of all posts containing the tag.

## Adding Tags To Our Div

To make a list of our tags, we will simply loop through `site.tags` and structure a list using `tag[0]` and `tag[1]`.

I would like to have a list that displays tag names and post counts per tag, separated by pipes. Separation can be controlled with with an {% raw %}`{% unless %}`{% endraw %} statement, skipping the last iteration of the for loop.

Because `tag[1]` is an array, we can use the [`.size` filter](https://shopify.github.io/liquid/filters/size/) to get an integer equal to the number of posts containing that tag.

{% raw %}```liquid
{% for tag in site.tags %}
    {{ tag[0] }} ({{ tag[1].size }}){% unless forloop.last %} | {% endunless %}
{% endfor %}
```{% endraw %}

Executing this loop within our div will construct the list for us. We can make the list available throughout the theme by placing the snippet in the `/_includes/` directory. I named mine `tag-list.html`, so it's accessible with:
{% raw %}```liquid
{% include tag-list.html %}
```{% endraw %}

*I've added constructed links to the tagged posts in my version below, and will cover how that works in [this post](link).*

`/_includes/tag-list.html`:
{% raw %}```html
<style>
.tag-list {
    border: 3px solid gray;
    padding: 7px;
    margin: 7px;
    text-align: center
}
</style>

<div class="tag-list">
    <h4>Post Tags:</h4>
    {% for tag in site.tags %}
        <a href="/tags/#{{ tag[0] | slugify }}"> {{ tag[0] }} ({{ tag[1].size }}) </a>{% unless forloop.last %} |
        {% endunless %}
    {% endfor %}
</div>
```{% endraw %}

The resulting snippet should look something like this:
{% include tag-list.html %}

