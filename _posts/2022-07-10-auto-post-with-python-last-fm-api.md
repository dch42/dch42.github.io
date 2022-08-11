---
layout: post
title: Automated Jekyll Posts With Python and Last.fm API
subtitle: Programatically creating markdown files from fetched Last.fm data
author: CD
categories: python
tags: python api
sidebar: []
---

#### Last.fm API
As a long-term user of [Last.fm](https://last.fm), I've enjoyed tracking and quantifying my music listening habits over the years. 

The Last.fm API allows us to do all sorts of neat things with that data, so I decided it would be a fun mini-project to integrate a script with this blog to fetch and format my weekly listening data as a nice, Jekyll-friendly markdown file.

#### API Endpoint
This project interacts with two user methods, `getUserArtistChart` and `getUserAlbumChart`. Data will be parsed from JSON, and each request limited to the top ten items.

Thus the request URL format looks like this:

~~~
https://ws.audioscrobbler.com/2.0/?method=user.get{method}&user={username}&api_key={key}&format=json&limit=10
~~~

#### YAML Config
To keep the script sanitary, I stash `username` and `key` in a yaml config file, alongside strings defining Jekyll post front matter and an article intro blurb. The data from the config file is reassembled in the python script:

~~~python
front_matter = f"""---
layout: {layout}
title: {title} {date}
subtitle: {subtitle}
author: {author}
categories: {categories}
tags: {tags}
sidebar: []
---
"""
post_intro = f"""
{content}
"""
~~~

#### Markdown Formatting
Markdown has nice [lazy table syntax](https://www.w3schools.io/file/markdown-table/), so markdown table definitions are assigned to multiline strings to be written to the outfile later:

~~~python
album_table = """
### Top Albums

rank | artist | album | plays
:---:|---|---|:---:"""

artist_table = """
### Top Artists

rank | artist | plays
:---:|---|:---:"""
~~~

The following function is used to append this formatting data and parsed API response to the post file:

~~~python
def write_md(text):
    """Write data to Jekyll post formatted md file"""
    md_file = f'./{date}-weekly-music-stats.md'
    with open(md_file, 'a') as md:
        md.write(text+"\n")
~~~

#### Fetching, Parsing, and Exporting API Response
The next function parses the JSON data from the API request, and appends it to the outfile row-by-row, table-formatted with markdown (items separated with pipes).

The `getWeeklyAlbumChart` method doesn't return urls for artist pages, but they should be able to be created by appending the music root url with the artist name string, using python's `replace()` method to swap spaces for `+`.

~~~python
def fetch_chart_data(method):
    """Fetch data from last.fm API and append to markdown file"""
    if method == 'weeklyartistchart':
        write_md(artist_table)
    else:
        write_md(album_table)
    url = f'https://ws.audioscrobbler.com/2.0/?method=user.get{method}&user={username}&api_key={key}&format=json&limit=10'
    response = requests.get(url).json()
    if method == 'weeklyartistchart':
        for item in response['weeklyartistchart']['artist']:
            row = f"{item['@attr']['rank']} | [{item['name']}]({item['url']}) | {item['playcount']}"
            write_md(row)
    else:
        for item in response['weeklyalbumchart']['album']:
            artist_url = f"https://last.fm/music/{item['artist']['#text'].replace(' ', '+')}"
            row = f"{item['@attr']['rank']} | [{item['artist']['#text']}]({artist_url}) | [{item['name']}]({item['url']}) | {item['playcount']}"
            write_md(row)
~~~

#### Final Steps
Finally, the generated markdown file must be moved to the Jekyll `_posts` directory, defined in config yaml file. 

~~~python
def post_to_jekyll():
    """Move post to Jekyll _posts dir"""
    os.system(f'cp -v ./{date}-weekly-music-stats.md {posts_path}')
~~~

And thus, our `main()` looks something like this:

~~~python
def main():
    for txt in front_matter, post_intro:
        write_md(txt)
    for method in 'weeklyartistchart', 'weeklyalbumchart':
        fetch_chart_data(method)
    post_to_jekyll()
~~~

To further automate the process, I may try to add it to a Github Actions workflow in the future, or just make it a cronjob :)

*To interact with the [Last.fm API](https://www.last.fm/api), you’ll need to apply for an [API account](https://www.last.fm/api/account/create).*