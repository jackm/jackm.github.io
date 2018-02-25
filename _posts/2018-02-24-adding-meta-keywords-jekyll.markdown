---
layout: post
title:  "Adding meta keywords to your Jekyll site"
date:   2018-02-24 23:41:00
keywords:
  - jekyll
  - theme
  - meta
  - metadata
  - keywords
  - seo
  - html meta tag
---

The default Jekyll theme (minima) does not include support for adding [meta keywords][html-meta-tag] to post pages.
The good news is that Jekyll and its plugins are meant to be customized the way you like it!

1. Copy the default `head.html` file from your Jekyll theme to the `_includes` folder within your Jekyll site directory

   * Jekyll will check this directory first and any files in there will override the ones included with the theme by default

   * To find where the Jekyll theme files are located in the filesystem, run `bundle show minima` (change "minima" if you're using a different gem)

   * Example:

     `cp /var/lib/gems/2.3.0/gems/minima-2.0.0/_includes/head.html /path/to/blog/_includes/`

   * If the includes folder does not yet exist in your site directory, you must create it first

1. Edit `head.html` and add the following snippit within the HTML _head_ tags:

   ```html
   {% raw %}{% if page.keywords %}
   <meta name="keywords" content="{{ page.keywords | join: ', ' | escape }}">
   {% endif %}{% endraw %}
   ```

1. Include the "keywords" variable within the front matter of a post

   ```yaml
   ---
   layout: post
   title:  "Best blog post ever made"
   date:   2029-01-01 16:20:00
   keywords:
     - some keyword
     - another keyword
     - get rich quick
   ---
   ```

That's it! Now whenver Jekyll generates html pages for posts, it will include the HTML meta tag in the HTML header and search engines will use the keywords to better rank your page on their search results.

[html-meta-tag]: https://www.w3schools.com/tags/tag_meta.asp