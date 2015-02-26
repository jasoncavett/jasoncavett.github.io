source 'https://rubygems.org'

# For Theme
gem 'jekyll'
gem 'jekyll-sitemap'
gem 'octopress', '~> 3.0.0.rc.12'
gem 'rouge'

# For GitHub Pages
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
