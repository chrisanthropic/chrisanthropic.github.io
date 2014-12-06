---
layout: post
title: "Jekyll With Bourbon and Neat"
date: 2014-12-05T15:55:47-08:00
sitemap:
  lastmod: 2014-12-05T15:55:47-08:00
  priority: 0.5
  changefreq: monthly
  exclude: 'no'
---

Until recently I've been using Zurb Foundation to build my sites but I rarely use more than a third of what's avaialble. In the spirit of trimming the fat and simplifying I started looking for other options.

For various reasons I decided to try out a 'semantic grid' and after some research figured I'd give [Bourbon](http://bourbon.io/) and [Neat](http://neat.bourbon.io/) a try.

Below are the steps I took to install Bourbon and Neat into my Jekyll project. I haven't done anything with them yet, but they're there waiting for me.

### Bourbon

* add bourbon to gemfile
  `gem 'bourbon'`
* Install the gem
  `bundle install --binstubs --path=vendor`
* Install bourbon
  `bundle exec bourbon install --path _sass/vendor/`
* Import bourbon
  add `@import "vendor/bourbon/bourbon";` to /css/main.scss

### Neat
* add neat to gemfile
  `gem 'neat'`
* Install the gem
  `bundle install --binstubs --path=vendor`
* Install neat
  `bundle exec neat install`
* Move the 'neat' directory to /css/vendor/
  `mv neat/ _sass/vendor/neat`
* Import neat
  add `@import "vendor/neat/neat";` to /css/main.scss