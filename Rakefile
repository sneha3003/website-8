require 'middleman-gh-pages'

require "bundler/setup"
require 'yaml'

def git_initialize(repository)
  unless File.exist?(".git")
    system "git init"
    system "git remote add origin https://github.com/emberjs/#{repository}.git"
  end
end

def git_update
  system "git fetch origin"
  system "git reset --hard origin/master"
  # Remove all files so we don't accidentally keep old stuff
  # These will be regenerated by the build process
  system "rm `git ls-files`"
end

def aura_path
  File.expand_path(ENV['EMBER_PATH'] || File.expand_path("../../aura", __FILE__))
end

def generate_docs
  print "Generating docs data from #{aura_path}... "

  sha = nil

  Dir.chdir(aura_path) do
    # returns either `tag` or `tag-numcommits-gSHA`
    describe = `git describe --tags --always`.strip
    sha = describe =~ /-g(.+)/ ? $1 : describe
    system("grunt yuidoc")
  end

  # JSON is valid YAML
  data = YAML.load_file(File.join(aura_path, "docs/data.json"))
  data["project"]["sha"] = sha
  File.open(File.expand_path("../data/api.yml", __FILE__), "w") do |f|
    YAML.dump(data, f)
  end

  puts "Built #{sha}"
end

# def build
#   system "middleman build"
# end

task :aura_path do 
  puts aura_path
end

# desc "Generate API Docs"
task :generate_docs do
  generate_docs
end

# desc "Build the website"
# task :build => :generate_docs do
#   build
# end

desc "Preview"
task :preview do
  require 'listen'

  generate_docs

  paths = Dir.glob(File.join(aura_path, "packages/*/lib"))
  listener = Listen.to(*paths, :filter => /\.js$/)
  listener.change { generate_docs }
  listener.start(false)

  system "middleman server"
end

