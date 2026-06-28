source "https://rubygems.org"

# Use the same toolchain as GitHub Pages so local previews match production.
gem "github-pages", group: :jekyll_plugins
gem "webrick", "~> 1.8"

# Windows / JRuby timezone support
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end
gem "wdm", "~> 0.1", :platforms => [:mingw, :x64_mingw, :mswin]
