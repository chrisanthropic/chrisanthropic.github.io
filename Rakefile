require "reduce"

##############
#   Build    #
##############

# Generate the site
# Minify, optimize, and compress

desc "build the site"
task :build do
  sh "bundle exec jekyll build"
  Rake::Task[:minify].execute
  Rake::Task[:gzip_html].execute
end

##############
#   Deploy   #
##############

# Deploy the site
# Ping / Notify after site is deployed

desc "deploy the site"
task :deploy do
  sh "bundle exec s3_website push"
  Rake::Task[:notify].execute
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