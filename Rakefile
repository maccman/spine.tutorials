require "rdiscount"
require "mustache"

def generate(source)
  Mustache.render(
    File.read("site/template.ms"), 
    :content => RDiscount.new(source).to_html
  )
end

task :generate do
  Dir["tutorials/*.md"].each do |path|
    File.open(File.basename(path, ".md") + ".html", "w+") do |f|
      f.write generate(File.read(path))
    end
  end
end

task :default => :generate