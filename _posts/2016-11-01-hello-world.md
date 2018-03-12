---
layout: post
title: "Welcome to IO!"
description: "Find out what technologies where used to build this blog."
---

This is my personal blog where I write about stuff I have worked/tried in software. I thought it
would be a good idea to start explaining what technologies I use to power this blog.

## Blog Engine

I use [Jekyll](https://jekyllrb.com/) to run the blog. Jekyll is a static site generator, it renders your pages based on different static files that you define in your repository. Pages can be written in multiple markups like Markdown, Liquid or HTML to name a few.

There is an awesome community around Jekyll, lots of static sites are powered by this small engine. Thanks to this community, tons of themes are available for free. This blog is based on a particular theme that I really liked, [Chalk](https://github.com/nielsenramon/chalk).

@nielsenramon has done a great job with his theme, it is really easy to setup and start writing! Here I will show the different steps I did in order to setup and deploy IO site with [Github Pages](https://pages.github.com/) (more on this later).

First we need to clone the original repository:

{% highlight zsh %}
git clone github.com/nielsenramon/chalk io
{% endhighlight %}

I removed the default data (posts, images) and update the favicon. I also deleted the circle.yml file, won't be running CI for the moment. Now we can change the default configuration. The file `_config.yml` contains the Jekyll mandatory settings and some custom settings for Chalk.

We are ready to start writing. I decided to write the posts in Markdown format, it is really easy to use and super powerful. My first post is the one you are reading :)

To preview your site locally you need to install all the dependencies. Chalk comes with a very handy setup
script at `bin/setup`. In order to run the script you will need to first install ruby and npm. I suggest
installing ruby with [rvm](https://rvm.io/) and npm with [brew](http://brew.sh/).

{% highlight zsh %}
# Install rvm
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash

# Install ruby
rvm install ruby

# Install brew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# Install node and npm
brew install node
{% endhighlight %}

Now we can proceed and install the rest of the stuff by simply running:

{% highlight zsh %}
bin/setup
{% endhighlight %}

Done! We can now start the Jekyll server with this command:

{% highlight zsh %}
bundle exec jekyll serve
{% endhighlight %}

## Deploy

We are now ready to deploy our static blog. Github provides a very convenient tool for hosting static sites
powered by Jekyll, Github Pages (gh-pages). It is really easy to deploy a site with gh-pages, you just need to push your code into a Github repository and setup some minimal configurations (settings tab) on the repository.

{% include image.html path="2016-11-01-hello-world/gh-pages-settings.png" path-detail="2016-11-01-hello-world/gh-pages-settings.png" alt="gh-pages settings" %}

Chalk provides a useful deploy script to build and push the site. It will deploy the builded Jekyll site to a branch named gh-pages. Don't forget to configure the correct branch name like in the previous image. To run the deploy script:

{% highlight zsh %}
bin/deploy
{% endhighlight %}

## Ready

Now the site should be correctly deployed in the Github Pages service.

I hope you have found this useful.
