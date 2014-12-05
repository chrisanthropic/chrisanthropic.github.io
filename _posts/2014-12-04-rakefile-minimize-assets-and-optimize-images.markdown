---
layout: post
title: "Rakefile: Minimize Assets and Optimize Images"
date: 2014-12-04T16:38:13-08:00
sitemap:
  lastmod: 2014-12-04T16:38:13-08:00
  priority: 0.5
  changefreq: monthly
  exclude: 'no'
---

I used Octopress to build my wife's webcomic site [ShamseeComic](http://www.shamseecomic.com) and learning how to optimize everything has been fun.

Now that I'm starting on this site from scratch (with a better understanding of Jekyll) I figured I'd revisit my old method and update it.

Below are the steps I took to add a few optimization tasks to my blank Rakefile. Here's the tasks:

* Minify all HTML and CSS when the site is built
* Optimize all images when the site is built
* Notify Google and Yahoo when the site is deployed
  * This one took me a bit, but I was trying to reproduce the 'Ping Service' that's included in Wordpress

#### Gemfile

* First I added the `reduce` gem to my Gemfile since it's used for the minification and optimization
  `gem "reduce"`
* Next I created a Rakefile with the following:

```
  require "reduce"

  ##############
  #   Build    #
  ##############

  # Generate the site
  # Minify, optimize, and compress

  desc "build the site"
  task :build do
    system "bundle exec jekyll build"
    Rake::Task[:minify].execute
  end

  ##############
  #   Deploy   #
  ##############

  # Deploy the site
  # Ping / Notify after site is deployed

  desc "deploy the site"
  task :deploy do
    system "bundle exec s3_website push"
    system "bundle exec rake notify"
  end

  ##############
  #   Reduce   #
  ##############

  # Minifies html/css/js and also optimizes images
  # Courtesy of https://github.com/pacbard/blog/blob/master/_rake/minify.rake

  desc "Minify _site/"
  task :minify do
    puts "\n## Compressing static assets"
    original = 0.0
    compressed = 0
    Dir.glob("_site/**/*.*") do |file|
      case File.extname(file)
	when ".css", ".gif", ".html", ".jpg", ".jpeg", ".js", ".png", ".xml"
	  puts "Processing: #{file}"
	  original += File.size(file).to_f
	  min = Reduce.reduce(file)
	  File.open(file, "w") do |f|
	    f.write(min)
	  end
	  compressed += File.size(file)
	else
	  puts "Skipping: #{file}"
	end
    end
    puts "Total compression %0.2f\%" % (((original-compressed)/original)*100)
  end

  ##############
  #   Notify   #
  ##############

  # Ping Google and Yahoo to let them know you updated your site

  site = "www.chrisanthropic.com"

  desc 'Notify Google of the new sitemap'
  task :sitemapgoogle do
    begin
      require 'net/http'
      require 'uri'
      puts '* Pinging Google about our sitemap'
      Net::HTTP.get('www.google.com', '/webmasters/tools/ping?sitemap=' + URI.escape('#{site}/sitemap.xml'))
    rescue LoadError
      puts '! Could not ping Google about our sitemap, because Net::HTTP or URI could not be found.'
    end
  end

  desc 'Notify Bing of the new sitemap'
  task :sitemapbing do
    begin
      require 'net/http'
      require 'uri'
      puts '* Pinging Bing about our sitemap'
      Net::HTTP.get('www.bing.com', '/webmaster/ping.aspx?siteMap=' + URI.escape('#{site}/sitemap.xml'))
    rescue LoadError
      puts '! Could not ping Bing about our sitemap, because Net::HTTP or URI could not be found.'
    end
  end

  desc "Notify various services about new content"
  task :notify => [:sitemapgoogle, :sitemapbing] do
  end
```
Now I build my site with the `bundle exec rake build` command and it automatically builds my site, minifies the assets, and optimizes all images.

When I deploy my site with `bundle exec rake deploy` it deploys it to S3 and then notifies Google and Bing about my updated sitemap.xml file.

Pretty cool.