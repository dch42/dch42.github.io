---
layout: post
title: Adding A Tag Directory Page to Jekyll With Liquid
subtitle:
author: CD
categories: jekyll
tags: liquid jekyll
sidebar: []
---

To see how to create a snippet to display a tag list with post counts, see [this previous post](tag-list-jekyll).

## Creating Our Tags Page

First, we'll need a place for our tag directory to live. 

I'd like them to be available at `/tags/`, so I'll create a file `tags.html` in the root directory of the theme and populate it with front matter:

~~~liquid
---
layout: default
title: Tag Directory
---
~~~

## Constructing A Tag Directory

To make the code reusable elsewhere, we'll make a snippet in our `/_includes/` directory.

We can accomplish our directory layout using a couple of loops, the first being a loop that lists each unique tag for our posts.

In the [previous post](tag-list-jekyll), we established that `site.tags.tag` is comprised of two parts, with the name of the tag located at index `[0]`.

We can create a list of header-styled tags and pre-emptively assign an `id` for later linking:

{% raw %}
~~~liquid
{% for tag in site.tags %}
    <h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
{% endfor %}
~~~
{% endraw %}

To display posts for each tag, we can add another for loop. 

This loop will iterate through `site.posts` and check if they contain the tag from our current parent loop iteration, populating the an unordered list if the condition is met. 

{% raw %}
~~~liquid
{% for tag in site.tags %}
    <h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
        <ul>
            {% for post in site.posts %}
            {% if post.tags contains tag[0] %}
            <li>
                {% if post.date %}
                    {{ post.date | date: "%b %-d, %Y" }}
                {% endif %}
                <h4>
                    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                </h4>
            </li>
            {% endif %}
            {% endfor %}
        </ul>
{% endfor %}
~~~
{% endraw %}

*The example above should work fine with all themes, but I edited the version I'm using on this blog to work with classes and styles for [basically basic](https://github.com/mmistakes/jekyll-theme-basically-basic):*

{% raw %}
~~~liquid
{% for tag in site.tags %}
<h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
<ul>
    {% for post in site.posts %}
    {% if post.tags contains tag[0] %}
    <li>
        <h4>
            <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </h4>
        <footer class="entry-meta">
            <ul>
                {% if post.date %}
                <li><span class="icon">{% include icon-calendar.svg %}</span><time class="entry-time"
                        datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %-d, %Y" }}</time>
                </li>
                {% endif %}
                {% if post.read_time %}
                <li><span class="icon">{% include icon-stopwatch.svg %}</span>{% capture read_time %}{% include
                    read-time.html
                    %}{% endcapture %}{{ read_time | strip }}</li>
                {% endif %}
            </ul>
        </footer>
    </li>
    {% endif %}
    {% endfor %}
</ul>
{% endfor %}
~~~
{% endraw %}

Now we can tie everything together by including it in our `tags.html` file:

{% raw %}
~~~liquid
---
layout: default
title: Tag Directory
---

{% include tag-list.html %}

<br><hr><br>

{% include tag-post-dir.html %}
~~~
{% endraw %}