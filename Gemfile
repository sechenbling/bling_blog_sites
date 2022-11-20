source "https://gems.ruby-china.com/"

gem "jekyll", ">= 3.8.6", "< 5.0"

gem "github-pages", group: :jekyll_plugins
gem "webrick", "~> 1.7"
# plugins
# group :jekyll_plugins do
#   gem "jekyll-paginate"
#   gem "jekyll-redirect-from"
#   gem "jekyll-seo-tag", "~> 2.6.1"
#   gem "jekyll-archives"
#   gem "jekyll-sitemap"
#   gem "jekyll-feed", "~> 0.12"
# end

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

#
# # gem "jekyll", "~> 4.3.1"
# # gem "minima", "~> 2.5"
#
# # group :jekyll_plugins do
# #   gem "jekyll-feed", "~> 0.12"
#
#
# # Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# # and associated library.
# platforms :mingw, :x64_mingw, :mswin, :jruby do
#   gem "tzinfo", ">= 1", "< 3"
#   gem "tzinfo-data"
# end
#
# # Performance-booster for watching directories on Windows
# gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
#
# # Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# # do not have a Java counterpart.
# gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
#
# gem "github-pages", group: :jekyll_plugins
#
# gem "webrick", "~> 1.7"