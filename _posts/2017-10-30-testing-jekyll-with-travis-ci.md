---
title: "Testing Jekyll with Travis CI"
layout: post
tags:
- CI/CD
- DevOps
- Travis CI
category: [DevOps]
excerpt: "I've finally settled with the fact that Jekyll is going to be my mainstay for a while. It's got a lot of features I like some of which are subsequent to being a static site generator, others being thoughtful implementations."
---

I've finally settled with the fact that Jekyll is going to be my mainstay for a while. It's got a lot of features I like some of which are subsequent to being a static site generator, others being thoughtful implementations. Here's a few highlights:

* [Kramdown](https://kramdown.gettalong.org/). If you haven't heard of Kramdown, it's like normal markup language on steroids. It gave me a lot of bootstrap-esque features I liked, like notifications.
* [Liquid Template Language](https://shopify.github.io/liquid/). Liquid is a lot like Jinja2 but for Ruby.
* My host is GitHub Pages, so I never fail to scale or incur huge costs.
* My website is all HTML, CSS, and JS so it loads rather quickly.
* Local building and website previewing. No surprises.
* My IDE is my writing platform.
* Everything is version controlled.
* Testing.

That last one makes me smile. I'm all about iterative development but like everyone else I get nervous as I figure out my path forward and begin rewriting. I like being able to at least establish some simple checks. In terms of Jekyll I wanted to know at least two things:

* Does my site build?
* Do I have formatting errors?

I was relieved to find that the Jekyll CLI supports proper exit codes for builds (`bundle exec jekyll b`) and that there's a gem available to do HTML proofing called, wouldn't you know it, [HTMLProofer](https://github.com/gjtorikian/html-proofer). Setting this up was rather simple for my workflow, especially once I discovered [Travis CI](https://travis-ci.org).

Travis authenticates against your GitHub account and may require a token depending on how your profile is setup. In terms of GitHub Pages, my page is _not_ attached to an organization so it's run off of the _master_ rather than _gh-pages_ branch. This is actually quite nice because it allows me to create a branch for an article I want to write. When I think I'm done, I just push to the branch and open a pull-request.

You can make some fundamental decisions at this point but I chose to leave branch building and PR building on. Sometimes, if it's just a quick update, I'll push straight into master so it's nice to know if I did something stupid.

![Travis Settings](/img/posts/2017-10-30-testing-jekyll-with-travis-ci/travis-settings.png)

Next you need to drop in a Gemfile:

```ruby
source 'https://rubygems.org'

gem "github-pages", group: :jekyll_plugins
gem 'html-proofer'
```

Travis, when using Ruby, comes with Rake and Bundler pre-installed. _github-pages_ loads the version of Jekyll that GitHub Pages uses plus their white-listed plugins. This makes local development super simple, in case you weren't sold on Jekyll yet.

Next we need to tell Travis what we're doing and how to do it. Here's my _.travis.yml_:

```yaml
language: ruby
rvm:
  - 2.4
script:
  - bundle exec jekyll build --future
  - bundle exec htmlproofer ./_site --only-4xx --check-favicon --check-html
env:
  global:
    - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
```

I have a Mac so I have Ruby 2.4 installed by default. _script_ directs Travis to run some shell commands on your behalf. If the step returns _0_ then it passes, anything else and it fails so be sure to consider your applications exit codes. `jekyll build` simply builds my site into the _\_site_ directory, this includes any updates to SASS. I included `--future` because by default Jekyll won't build articles that are beyond today but I definitely want them tested. Don't worry, this won't effect your GitHub Page. Next we instruct _htmlproofer_ to do it's testing. If you're interested here's a list of [all their flags](https://github.com/gjtorikian/html-proofer).

Now you're set! The best part of this is you can actually run htmlproofer locally since you have the Gemfile. Just run a quick `bundle` in the root directory, so long as you have [bundler](http://bundler.io/) installed, then copy the exact _bundle_ commands from above: `bundle exec htmlproofer ./_site --only-4xx --check-favicon --check-html`

This provides a nice, quick and to the point CI/CD strategy for your Jekyll blog. If you want a good frame of reference, [here's my repo](https://github.com/mattouille/mattouille.github.io).
