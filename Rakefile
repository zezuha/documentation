require 'yaml'
require 'open-uri'
require 'nokogiri'
require 'webpacker'
require 'active_support/all'

ENV['NODE_ENV'] ||= ENV['JEKYLL_ENV']
ENV['RAILS_ENV'] ||= ENV['JEKYLL_ENV']

# Fake a few Rails stuff to be able to use the webpacker gem
class Rails
  def self.root
    Pathname.new(Dir.pwd)
  end

  def self.env
    ENV['RAILS_ENV']
  end
end

Webpacker::Compiler.module_eval do
  def webpack_env
    env.merge("NODE_ENV" => @webpacker.env, "WEBPACKER_ASSET_HOST" => "")
  end
end

HOMEPAGE_URL = ENV['HOMEPAGE_URL'] || "https://scalingo.com/"

namespace :assets do
  desc "assets precompilation"
  task :precompile do
    Rake::Task["webpacker:compile"].execute
    exec("jekyll build")
  end
end

$stdout.sync = true

def ensure_log_goes_to_stdout
  old_logger = Webpacker.logger
  Webpacker.logger = ActiveSupport::Logger.new(STDOUT)
  yield
ensure
  Webpacker.logger = old_logger
end

namespace :webpacker do
  desc "Compile JavaScript packs using webpack for production with digests"
  task :compile do
    ensure_log_goes_to_stdout do
      if Webpacker.compile
        # Successful compilation!
      else
        # Failed compilation
        exit!
      end
    end
  end
end

namespace :fetch_from_homepage do
  task :all do
    Rake::Task["fetch_from_homepage:header"].execute
    Rake::Task["fetch_from_homepage:footer"].execute
  end

  task :header do
    doc = Nokogiri::HTML(open(HOMEPAGE_URL))
    toolbar = doc.css("header.mdc-toolbar").to_xhtml
    sidebar = doc.css("aside").to_xhtml
    template_path = "_includes/header.html"
    File.open(template_path, 'w+') { |file|
      file.write(toolbar)
      file.write(sidebar)
    }
  end
  task :footer do
    doc = Nokogiri::HTML(open(HOMEPAGE_URL))
    footer = doc.css("footer").to_xhtml
    template_path = "_includes/footer.html"
    File.open(template_path, 'w+') { |file|
      file.write(footer)
    }
  end
end