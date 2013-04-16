# usage: rake post title="a title"

require 'time'

desc "create a new post in _posts directory"
task :post do
  title = ENV['title'] || 'new post'
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  date = Time.now.strftime('%Y-%m-%d')
  filename = File.join('_posts', "#{date}-#{slug}.md")

  if File.exists? filename
    abort "rake aborted! #{filename} already exists"
  end

  puts "creating new post #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title}\""
    post.puts "---"
  end
end

task default: :post
