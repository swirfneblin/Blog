---
layout: post
title:  "Passos para criar um site/blog com Jekyll"
date:   2020-06-26 13:20:00 -0300
categories: jekyll
---

# 3 Steps to Create a Site/Blog with Jekyll – Hyvor Talk Blog

> This article describes how to install Jekyll and deploy on Github Pages. It only take several minutes for the complete process.

![Jekyll](https://talk.hyvor.com/blog/wp-content/uploads/2020/02/photo-1520888240-b6b27468f871.jpg)

595

On the one hand, you can set up a fully-functional CMS like WordPress as your blog. On the other hand, you can set up a static site, such as Jekyll or Gatsby, and have a simple but blazing fast blog. This article guides you through the process of setting up a blog site with Jekyll and [Github Pages](https://pages.github.com/). First, we’ll set up the Jekyll blog locally and then we’ll deploy it on Github Pages.

![Jekyll Island, United States](https://talk.hyvor.com/blog/wp-content/uploads/2020/02/photo-1520888240-b6b27468f871.jpg)

Jekyll Island, United States ([Unsplash](https://unsplash.com/photos/w3C4qaslpWs))

Introduction
------------

Jekyll, the official website at [jekyllrb.com](http://jekyllrb.com/), is a static site generator that is built to be minimal, fast, and easy-to-manage. Github co-founder Tom Preston-Werner created Jekyll in 2008 (Yes, that’s so long).

Jekyll is the engine behind Github Pages. Hence, we can host Jekyll Blogs on Github Pages for no cost. To our surprise, Github Pages supports custom domains. Therefore, we can host it there while having our own domain name (`blog.example.com`).

> Jekyll does what you tell it to do — no more, no less. It doesn’t try to outsmart users by making bold assumptions, nor does it burden them with needless complexity and configuration. Put simply, Jekyll gets out of your way and allows you to concentrate on what truly matters: your content.
> 
> – Jekyll README.md

Facts about Jekyll
------------------

*   Uses [Markdown](https://en.wikipedia.org/wiki/Markdown)
*   Uses [Liquid Templates](https://jekyllrb.com/docs/liquid/)
*   0.01% of websites use Jekyll ([Source](https://w3techs.com/technologies/details/cm-jekyll))
*   Jekyll is installed as a [Ruby Gem](https://rubygems.org/) in local computers.
*   Jekyll does not parse files on each request. It creates static pages and saves them in the `_site` folder on the build. Files are statically served through a server.
*   Jekyll does not use databases.

**Demystification**: A common confusion among those who are new to Jekyll is considering Jekyll as a blogging software (or Content Management System) – it is not! Jekyll is only an engine to parse files. In WordPress, you can write on a web-based rich editor and publish content. However, in Jekyll, you write content in Markdown in a plain text editor. Then, you publish new updates online.

(Make sure you are fine with that.)

Prerequisites
-------------

While coding knowledge isn’t necessary to set up or use Jekyll, an understanding of the following concepts is much needed.

*   **Markdown** – This is how content is written in Jekyll. It doesn’t take much time to learn markdown. Check out [Markdown cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).
*   **Git** – Used to version control the code. As we will be using Github to host our code, you need to be familiar with Git version controlling. If you aren’t familiar with it, I suggest you start with the [Git Handbook](https://guides.github.com/introduction/git-handbook/) by Github.
*   **Command Line** – A little experience using the command line will be essential. If you don’t have that experience, don’t worry. You’ll only need to know how to open the command line and navigate to a folder (Search Google with your OS).

(Note for non-devs: The process seems overwhelming. But, it’s easy. You just need to get accustomed to it)

Setting up Jekyll
-----------------

I’d like to break down the process into 3 steps.

1.  Setting up Jekyll Locally
2.  Setting up Github and Github Pages and deploying
3.  Customizing

### 1\. Setting up Jekyll Locally

Jekyll is written in Ruby. So, you have to install it first on your computer.

*   [Official Installation Guide for Ruby](https://www.ruby-lang.org/en/documentation/installation/)

Next, you need to install two Ruby Gems (Ruby Gems are packages that add functionalities to Ruby projects)

Here, we will install `jekyll` and `bundler` Gems using the `gem` command which will be available after installing Ruby. Open the command line and run the following code.

    gem install jekyll bundler

Then, navigate to the folder where you need the new blog to reside. (Ex: In windows, `cd F:\MyProjects`)

Next, run the following command to install a new Jekyll site.

    jekyll new my-blog

This installs the Jekyll files in the `my-blog` folder in the current directory. Next, navigate to that folder.

    cd my-blog

Final command:

    bundler exec jekyll serve

This will serve the newly created static site at `http://localhost:4000`. Visit the URL on your web browser and see your beautiful Jekyll site.

There, you’ll notice some placeholder values such as `your-email@example.com`. We’ll change them and write our first post now.

#### Changing Configuration

Open `_config.yml` and do some editing. It is a YAML file. Thankfully, Jekyll’s config file explains the YAML syntax in itself. So, see the instructions in the file before editing it.

    title: My Cute Jekyll Blog
    email: me@mycutejekyllblog.com
    description: >-
      This is my cute jekyll blog. I post some awesome stuff every day.
    baseurl: ""
    url: "https://mycutejekyllblog.com"
    twitter_username: MyCuteJekyllBlog
    github_username:  MyCuteJekyllBlog

The `serve` command automatically updates the static content when you edit the content and data files. However, after editing `_config.yml`, you have to **restart the server**!

#### Creating the First Blog Post

All posts are saved in the `_posts` folder.

All posts should have the `YEAR-MONTH-DAY-title.md` format. Some examples are:

*   2020-02-16-how-to-get-up-early.md
*   2020-02-20-best-tools-for-remote-teams.md

All posts start with a [Front Matter](https://jekyllrb.com/docs/front-matter/), a YAML code between triple-dashed lines.

    ---
    layout: post
    title: Best Tools for Remote Teams
    bestTool: SomeTool
    ---

In the Front Matter, you can set values for predefined variables or define new variables. In the above example, `layout` and `title` are predefined variables while `bestTool` is a new variable that can be accessed on the page as follows.

All blog posts have the layout type `post`.

    ---
    layout: post
    title: Best Tools for Remote Teams
    bestTool: SomeTool
    ---
    
    <h2>The Best Tool is {{ page.bestTool }}</h2>

Now, you know how to create a blog post. Create one an save it in the `_posts` folder. Then, reload the site. You’ll see new updates in realtime.

### 2\. Setting up Github and Github Pages and deploying

Up to now, we created our Jekyll blog locally, customized the configuration, and published our first post.

Now, we are going to deploy it.

First of all, we have to version control our site using [Git](https://git-scm.com/). Run the following commands in the folder of the site.

    git init
    git add .
    git commit -m "Initial Commit"

When installing Jekyll, it should automatically create a `.gitignore` file in the root with at least the following content. (If not, create one)

    _site
    .sass-cache
    .jekyll-metadata

Then, log in to Github and [create a new repository](https://github.com/new) named `your_username.github.io`.

Next, copy the HTTPS URL of the repository.

![Copying the Github Repository URL before deploying Jekyll Site](https://talk.hyvor.com/blog/wp-content/uploads/2020/02/image-1.png)

Then, add the remote repository and push the code. (Make sure to replace the link with your own one)

    git remote add origin https://github.com/HungryMan/hungryman.github.io.git
    git push origin master

Done! Your site must now be available on `yourusername.github.io`. ?

#### Updating a post

After updating a post (or posts) all you have to do is committing and pushing the changes.

    git add .
    git commit -m "Updated x post"
    git push origin master

### 3\. Customizing

You can do several things to customize and add functionalities to your static site.

#### 1\. Add Comments

Static sites do not have databases. So, you have to use a third-party commenting system on your site. We recommend you to use [Hyvor Talk](https://talk.hyvor.com/) to add comments to your new site because it’s fast, free, ad-free, and fully-customizable, and it only takes a few minutes to set up (just copying and pasting code).

See [How to add Hyvor Talk comments to Jekyll?](https://groups.hyvor.com/HyvorTalk/462/how-to-add-comments-jekyll-blog)

**Recommended Reading**: [Why Hyvor Talk is a better option for static sites than its alternatives?](https://talk.hyvor.com/blog/comments-for-static-sites/)

#### 2\. Change Permalink

By default, your blog has the URL structure `/:categories/:year/:month/:day/:title`. However, [search engines love](https://talk.hyvor.com/blog/on-page-seo-checklist/) (and users too) concise and descriptive URLs.

Ex: `https://mywebsite.com/on-page-seo` is better than `https://mywebsite.com/2020/01/20/on-page-seo`.

Add permalink to `_config.yml`.

    permalink: /:title

Learn more about [permalinks](https://jekyllrb.com/docs/permalinks/) here.

#### 3\. Set up a custom domain

First, you have to buy a domain from a domain registrar.

After doing that, add a CNAME record in your [DNS zone](https://talk.hyvor.com/blog) pointing to `yourusername.github.io`.

Next, create a file named `CNAME` in the root of your site. Then, add your domain name there.

    blog.example.com

Finally, commit and push the changes. (As we updated a post)

#### 4\. Change the Jekyll Theme

Don’t use common themes. Find a brand new, beautiful theme for your website. There are thousands of free and premium themes out there. Do a little search. You’ll find many of them.

See how to change [Jekyll Themes](https://jekyllrb.com/docs/themes/).

Conclusion
----------

In this article, we discussed how to build a static blog with Jekyll sites. If you are a developer, the process must be a piece of cake. However, it does not take much time to set up a blog with Jekyll. And, writing and updating content is much easier. If you encounter errors while setting up, let us know below.

Thank you for reading! ✌

#### About Hyvor Talk

[Hyvor Talk](https://talk.hyvor.com/?utm_source=blog&utm_medium=bottom_note) is a privacy-focused alternative to Disqus. It comes with all moderation tools you need to build an amazing audience on your website. It is built to be beautiful and fully-customizable. You can control everything in one console. Try it for free now! ❤


[Source](https://talk.hyvor.com/blog/creating-a-static-site-blog-with-jekyll-and-github-pages/)