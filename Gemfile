# frozen_string_literal: true

source "https://rubygems.org"

# Use the latest Chirpy Jekyll theme
gem "jekyll-theme-chirpy", "~> 7.4"

# HTML validation during CI
group :test do
  gem "html-proofer", "~> 5.0"
end

# Windows & JRuby compatibility
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# File watcher for Windows
gem "wdm", "~> 0.2.0", platforms: [:mingw, :x64_mingw, :mswin]

# Maintain compatibility for JRuby HTTP parsing
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]

# Fix musl-based Alpine builds (used in Docker, etc.)
if RUBY_PLATFORM =~ /linux-musl/
  gem "jekyll-sass-converter", "~> 3.0"
end

# Optional: pin Jekyll version to match Chirpy’s tested baseline
gem "jekyll", "~> 4.3"