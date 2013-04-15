# usage: rake post title="a title" date="2012-12-25"

require 'time'

desc "create a new post in _posts directory"
task :post do
  title = ENV['title'] || 'new post'
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  date = Time.now.strftime('%Y-%m-%d')
  filename = File.join('_posts', "#{date}-#{slug}.md")

  if File.exists? filename
    abort 'rake aborted!' if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  puts "creating new post #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: default"
    post.puts "title: \"#{title}\""
    post.puts "---"
  end
end

task default: :post
