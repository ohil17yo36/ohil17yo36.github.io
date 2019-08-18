---
layout: post
title:  "Network Architecture Design"
date:   2019-08-18 13:26:47 -0700
categories: network architecture
---
# High level Network Architecture design

Imagine you are tasked to design the `network architecture` of a firm. 

### First thoughts
1. Should all the devices be exposed to the internet ?
2. When some devices are infiltrated by an attack, should there be a way to simply turn off all connections from the rogue devices to other devices
3. Categorize devices into layers/zones based on how much attack surface is exposed? 


### Functional requirements
1. Add multiple layers for defense in depth
2. Only expose the outermost layer to the external internet

### Architectural choices
1. Separate each layer by a network layer 3 firewall
2. Blacklist all connections between device across layers by default
3. Have a mechanism to whitelist connections as need arises.

<!--
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
-->