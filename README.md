# Jekyll::Paginate::Content

[![Gem Version](https://badge.fury.io/rb/jekyll-paginate-content.svg)](https://badge.fury.io/rb/jekyll-paginate-content)

*Jekyll::Paginate::Content* (JPC) is a plugin for [Jekyll](https://jekyllrb.com/) that automatically splits pages, posts, and other content into one or more pages, at points where `<!--page-->` *(configurable)* is inserted. It mimics [jekyll-paginate-v2](https://github.com/sverrirs/jekyll-paginate-v2) (JPv2) naming conventions and features, so if you use that, there is almost nothing new to learn.

**Features:** Automatic content splitting into several pages, configurable permalinks, page trail, single-page view, SEO support.

- [Jekyll::Paginate::Content](#jekyll--paginate--content)
  * [TL;DR](#tldr)
  * [Why use this?](#why-use-this)
  * [Installation](#installation)
  * [Configuration](#configuration)
  * [Usage](#usage)
  * [Properties](#properties)
  * [Page/Post properties](#pagepost-properties)
    + [Setting custom properties](#setting-custom-properties)
    + [Overriding and restoring properties](#overriding-and-restoring-properties)
      - [Special values](#special-values)
    + [Default properties](#default-properties)
    + [Example](#example)
  * [Pagination trails](#pagination-trails)
    + [Usage](#usage-1)
  * [Search Engine Optimization (SEO)](#search-engine-optimization-seo)
    + [Automatic](#automatic)
    + [Manual](#manual)
  * [TODO](#todo)
  * [Demo](#demo)
  * [Contributing](#contributing)
  * [License](#license)
  * [Code of Conduct](#code-of-conduct)
  * [Also by the Author](#also-by-the-author)

## TL;DR

```markdown
---
title: JPC demo
layout: page
paginate: true
---

{% if paginator.paginated %}
  <a href="{{ paginator.single_page }}">View as a single page</a>
{% elsif paginator %}
  <a href="{{ paginator.first_path }}">View as {{ paginator.total_pages }} pages</a>
{% endif %}

This shows up at the top of all pages.

<!--page_header-->
This is page 1 of the JPC example.

<!--page-->
This is page 2.

{% if paginator.paginated %}
<p>This won't show up in the single-page view.</p>

<p><a href="{{ paginator.next_page_path }}">Go on to page {{ paginator.next_page }}</a></p>
{% endif %}

<!--page-->
This is page 3.

<!--page-->
This is page 4.

<!--page-->
I have a [link] here in page 5.
{% if paginator.paginated %}
We're near the last page (page {{ paginator.last_page }}).
{% endif %}

<!--page-->
This is page 6.

<!--page-->
This is the last page.

<!--page_footer-->
This goes into all the pages, too!

[link]: https://ibrado.org/
```

## Why use this?

1. You want to split long posts and pages/articles/reviews, etc. into multiple pages, e.g. chapters;
1. You want to offer faster loading times to your readers;
1. You want more ad revenue from your Jekyll site.

## Installation

Add the gem to your application's Gemfile:

```ruby
group :jekyll_plugins do
  # other plugins here
  gem 'jekyll-paginate-content'
end
```

And then execute:

    $ bundle

Or install it yourself:

    $ gem install jekyll-paginate-content

## Configuration

No configuration is required to run *Jekyll::Paginate::Content*. If you want to tweak its behavior, you may set the following options in `_config.yml`:

```yaml

paginate_content:
  #enabled: false                    # Default: true
  debug: true                        # Show additional messages during run; default: false
  #collection: pages, "articles"     # Which collections to paginate; default: pages and posts
  collections:                       # Ditto, just a different way of writing it
    - pages                          # Quotes are optional if collection names are obviously strings
    - posts
    - articles

  auto: true                         # Set to true to search for the page separator even if you
                                     #   don't set paginate: true in the front-matter
                                     #   NOTE: This is slower. Default: false

  separator: "<!--split-->"          # The page separator; default: "<!--page-->"
  header: "<!--head-->"              # The header separator; default: "<!--page_header-->"
  footer: "<!--foot-->"              # The footer separator; default: "<!--page_footer-->"

  permalink: '/borg:numof:max.html'  # Relative path to the new pages; default: "/:num/"
                                     #   :num will be replaced by the current page number
                                     #   :max will be replaced by the total number of page parts
                                     # e.g. /borg7of9.html

  single_page: '/full.html'          # Relative path to the single-page view; default: "/view-all/"

  title: ':title - :num/:max'        # Title format of the split pages, default: original title
                                     #   :num and :max are as in permalink, :title is the original title

  retitle_first: false               # Should the first part be retitled too? Default: true

  trail:                             # The page trail settings: number of pages to list
    before: 3                        #   before and after the current page
    after: 3                         #   Omit or set to 0 for all pages (default)

  seo_canonical: false               # Set link ref="canonical" to the view-all page; default: true

  #properties:                       # Set properties per type of page, see below
  #  all:
  #    field1: value1
  #    # ...etc...
  #  first:
  #    field2: value2
  #    # ...etc...
  #  part:
  #    field3: value3
  #     # ...etc...
  #   last:
  #    field4: value4
  #    # ...etc...
  #  single:
  #    field5: value5
  #    # ...etc...

```

Here's a cleaned-up version with the defaults:

```yaml

paginate_content:
  #enabled: true
  #debug: false

  #collections:
  #  - pages
  #  - posts

  #auto: false

  #separator: "<!--page-->"
  #header: "<!--page_header-->"
  #footer: "<!--page_footer-->"

  #permalink: '/:num/"
  #single_page: '/view-all/'

  #title: ':title'
  #retitle_first: true

  #trail:
  #  before: 0
  #  after: 0

  #seo_canonical: true

  #properties:
  #  all:
  #  first:
  #  part:
  #  last:
  #  single:

```


## Usage

Just add a `paginate: true` entry to your front-matter:

```yaml
---
title: Test
layout: post
date: 2017-12-15 22:33:44
paginate: true
---
```

or set `auto` to `true` in your `_config.yml`:

```yaml
paginate_content:
  auto: true
```

Note that using `auto` mode will be slower.

## Properties

These properties/fields are available to your layouts and content via the `paginator` object, e.g. `{{ paginator.page }}`.


| Field                | Aliases         | Description                         |
|----------------------|-----------------|-------------------------------------|
| `first_page`         |                 | First page number, i.e. 1           |
| `first_page_path`    | `first_path`    | Relative URL to the first page      |
| `next_page`          |                 | Next page number                    |
| `next_page_path`     | `next_path`     | Relative URL to the next page       |
| `previous_page`      | `prev_page`     | Previous page number                |
| `previous_page_path` | `previous_path`<br/>`prev_path` | Relative URL to the previous page   |
| `last_page`          |                 | Last page number                    |
| `last_page_path`     | `last_path`     | Relative URL to the last page       |
| `page`               | `page_num`      | Current page number                 |
| `page_path`          |                 | Path to the current page            |
| `page_trail`         |                 | Page trail, see [below](#trails)    |
| `paginated`          | `activated`     | `true` if this is a partial page    |
| `total_pages`        | `pages`         | Total number of pages               |
|                      |                 |                                     |
| `single_page`        | `view_all`      | Path to the original/full page      |
| `seo`                |                 | HTML header tags for SEO, see [below](#seo) |
|                      |                 |                                     |
| `has_next`           |                 | `true` if there is a next page      |
| `has_previous`       | `has_prev`      | `true` if there is a previous page  |
| `is_first`           |                 | `true` if this is the first page    |
| `is_last`            |                 | `true` if this is the last page     |
| `next_is_last`       |                 | `true` if this page is next-to-last |
| `previous_is_first`  | `prev_is_first` | `true` if this is the second page   |


## Page/Post properties

These properties are automatically set for pages/documents that have been processed, e.g `{{ post.autogen }}`

| Field                | Description
|----------------------|----------------------------------------------------------------------
| `permalink`          | Relative path of the current page
|                      |
| `hidden`             | `true` for all pages (including the single-page view) except the first page
| `tag`, `tags`        | `nil` for all except the first page
| `category`, `categories` | `nil` for all except the first page
|                      |
| `autogen`            | "jekyll-paginate-content" for all but the single-page view
| `pagination_info`    | `.curr_page` = current page number<br/>`.total_pages` = total number of pages<br/>`.type` = "first", "part", "last", or "single"<br/>`.id` = a string which is the same for all related pages (nanosecond timestamp)

The tags, categories, and `hidden` are set up this way to avoid duplicate counts and having the parts show up in e.g. your tag index listings. You may override this behavior as discussed [below](#override).

### Setting custom properties

`paginate_content` in `_config.yml` has a `properties` option:

```yaml
paginate_content:
  properties:
    all:
      field1: value1
      # ...etc...
    first:
      field2: value2
      # ...etc...
    part:
      field3: value3
      # ...etc...
    last:
      field4: value4
      # ...etc...
    single:
      field5: value5
      # ...etc...
```

where the properties/fields listed under `all` will be set for all pages, `first` properties for the first page (possibly overriding values in `all`), etc.

**Example:** To help with your layouts, you may want to set a property for the single-page view, say, activating comments:

```yaml
paginate_content:
  properties:
    single:
      comments: true
```

In your layout, you'd use something like

```html
{% if post.comments %}
   <!-- Disqus section -->
{% endif %}
```

The single-page view would then show the [Disqus](https://disqus.com/) comments section. 

<a name="override"></a>
### Overriding and restoring properties

You can set almost any front-matter property via the `properties` section, except for `title`, `layout`, `date`, `permalink`, and `pagination_info`. Use with caution.

#### Special values

You may use the following values for properties:

| Value | Meaning
|-------|--------------------------------------
| `~`   | `nil` (in essence, disabling the property)
| `$`   | The original value of the property
| `$.property` | The original value of the specified `property`
| `/`   | Totally remove this property
 
### Default properties

For reference, the default properties effectively map out to:

```yaml
  properties:
    all:
      autogen: 'jekyll-paginate-content'
      hidden: true
      tag: ~
      tags: ~
      category: ~
      categories: ~
      pagination_info:
        curr_page: (a number)
        total_pages:  (a number)
        id: '(a string)'

    first:
      hidden: false
      tag: $
      tags: $
      category: $
      categories: $
      pagination_info:
        type: 'first'

    part:
      pagination_info:
        type: 'part'

    last:
      pagination_info:
        type: 'last'

    single:
      autogen: ~
      pagination_info:
        curr_page: /
        total_pages: /
        type: 'full'
```

### Example

As an example, the author's `_config.yml` has the following:

```yaml
  properties:
    all:
      comments: false
      share: false
      x_title: $.title

    #first:
      # keeps original tags and categories

    part:
      x_tags: []
      x_cats: []

    last:
      comments: true
      share: true
      x_tags: $.tags
      x_cats: $.categories

    single:
      comments: true
      share: true
      x_tags: $.tags
      x_cats: $.categories
```

`x_tags` and `x_cats` are used in this case to store the original tags and categories for generating a list of related posts only for last pages or single-page views. `comments` and `share` are likewise used to turn on the sections for comments and social media sharing for these pages.

`x_title` is used to save the original title and use that in social media sharing. The example below also does something similar for the share URL:

```
{% if paginator.first_path %}
  {% assign share_url = paginator.first_path %}
{% else %}
  {% assign share_url = page.url %}
{% endif %}

{% if page.x_title %}
  {% assign share_title = page.x_title %}
{% else %}
  {% assign share_title = page.title %}
{% endif %}
```

<a name="trails"></a>
## Pagination trails

You use `paginator.page_trail` to create a pager that will allow your readers to move from page to page. It is set up as follows:

```yaml
paginate_content:
  trail:
    before: 2
    after: 2
```

`before` refers to the number of page links you want to appear before the current page; similarly, `after` is the number of page links after the current page. So, in the above example, you have 2 before + 1 current + 2 after = 5 links to pages in your trail "window".

If you don't specify the `trail` properties, or set `before` and `after` to 0, all page links will be returned.

Let's say your document has 7 pages, and you have a `trail` as above. The pager would look something like this as you go from page to page:

<pre><strong>&laquo; <1> [2] [3] [4] [5] &raquo;
&laquo; [1] <2> [3] [4] [5] &raquo;
&laquo; [1] [2] <3> [4] [5] &raquo;
&laquo; [2] [3] <4> [5] [6] &raquo;
&laquo; [3] [4] <5> [6] [7] &raquo;
&laquo; [3] [4] [5] <6> [7] &raquo;
&laquo; [3] [4] [5] [6] <7> &raquo;
</strong></pre>

### Usage

`paginator.page_trail` has the following fields:

| Field   | Description
|---------|-------------------------------------- 
| `num`   | The page number
| `path`  | The path to the page
| `title` | The title of the page

Here is an example lifted from [JPv2's documentation](https://github.com/sverrirs/jekyll-paginate-v2/blob/master/README-GENERATOR.md#creating-pagination-trails):


```html
{% if paginator.page_trail %}
  <ul class="pager">
  {% for trail in paginator.page_trail %}
    <li {% if page.url == trail.path %}class="selected"{% endif %}>
        <a href="{{ trail.path | prepend: site.url }}" title="{{trail.title}}">{{ trail.num }}</a>
    </li>
  {% endfor %}
  </ul>
{% endif %}
```

and its [accompanying style](https://github.com/sverrirs/jekyll-paginate-v2/blob/master/examples/03-tags/_layouts/home.html):

```html
<style>
  ul.pager { text-align: center; list-style: none; }
  ul.pager li {display: inline;border: 1px solid black; padding: 10px; margin: 5px;}
  .selected { background-color: magenta; }
</style>
```

You'll end up with something like this, for page 4:

<p align="center">
  <img src="https://raw.githubusercontent.com/ibrado/jekyll-paginate-content/development/res/jpv2-trail.png" />
</p>

The author's own pager is a little more involved:

```html
{% if paginator.page_trail %}
  <div class="pager">
    {% if paginator.is_first %}<span class="pager-inactive">
    {% else %}<a href="{{ paginator.first_path }}">{% endif %}
    <i class="fa fa-fast-backward" aria-hidden="true"></i>
    {% if paginator.is_first %}</span>{% else %}</a>{% endif %}

    {% if paginator.has_previous %}<a href="{{ paginator.previous_path }}">
    {% else %}<span class="pager-inactive">{% endif %}
    <i class="fa fa-backward" aria-hidden="true"></i>
    {% if paginator.has_previous %}</a>{% else %}</span>{% endif %}

    {% for p in paginator.page_trail %}
      {% if p.num  == paginator.page_num %}
       {{ p.num }}
      {% else %}
      <a href="{{ p.path }}" data-toggle="tooltip" data-placement="top" title="{{ p.title }}">{{ p.num }}</a>
      {% endif %}
    {% endfor %}

    {% if paginator.has_next %}<a href="{{ paginator.next_path }}">
    {% else %}<span class="pager-inactive">{% endif %}
    <i class="fa fa-forward" aria-hidden="true"></i>
    {% if paginator.has_next %}</a>{% else %}</span>{% endif %}

    {% if paginator.is_last %}<span class="pager-inactive">
    {% else %}<a href="{{ paginator.last_path }}">{% endif %}
    <i class="fa fa-fast-forward" aria-hidden="true"></i>
    {% if paginator.is_last %}</span>{% else %}</a>{% endif %}
  </div>
{% endif %}
```

This results in a pager that looks like this:

<p align="center">
  <img src="https://raw.githubusercontent.com/ibrado/jekyll-paginate-content/development/res/ajni-trail-p1.png" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/ibrado/jekyll-paginate-content/development/res/ajni-trail-p4.png" />
</p>

Of course, you always have the option of adding some navigational cues to your content:

```html
{% if paginator.paginated %}
  <a href="{{ paginator.next_page_path }}">On to the next chapter...</a>
{% endif %}
```

This text will not appear in the single-page view.

<a name="seo"></a>
## Search Engine Optimization (SEO)

Now that your site features split pages (*finally!*), how do you optimize it for search engines?

`paginator.seo` has the following fields:

| Field       | Description
|-------------|-------------------------------------- 
| `canonical` | HTML `link rel` of the canonical URL (primary search result for particular content)
| `prev`      | Ditto for the previous page, if applicable
| `next`      | Ditto for the next page, if applicable
| `links`     | All of the above, combined

### Automatic

If you have a dedicated HTML header for the content that you paginate, simply add the following somewhere inside the <tt>&lt;head&gt;</tt>:

```
{{ paginator.seo.links }}
```

It will produce up to three lines, like so (assuming you are on page 5):

```html
  <link rel="canonical" href="https://example.com/2017/12/my-post/view-all/" />
  <link rel="prev" href="https://example.com/2017/12/my-post/4/" />
  <link rel="next" href="https://example.com/2017/12/my-post/6/" />
```

`rel="prev"` and/or `rel="next"` will not be included if there is no previous and/or next page, respectively. If you don't want to set canonical to the single-view page, just set `seo_canonical` in your `_config.yml` to `false`.

### Manual

If, however, you include a header file that is also included by files that JPv2 may process, such as your home `index.html`, it would probably be better to do it this way:

```html
{{ paginator.seo.canonical }}
{% unless paginator %}
  <link rel="canonical" href="{{ site.url | append: page.url }}" />
{% endunless %}
{% if paginator.previous_page_path %}
  <link rel="prev" href="{{ site.url | append: paginator.previous_page_path }}" />
{% endif %}
{% if paginator.next_page_path %}
  <link rel="next" href="{{ site.url | append: paginator.next_page_path }}" />
{% endif %}
```
This way it works with JPv2, JPC, and with no paginator active.

What about `canonical` for JPv2-generated pages? Unless you have a "view-all" page that includes all your unpaginated posts and you want search engines to use that huge page as the primary search result, it is probably best to just not put a `canonical` link at all.


## TODO

1. Automatic Table of Contents
1. Automatic splitting based on headers (&lt;h2&gt;, etc.)


## Demo

See the [author's blog](https://ibrado.org/) for a (possible) demo.

## Contributing

1. Fork this project: [https://github.com/ibrado/jekyll-paginate-content/fork](https://github.com/ibrado/jekyll-paginate-content/fork)
1. Clone it (`git clone git://github.com/your_user_name/jekyll-paginate-content.git`)
1. `cd jekyll-paginate-content`
1. Create a new branch (e.g. `git checkout -b my-bug-fix`)
1. Make your changes
1. Commit your changes (`git commit -m "Bug fix"`)
1. Build it (`gem build jekyll-paginate-content.gemspec`)
1. Install and test it (`gem install ./jekyll-paginate-content-*.gem`)
1. Repeat from step 5 as necessary
1. Push the branch (`git push -u origin my-bug-fix`)
1. Create a Pull Request, making sure to select the proper branch, e.g. `my-bug-fix` (via https://github.com/your_user_name/jekyll-paginate-content)

Bug reports and pull requests are welcome on GitHub at [https://github.com/ibrado/jekyll-paginate-content](https://github.com/ibrado/jekyll-paginate-content). This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct
Everyone interacting in the Jekyll::Paginate::Content project's codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/[USERNAME]/jekyll-paginate-content/blob/master/CODE_OF_CONDUCT.md).

## Also by the Author

[Jekyll Stickyposts Plugin](https://github.com/ibrado/jekyll-stickyposts) - Move/pin posts tagged `sticky: true` before all others. Sorting on custom fields supported; collection and paginator friendly.

[Jekyll Tweetsert Plugin](https://github.com/ibrado/jekyll-tweetsert) - Turn tweets into Jekyll posts. Multiple timelines, filters, hashtags, automatic category/tags, and more!
