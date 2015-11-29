---
layout: post
title: "Optimize asset caching"
category: "web"
---
<img src="/assets/pagespeed.png" alt="book cover" style="float:right; height:300px; margin-left: 15px;">
I've been playing around with google [PageSpeed tools](https://developers.google.com/speed/pagespeed/?hl=en)  on this site, trying to make it nice to read on mobile connections an such, but mainly because I enjoy it.

One of the items PageSpeed complained about was asset caching. I had not configured the server that serves this blog to set `expires` headers on any of the content, so images were not being cached by the clients. 
<!-- more -->
When I opened up `nginx.conf` I was faced with a difficult decision. I initially thought I would just tell the clients to cache the images for as long as possible, but I also want to be able to change images on posts. It would suck for that one person that ever reads my rants to not get the updated image, so I settled on a value of 1 day, and figured that if anyone actually visits here twice in the same day, at least their second visit will be slightly faster (hooray!).

This kept bothering me though, so I decided to do something similar to what the Rails asset pipeline does with assets, that is, hash the contents and rename the file so that for example `kittens.jpg` becomes `0607a069c906a668f16efe91b3e9516f-kittens.jpg`. That way, I can set the `expires` header arbitrarily far in the future, and if I change any content clients will always get the newest version, as it will have a new name.

There are several plugins for [Jekyll](http://jekyllrb.com) to do this, for example [jekyll-asset-pipeline](https://github.com/matthodan/jekyll-asset-pipeline), but I don't really need or want any of the other features . Besides, it's always fun to roll your own, right?

So I ended up settling for the _simplest thing that could possibly work_™, a Rake task.
Here is the relevant portion of my `Rakefile`. As you can see, this is incredibly simplistic, it just hashes the file, creates a new filename, and then `gsub`s the filename in all posts.

{% highlight ruby %}
desc "Hash assets"
task :assets do
  abort("rake aborted: '#{CONFIG['static']}' directory not found.") unless FileTest.directory?(CONFIG['static'])
  Dir.foreach CONFIG['static'] do |file_name|
    if file_name.end_with?('jpg', 'png', 'gif')
      hashed_name = (hash_file File.join(CONFIG['static'], file_name)) + '-' + file_name
      puts "#{file_name} => #{hashed_name}"
      Dir.foreach CONFIG['posts'] do |post_name|
        post = File.join(CONFIG['posts'], post_name)
        replace_word_in_file(post, file_name, hashed_name) unless File.directory? post
      end
      File.rename(File.join(CONFIG['static'], file_name), File.join(CONFIG['static'], hashed_name))
    end
  end
end

def hash_file(filename)
  md5 = Digest::MD5.new
  md5 << File.read(filename)
  md5.hexdigest
end

def replace_word_in_file(file, old, new)
  text = File.read(file)
  File.open(file, 'w') {|f| f.puts text.gsub(old, new)}
end
{% endhighlight %}

I then run this rake task as part of the deployment script. This _seems_ to work, at least for now.
The code for this blog is on [Github](https://github.com/kthelgason/blag) so feel free to yell at me there if you think this is stupid, or even better: submit a PR and fix it! 

