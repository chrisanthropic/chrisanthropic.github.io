---
layout: post
title: "Creating Gem Based Themes For Jekyll"
sitemap:
  priority: 0.5
  exclude: 'no'
---

# How I Built Starving Artist Jekyll Theme
Jekyll 3.2 introduced the ability for theme creators to easily generate Ruby Gems from Jekyll themes and for users to easily use and change between themes. 

Here's how I built my most recent theme - Starving Artist. ( [Demo](http://chrisanthropic.github.io/starving-artist-jekyll-theme/) \| [Source](https://github.com/chrisanthropic/starving-artist-jekyll-theme) )

### Start a New Site
Jekyll requires Ruby so make sure Ruby is installed before you begin.

- Install Jekyll and Bundler
  - `gem install bundler`
- Create a New Site
  - `jekyll new mytheme`
- Move into that directory
  - `cd mytheme`
- Install Required Gems
  - `bundle install`
- Verify
  - Run `jekyll serve`
  - Browse to [http://localhost:4000](http://localhost:4000)

## Disable the Default Theme
- Delete the line `gem "minima"`
- Tell Jekyll not to use a Theme
  - Open `_config.yml` and delete the line `theme: minima`
- Do NOT import Any Theme css 
  - Open your `css/style.css` and delete the line `@import "minima;"`

## Create Your Theme
- Create the `_layouts`, `_includes`, and `_sass` directories.
  - `jekyll new-theme YOUR-THEME-NAME`
  - Copy and paste all of the contents of the 'YOUR-THEME-NAME' directory into the root of your Jekyll site.
  - Delete the (now empty) YOUR-THEME-NAME directory.
- Design your theme.
  - I like to create a Demo site that documents the theme itself and shows all of its features.
- Verify
  - Run `jekyll serve`
  - Browse to [http://localhost:4000](http://localhost:4000)

## Create Your Gem
Once you're happy with your theme, or you're simply ready to test it, then it's time to create your gem.

- open the `YOUR-THEME-NAME.gemspec` file and edit the spec.files command (at a minimum) since the current version creates incomplete and sometimes empty Gems.  Replace this line:

  ```
  spec.files = `git ls-files -z`.split("\x0").select { |f| 
    f.match(%r{^(_layouts|_includes|_sass|LICENSE|README)/i}) }
  ```

  With this line:

  ```
  spec.files = `git ls-files -z`.split("\x0").select do |f|
    f.match(%r{^(_(includes|layouts|sass)/|(LICENSE|README)((\.(txt|md|markdown)|$)))}i)
  end
  ```

- Run `gem build YOUR-THEME-NAME.gemspec`

## Test Your Gem
Testing your Gem based theme is simple. Create a brand new Jekyll site and then tell it to use your local Gem file. 

### Start a New Site
- Install Jekyll and Bundler
  - `gem install bundler`
- Create a New Site
  - `jekyl new mysite`
- Move into that directory
  - `cd mysite`
- Install Required Gems
  - `bundle install`
- Verify
  - Run `jekyll serve`
  - Browse to [http://localhost:4000](http://localhost:4000)
- Set Your Theme
  - Open `Gemfile` and add the line `gem "YOUR-THEME", :path => ""`
    - By default it checks Rubygems.org for the Gem and downloads it, but with `:path => ""` we're telling it to use the local version of the gem.
  - Run `bundle install`
- Tell Jekyll to use Your Theme
  - Open `_config.yml` and add the line `theme: YOUR-THEME` to this:
- Import the Theme CSS
  - Open your `css/style.css` and add the line `@import "YOUR-THEME";`
    - **IMPORTANT** the name of the file you `@import` is whatever the name of your main `.scss` file in your theme's `_sass` directory.

## Push Your Gem
Once you are satisfied with your gem you can optionally share it with the world. Doing so is simply and only requires you to sign up for a free account with Rubygems.org.

- Run `gem push YOUR-THEME.gem`
- Enter your Rubygems.org email and password when prompted.

That's it, now your Gem is available to the world!

If you find out that you released the Gem a bit early and want to pull it back down simply issue the following command:
- `gem yank YOUR-THEM -v VERSION-NUMBER`
