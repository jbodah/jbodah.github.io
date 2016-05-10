STOPWORDS = %w(i a about an are as at be by com for from how in is it of on or that the this to was what when where who will with the www)

namespace :post do
  desc "Creates a new blog post in _posts folder"
  task :new do
    STDOUT.print "Enter post name:\n> "
    post_name = STDIN.gets.chomp
    post_slug = post_name
      .gsub(/how to/i, "howto")
      .gsub(/[^a-zA-Z ]/,'').gsub(/ +/,' ')
      .split(" ")
      .map { |word| word.downcase }
      .keep_if { |word| !STOPWORDS.index(word) }
      .join("-")
    current_date = Time.now.getutc.strftime("%Y-%m-%d")
    file_path = "_posts/#{current_date}-#{post_slug}.md"
    File.open(file_path, 'w') do |f|
      f.write <<-EOF
---
layout: post
title: '#{post_name}'
date: #{Time.now}
comments: true
categories:
---
EOF
    end
    system("vim #{file_path}")
  end
end
