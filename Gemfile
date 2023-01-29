source "https://gems.ruby-china.com/"

gem "jekyll", ">= 3.8.6", "< 5.0"

gem "font-awesome-sass", "~> 6.2.1"

gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-redirect-from"
  gem "jekyll-seo-tag", "~> 2.6.1"
  gem "jekyll-archives"
  gem "jekyll-sitemap"
end

gem 'rouge'

gem "webrick", "~> 1.7"

group :test do
  gem "html-proofer", "~> 3.16.0"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?

