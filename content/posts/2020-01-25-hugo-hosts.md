---
title: Hosting static sites generated with Hugo
date: "2020-01-25"
comments: true
buyMeACoffee: true
tags: 
  - hugo
  - gitlab
  - github
  - netlify
---

I'm using [Hugo](https://gohugo.io/) to generate a static website / blog from markdown.

This means that I have some source files on my local machine and I run `hugo` to generate static html/css files.
Once I have these static files I needed to decide how to publish them to the internet.

This post is a subjective comparison of possible hosting solutions when working with Hugo.

## Gitlab Pages

My initial plan was to use [Gitlab Pages](https://about.gitlab.com/product/pages/) because my source files are already checked into a private git repository on Gitlab.

I used a simple `.gitlab-ci.yaml` found on https://gitlab.com/pages/hugo:

```yaml
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master
```

This would run on my master branch and publish the generated content to Gitlab Pages. 
The setup is simple and everything was in one place, a custom domain can be used and we get a Let's Encrypt TLS certificate as well.

What made me look for an alternative solution? Two things:

- For some reason I could not get custom 404 pages to work with this setup. When accessing a non-existent URL it would always
  load a generic Gitlab 404 page instead of my custom Hugo one. It may be related to this [issue](https://gitlab.com/gitlab-org/gitlab-pages/issues/183).
- The loading times seemed atrociously slow for what I was serving :snail:. 5-10 seconds on an empty cache was quite a deal-breaker.

## Github Pages

Everybody and their mom has a Github account. Github also has [Pages](https://pages.github.com/). So I tried out hosting my hugo content there.

The setup is not so elegant anymore. I followed the instructions on https://gohugo.io/hosting-and-deployment/hosting-on-github/ and ended up with **two** git repos.
One for my source files (I used my existing Gitlab repo) and one for the generated content under `/public` which needs to be set up as a git submodule.

This could still be automated with Gitlab CI pretty easily with something like (not tested):

```yaml
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

github-pages:
  script:
  - hugo
  - cd public && git -am "Build for Github Pages" && git push
  artifacts:
    paths:
    - public
  only:
  - master
```

This tells Gitlab to build the static page and then push the git submodule to Github as well.

Github Pages were subjectively faster but there are downsides:

- I still didn't get a custom 404 page :disappointed_relieved:
- I don't like having one repo for the source and one for the generated content
- Github Pages works only with **public** repositories (unless you have a paid account AFAIK)

## Netlify

I haven't heard of [Netlify](https://www.netlify.com/) before. I think it came up in some Gitlab issue while I was looking for alternatives.

I used their web UI to set everything up but I read that there's also a CLI available which should be cool.
Anyway, I just had to connect my Gitlab account and select the source repository which I had already been working on.

Netlify automagically recognized that the contents of the repo had to be build with hugo and suggested that for the build options:

{{< image src="/img/posts/netlify_screenshot.png" alt="Netlify wizard" position="center" style="border-radius: 8px;" >}}

Whenever I push to `master` in Gitlab a hook on Netlify builds and publishes the site. I could get rid of the `.gitlab-ci.yml` altogether.

I like about this setup:

- I have **one private** git repo on Gitlab
- I haven't had issues with loading times on Netlify so far

I understand that there could be one big downside to this approach for other people. 
On Gitlab / Github you get a cool `xyz.gitlab.io` / `xyz.github.io` domain for free. 
A free `xyz.netfliy.com` domain is not so cool for a tech blog - so I'd say that you **need** a custom domain if you choose to go that route.

## Conclusion

If you're using hugo to generate your static site, you can use Gitlab Pages, Github Pages and Netlify as a hosting provider quite easily.

Gitlab turned out to have slow loading times. Github requires a weird repo / branching setup. Netlify works nicely together with Gitlab but 
you'll probably want a custom domain.

 |                   | Loading Times (cold cache) | Private Repo | Domains              | TLS 
 | ----------------- | -------------------------- | ------------ | -------------------- | -------------
 | **Gitlab Pages**  | ~5s :no_entry:             | Yes          | .gitlab.io, custom   | Let's Encrypt
 | **Github Pages**  | <1s                        | No :no_entry:| .github.io, custom   | Let's Encrypt
 | **Netlify**       | <1s                        | Yes          | .netlify.com, custom | Let's Encrypt