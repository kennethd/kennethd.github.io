---
layout: post
title:  "Install Jekyll on Funtoo"
date:   2016-10-22 14:30:00 -4000
categories: jekyll
---

I am installing Jekyll locally on my funtoo instance, for development of a github.io site.

I begin by installing the required system packages:

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ sudo emerge dev-lang/ruby
kenneth@kennethd:/tmp/jekyll-test$ sudo emerge net-libs/nodejs
{% endhighlight %}

Next, I create a `~/.gemrc`, my main purpose is to enable the `--user-install` flag by default.

It absolutely mystifies me that the ruby community is ok with the way packages
are installed system-wide by default.

Other configuration options are documented [here](https://jekyllrb.com/docs/configuration/)

{% highlight ruby %}
benchmark: false
verbose: true
# Enable/disable automatic updating of repository metadata
update_sources: true
# Print backtrace when RubyGems encounters an error
backtrace: true
# do not generate local documentation for every installed gem
# install packages into my $HOME
# rewrite gems to use `/usr/bin/env ruby` if not already using env shebang
# suggest alternates when gems are not found
gem: --no-ri --no-rdoc --no-document --suggestions --user-install --env-shebang
{% endhighlight %}

When installing gems to your $HOME, you have to add the directory they are
installed into to your $PATH.  For bash, add something like this to ~/.bashrc
and then source the file in your terminal (`. ~/.bashrc`):

{% highlight bash %}
if [[ ":$PATH:" != *":$HOME/.gem/ruby/2.2.0/bin:"* ]] && [[  -d "$HOME/.gem/ruby/2.2.0/bin" ]]
then
    export PATH="$PATH:$HOME/.gem/ruby/2.2.0/bin"
fi
{% endhighlight %}

Install some gems:

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ gem install jekyll
kenneth@kennethd:/tmp/jekyll-test$ gem install jekyll-docs
kenneth@kennethd:/tmp/jekyll-test$ gem install pygments.rb
kenneth@kennethd:/tmp/jekyll-test$ gem install bundler
{% endhighlight %}

Create a new site:

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ jekyll new .
kenneth@kennethd:/tmp/jekyll-test$ ls -l
total 24
-rw-r--r-- 1 kenneth kenneth  810 Oct 21 17:55 Gemfile
-rw-r--r-- 1 kenneth kenneth 3542 Oct 21 17:59 Gemfile.lock
-rw-r--r-- 1 kenneth kenneth 1408 Oct 21 17:53 _config.yml
drwxr-xr-x 2 kenneth kenneth 4096 Oct 21 18:51 _posts
-rw-r--r-- 1 kenneth kenneth  524 Oct 21 17:53 about.md
-rw-r--r-- 1 kenneth kenneth  213 Oct 21 17:53 index.md
{% endhighlight %}

You will want to edit the auto-generated `_config.yml` to add your own name,
etc.  If you noticed, I had also installed the `pygments.rb` gem above, to use
it on my site I have to add a line to `_config.yml` (but see
[below](#first-problems) before following along):

{% highlight ruby %}
highlighter: pygments
{% endhighlight %}

And then to allow the site to build with that reference in place, add to the
`Gemfile`:

{% highlight ruby %}
gem "pygments.rb", "~> 0.6"
{% endhighlight %}

The `~> 0.6` version specifier means: upgrade to most recent sub-version of
0.6, but don't automatically upgrade to 0.7 or above.  I chose it because I
already have two versions of pygments.rb on my system (probably due to
dependencies):

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ gem list --local | grep pygments
pygments.rb (0.6.3, 0.6.1)
{% endhighlight %}

At this point we should be able to launch a server and navigate to
http://127.0.0.1:4000/

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ bundle install
kenneth@kennethd:/tmp/jekyll-test$ bundle exec jekyll serve
Configuration file: /tmp/jekyll-test/_config.yml
Configuration file: /tmp/jekyll-test/_config.yml
            Source: /tmp/jekyll-test
       Destination: /tmp/jekyll-test/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.286 seconds.
 Auto-regeneration: enabled for '/tmp/jekyll-test'
Configuration file: /tmp/jekyll-test/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
{% endhighlight %}

![Welcome to Jekyll!](/assets/welcome-to-jekyll.png)

Ok!  Let's make the first commit:

{% highlight shell-session %}
kenneth@kennethd:/tmp/jekyll-test$ git init
kenneth@kennethd:/tmp/jekyll-test$ git config user.name "Kenneth Dombrowski"
kenneth@kennethd:/tmp/jekyll-test$ git config user.email kenneth@ylayali.net
kenneth@kennethd:/tmp/jekyll-test$ git remote add origin git@github.com:kennethd/kennethd.github.io.git
kenneth@kennethd:/tmp/jekyll-test$ git add .gitignore Gemfile _config.yml _posts/ about.md assets/ index.md
kenneth@kennethd:/tmp/jekyll-test$ git commit -m "install jekyll"
kenneth@kennethd:/tmp/jekyll-test$ git push origin master
{% endhighlight %}

# First problems

All of this worked perfectly on my funtoo instance, but as I pushed to
github.io, a few problems cropped up:

First, the `Gemfile` contained some unsupported lines, for `github-pages`, I
reduced it to the following:

{% highlight ruby %}
source "https://rubygems.org"
gem "minima", "~> 2.0"
gem "github-pages", group: :jekyll_plugins
{% endhighlight %}

Second, `pygments` is not supported by github.io, you have to use `rouge`.  In
the end, my `_config.yml` looked something like this:

{% highlight yaml %}
title: kennethd.github.io
email: kenneth@ylayali.net
author: Kenneth Dombrowski
description: >
    Notes about Linux, Programming, Web Development, and Data Sciences
baseurl: ""
url: "https://kennethd.github.io"
twitter_username: kennnethd
github_username: kennethd

markdown: kramdown
theme: minima
highlighter: rouge
gems:
  - jekyll-feed
exclude:
  - Gemfile
  - Gemfile.lock
{% endhighlight %}

Third, the files required by use of the `minima` theme were not discovered in
the github.io environment.  In the end I just copied all of the files from the
theme into my project's `_layouts`, `_includes`, and `_sass` directories,
which is an entirely unsatisfactory solution, and something I will have to
revisit.

