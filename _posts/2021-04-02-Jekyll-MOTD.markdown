---
layout: post
title:  "Use Jekyll for a Linux MOTD"
date:   2021-04-02 09:00:00 -0700
categories: blog
excerpt_separator: <!--more-->
---

For a bit, I've been wanting to find a way to use my most recent jekyll posts as some kind of MOTD. I'm working with micro-fiction, but this could work for anything, really. It's also, really, really easy.

<!--more-->
### Make Jekyll Serve Raw Text

The first thing you have to do is make jekyll serve raw text, that way you don't get all the markdown/html markup, like paragraph tags everywhere. To do this, create a new layout file. Mine is called raw. It just gets rid of all of the automated p tags and gives the page to you raw. It should have one line in it like this:

{% highlight md %}{%raw%}
{{ content | remove: "<p>" | remove: "</p>"  }}{%endraw%}
{% endhighlight %}


### Create an MOTD Endpoint
Then create a MOTD style endpoint for it. I'm using latestmicrofiction.md. Just put it in the project root. It should point to the new raw layout and have the permalink you want to use to get the MOTD. I'm pulling from a subset of categories to get my microfiction, but you could have "announcements" or "motd" or something similar. Up to you. My endpoint looks like this:

{% highlight md %}{%raw%}
---
layout: raw
title: Latest Microfiction
permalink: /latestmicrofiction/
---

{% for post in site.categories["microfiction"] limit:1 %}
{{ post.content | strip_html }}
{% endfor %}

-- Stuart Templeton (https://www.stuarttempleton.com)

{%endraw%}
{% endhighlight %}

### Use CURL to get your MOTD

Now you can pull the most recent entry directly from the command line with something like curl.

{%highlight bash%}
~$ curl https://www.stuarttempleton.com/latestmicrofiction/
{% endhighlight %}

Put that in your /etc/motd or /etc/profile or ~/.bashrc or whatever you like that works best for your system. 

{%highlight bash%}
~$ echo "curl https://www.stuarttempleton.com/latestmicrofiction/" >> ~/.bash_profile 
{%endhighlight%}

