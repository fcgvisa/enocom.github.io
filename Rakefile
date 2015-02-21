require "rake"

task :new_post, [:post_name] do |t, args|
  post_name = args[:post_name]

  post_date = Time.now.strftime("%Y-%m-%d")
  filename = "#{post_date}-#{post_name.downcase.tr(" ", "-")}.md"

  yml_preamble = <<-YML
---
layout: post
title: "#{post_name}"
categories: ruby
---
  YML

  File.open("_posts/#{filename}", "w") do |f|
    f.write(yml_preamble)
  end

  puts "Created post: #{filename}"
end
