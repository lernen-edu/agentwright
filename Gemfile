# Gemfile for local preview of the Agentwright site.
# GitHub Pages itself uses its own bundle on the server side — this file is
# only used when running `bundle exec jekyll serve` locally to preview the
# site before pushing.
#
# Usage:
#   gem install bundler   # one-time
#   bundle install
#   bundle exec jekyll serve --livereload
# Then open http://localhost:4000/agentwright/

source "https://rubygems.org"

# GitHub Pages gem pins the same Jekyll + plugins versions GitHub uses.
gem "github-pages", group: :jekyll_plugins

# Remote theme support (Just the Docs is loaded via remote_theme in _config.yml)
gem "jekyll-remote-theme"

# Front-matter parsing helpers
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-seo-tag"
  gem "jekyll-relative-links"
end

# Windows / JRuby / macOS — keep this small set for portability.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
