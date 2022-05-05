---
title: "Install Hugo"
date: 2022-01-29T01:18:11+01:00
draft: false
tags: ["hugo","go"]
categories: ["Markdown"]
---

# Installing my first GH pages

### Gots to learn right!?

* First Read the official [guide](https://pages.github.com/).
* Then I learned I could use [Hugo](). I love Go.
* I followed the Hugo [Geting started-guide](https://gohugo.io/getting-started/quick-start/).
* Also how to install and deploy on Github. [install on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/).
* I chose a theme I liked from the [Hugo theme pages](https://themes.gohugo.io/).
* Use the [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo) actions 
* Some trial and errors later.
* * You must change your Toml-file accordingly, in my case, from the [theme I chose](https://hugodoit.pages.dev/theme-documentation-basics/#basic-configuration).
* * You must build before deploy. `hugo`
* * You must select `gh-pages` branch as source in `Settings -> Pages`
* * In my `./content/posts/install-hugo.md` I had to change `draft: true` to `draft: false`

