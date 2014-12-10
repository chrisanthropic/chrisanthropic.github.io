---
layout: post
title: "Responsive Navigation Without Javascript"
date: 2014-12-09T16:54:34-08:00
sitemap:
  lastmod: 2014-12-09T16:54:34-08:00
  priority: 0.5
  changefreq: monthly
  exclude: 'no'
---

My goal is to have the core elements of this site available without the use of any Javascript. Now that we've got a responsive grid, how about a responsive drop-down navigation menu?

I found a project here that documents exactly that, so we'll be using it as a base.
  * NOTE - the recent Github code of that project now includes JS but it isn't required, I used an [old codeboase](https://github.com/catalinred/Animenu/archive/9fba2e18ecad6d37cf6024b3cdf0d2a64d7d263f.zip) from before the commit that added JS.

Here's what I did:

* create _includes/navigation.html
* paste default nav code

```
<nav class="animenu">
  <input type="checkbox" id="button">
  <label for="button" onclick>Menu</label>
    <ul>
      <li>
	<a href="{{ site_url }}">Home</a>
      </li>
      <li>
	<a href="{{ site_url }}/blog">Blog</a>
	  <ul>
	    <li><a href="">Sub Item 1</a></li>
	    <li><a href="">Sub Item 2</a></li>
	    <li><a href="">Sub Item 3</a></li>
	  </ul>
      </li>
            <li>
	<a href="{{ site_url }}/misc">Misc</a>
	  <ul>
	    <li><a href="">Sub Item 1</a></li>
	    <li><a href="">Sub Item 2</a></li>
	    <li><a href="">Sub Item 3</a></li>
	  </ul>
      </li>
    </ul>
</nav>
```
* Download [the code](https://github.com/catalinred/Animenu/archive/9fba2e18ecad6d37cf6024b3cdf0d2a64d7d263f.zip) and unzip it
* add contents of animenu/sass to your Jekyll site at _sass/animenu
  * don't forget to add `@import "animenu/style";` to your _sass/main.scss
* edit default layout to use _includes/navigation.html
```
{% raw %} {% include navigation.html %} {% endraw %}
```
* run `bundle exec rake build`

You now have a good looking responsive navigation menu that supports submenus and is Javascript free!
