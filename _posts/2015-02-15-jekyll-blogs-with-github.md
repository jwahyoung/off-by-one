---
uid: d874f6ad-b71b-4fbc-8922-9e680f306b0a
layout: post
title: Creating a Jekyll Blog on Github
description: "When I decided to launch {{site.title}}, I had to make some choices; how would I host the blog, and what backend would I use? Through GitHub pages, I was introduced to Jekyll, and so I've decided to share my experience with my setup."
date: 2015-02-15
tags: jekyll
published: true
authorid: jahyoung
author: Jedd Ahyoung
---

When I decided to launch **{{site.title}}**, I had to make some choices; how would I host the blog, and what backend would I use? Through GitHub pages, I was introduced to Jekyll, and so I've decided to share my experience with my setup.

## The Logistics

I knew, starting out, that I wanted my subdomain `blog.jedd-ahyoung.com` to point to **{{site.title}}**. GitHub allows you to host a user page by creating a repository called `[[your-username]].github.io`, and it serves your site for a given domain if you add a CNAME file to your repository root. However, you can only attach one domain to a GitHub pages repository, and I'd planned to move my main site there.

So, what was the solution? I decided to create a standalone repo for the blog, with `gh-pages` as the main branch. I could keep a CNAME file in the root with a single domain (my subdomain), while freeing the CNAME root in `jedd-ahyoung.github.io` for my primary domain. Using this method, one should be able to host as many subdomains as they want on Github, with each subdomain pointing to a different GitHub repository.

## Setting up the environment

GitHub provides a great [Getting Started tutorial](https://pages.github.com) for GitHub Pages, and Jekyll provides [good documentation](http://jekyllrb.com/docs/home/) for its offerings as well; as such, I won't repeat all of that here. If you decide to go this route, following GitHub's tutorial should get you going.

Although I already had Ruby, Bundler, and Jekyll installed, I wanted to grab the GitHub Pages gem to sync my development environment with GitHub. I ran into this:

	Jedds-MacBook-Pro:jedd-ahyoung.github.io jahyoung$ bundle install
	Resolving dependencies...
	Your Gemfile has no gem server sources. If you need gems that are not already on your machine, add a line like this to your
	Gemfile:
	source 'https://rubygems.org'
	Could not find gem 'github-pages (>= 0) ruby' in the gems available on this machine.

I fixed this by adding the line `source: 'https://rubygems.org'` to the top of my Gemfile. Not a showstopper, but if you run into this, that's the fix.

## Creating and testing the site

I'd settled on [Poole](https://github.com/poole/poole) as a base for my blogging site, and was able to get up and running pretty quickly. I did, however, run into problems with Jekyll's baseUrl option in `_config.yml`. While Poole suggests making baseUrl an absolute URL, it looked as though it was meant to be relative...

BaseUrl caused all sorts of problems during development. If I was able to get the configuration working locally, viewing it on GitHub would result in a broken page due to malformed asset URLs. After a few frustrated commits and pushes, I found that this worked:

	## Setup
	url: http://jedd-ahyoung.github.com
	baseUrl: /off-by-one/

*NOTE: Using a relative baseUrl configuration requires using an empty baseUrl argument when running Jekyll - `bundle exec jekyll serve --baseUrl=`. Without this, assets won't load correctly locally.*

I suppose I probably could have avoided this if I'd just set baseUrl to the absolute path of my repository, but then I don't know what the URL key would be for. As it turns out, it didn't matter at all once I accessed the site through the CNAME (blog.jedd-ahyoung.com), as I ran into the same issue and had to reconfigure everything. At the moment, it looks as though there isn't a good way to configure a project page to work correctly from multiple URLs.

Because of this, I found myself having to set up a redirect from the github.io subdomain to my personal subdomain. GitHub hosting doesn't support HTTP redirects, so it was necessary to perform a Javascript redirection by checking `window.location.hostname`. This seems to be the de-facto standard for GitHub redirects at the moment.

## Final Thoughts

I really wish that GitHub had better support for HTTP redirections through `_config.yml` - perhaps a redirection key/value that could generate webserver configuration to perform a redirect. Additionally, baseUrl vs. url in Jekyll's configuration doesn't seem very intuitive. However, publishing a blog to GitHub, overall, wasn't too bad thanks to the available documentation and Poole (which rocks).
