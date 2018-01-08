---
layout: post
title: "Jekyll Themed Category Pages Without Plugins"
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---

I don't like using Jekyll plugins unless absolutely necessarry, I'd much rather use Jekyll's built in features.

I like using post Categories to organize things and I really wanted a custom category archive page. Luckily, I found this [post](http://primalivet.com/2013/11/simple-category-pages-with-vanilla-jekyll/) that described how to do it without any plugins.

Here's what I did:

* Create a new layout called `category.html` with the following:

```
{% raw %}
---
layout: default
---
<h1>Simple Category Pages with vanilla Jekyll</h1>

{% unless page.content == '' %}
  <p>{{ page.content }}</p>
{% endunless %}

{% for post in site.categories.[page.category] %}
  <h2><a href=""></a></h2>
  <p></p>
{% endfor %}
{% endraw %}
```

  * The key here is that it uses the `default` layout so it will pull our content like any other page/post. 
  * This bit `{% raw %} {% for post in site.categories.[page.category] %} {% endraw %}` will look for yml frontmatter of `category` and loop it.

* Create a new folder in my root directory, titled the name of my category. In this example the category will be "sample".
* Create a new `index.html` file inside that folder, with the following content:

```
---
layout: category
title: Sample Category
category: sample
---
```

* It's also a good idea to create subdirectories in _posts for each category, so in this case you'd have `_posts/sample/`
* Now just make sure to add `categories: sample` to your posts yml frontmatter

So, you can create a custom index.html for each category, have it use the category layout, and that will limit the displayed posts to any with the category you define in the frontmatter of the index.html. 

All without plugins and using standard Jekyll features.

Sweet.
