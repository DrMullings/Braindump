---
layout: post
title:  "Basic setup of jekyll on opensuse"
date:   2021-03-24 15:35:00 +0100
categories: tutorials
---

# Motivation

I wanted to set up this blog to have a place to store various snippets and code

# Prerequisites

My system runs openSUSE Leap 15.2

Following packages needed to be installed: `ruby-devel gcc-c++`

- `sudo gem install jekyll bundler`
- `mkdir jekyllDir`
- `jekyll new jekyllDir`
- `bundle exec jekyll serve --livereload`

In `_config.yml`:
- adapt title and description
- add URL and baseurl from github

In `Gemfile`:
- follow instructions in comments to configure for GH pages

The default site should now be available under 127.0.0.1:4000
