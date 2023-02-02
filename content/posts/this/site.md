---
title: "Planning notes.grayside.dev"
date: "2023-02-01T10:56:54-08:00"
author: ""
authorTwitter: "" #do not include @
cover: ""
tags: ["decisions", "website", "static-site-generator", "hugo", "notes.grayside.dev"]
keywords: ["", ""]
description: "How this site went from objective to reality."
showFullContent: false
color: "green" # ["orange", "blue", "red", "green", "pink"]
---

I've wanted to restart my tech blogging practice for several years. I have been
blocked multiple times by my own perfectionism: I tend to choose a path that
involves customizing the tools, get a backlog of "critical tasks", then get
distracted before launching.

This time I defined a very minimal scope, launched a site with one post, and then
began improving it. The next challenge is feeling ready to broadly share links.

## Objective

Build a practice of writing things down, and sharing. Get in more discussions.

## Non-goals

Produce perfectly polished posts presenting proof of professional potential.
(I'll launch a separate site for that if I find time to write that way.)

Measure things.

## MVP

* Easily write, publish and link to posts
* See lists of all posts, optionally broken down by tags
* An RSS feed lets other people read my posts the way I prefer to read theirs
* Site aesthetic has some personality
* If folks like the content, they can find me on Mastodon to start a good chat

## Categories

* **Hosting:** Where will the site & code live?
* **Appearance:** What does it look like, what does the site say about my "brand"?
  What's the quirkiness level between the inside of my head and a business card?
* **Maintenance:** What will be involved in keeping this live?
* **Content creation experience:** How will I create and update content?
* **Data:** How will I back up data? Do I want to make it available for integrations?
  Will having more structured data in front matter be worthwhile?

## Maintenance & Lifecycle considerations

* Bursty Attention
* Long-term site (5-10? years)
* Long-term data (forever)
* Minimal tools & expenses
* Durable automation
* Simplicity

## Site architecture & tooling

I chose to use [Hugo](https://gohugo.io) as a static site generation tool.

* As a tool shipped as a binary, it's likely to continue working to some extent
  even if maintenance stops
* As a Go-based tool, I'm familiar with the templating, language, and ecosystem
  such that I will be comfortable customizing site behaviors or contributing to
  the community.
* `hugo serve` is a nice development experience

I'm using GitHub to host my code because I live there.

I'm using GitHub Pages to host my site:

* Keep number of vendors/dependencies minimal
* Avoid managing hardware
* Already familiar with using GitHub Workflows to work with GitHub Pages

## Deploying content

As a static site, the critical day to day use case is creating and publishing
content

git commit / git push is a very simple deploy model that I'm used to with code.
Extending it to content is natural.

The downside of this is using GitHub on my phone to author new content is not a
great experience. I might want to consider if there is a good app that streamlines
content creation backed by GitHub. It may also be interesting to see if I could
publish from a [Notion](https://notion.so) workspace.

## Choosing a Theme and Evaluating Customizations

For this site, I decided to present my identity as a coder and have more fun with
the look. I don't anticipate posting links to it in LinkedIn or trying to impress
employers. I'm hoping to start discussions with other techies.

I found the [terminal theme](https://themes.gohugo.io/themes/hugo-theme-terminal/)
in the [hugo theme gallery](https://themes.gohugo.io/) and made a few changes:

* I used the theme's built-in mechanism for customizations to add my own styling
* I completely overrode the theme's RSS template and I'm okay with that!

In general this theme is good. So far I've noticed a few drawbacks:

* It doesn't produce semantic HTML5 markup
* It's got some accessibility and web performance optimization opportunities
* A couple of the templates could be decomposed into more targeted partial templates
  for ease of customization

I'm avoiding customizing the theme templates because I'd really prefer to inherit
improvements from upstream. I'm currently watching the repository to see if
community Pull Requests are accepted.

## What if the tools go away?

* Backup the site with git clone
* Switch the theme to many others others in Hugo, possibly set some overrides in
  configuration or frontmatter to line up with design choices
* Switch from hugo to other static site generator: markdown with yaml front matter
  is a pretty common format and if some of the data model isn't supported the
  migration model can be "Search and Replace"
