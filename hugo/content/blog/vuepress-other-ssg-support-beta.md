---
title: Support for Vuepress [Beta]
description: ''
date: 2018-08-08 21:36:10 -1100
authors:
- Sebastian Engels
publishdate: 2017-12-07 04:00:00 +0000
expirydate: 2030-01-01 04:00:00 +0000
headline: ''
textline: ''
images: []
categories:
- CMS
tags: []
cta:
  headline: ''
  textline: ''
  calls_to_action: []
private: false
weight: ''
aliases: []
menu: []
draft: true

---
I'm thrilled to share these news with you, today! Vuepress support is here.  
  
Support for Vuepress and other static site generators was on our roadmap for a while now. If you have a look at our [launch on Producthunt](https://www.producthunt.com/posts/forestry) almost two years ago, the idea to add other SSGs had been there all along.

<iframe style="border: none;" src="https://cards.producthunt.com/cards/comments/320289?v=1" width="500" height="405" frameborder="0" scrolling="no" allowfullscreen></iframe>

_Why did it take so long?_ It's important for us create the best content management experience we can think of and make it a system that is easy-to-use for developers and editors first. That's why other features were given priority over expanding our support to other SSGs.

_Why Vuepress and not Gatsby?_ The main reason is @nichlas, our UX Designer, because he really loves everything Vue 😍. But there's also the fact that Vuepress' default support for .html and .md is similar to Hugo and Jekyll behavior which makes it a great candidate for a beta version. We already made a lot of changes (to the sidebar, imports etc.), so jumping to an SSG like Gatsby that doesn't default to .md or .html files felt like we skipped a step.

This is a big announcement for us and we're looking forward to hearing what you have to say about it. So let me walk you through importing your first Vuepress project in two steps.

<div id="ELEMENT_ID" data-proofer-ignore>
{{% create_site_button
repo="https://github.com/itsnwa/portfolio-vuepress"
branch="master"
engineName="vuepress"
engineVersion="0.12.0"
forkName="portfolio-vuepress"
heading="You don't have a Vuepress Project?"
linkText="Add Forestry's Vuepress Portfolio Theme" %}}
</div>

***

## 1. Import your Vuepress Project

You can now select your Vuepress project in our options when you [add a new site](https://app.forestry.io/dashboard/#add-site).

![](/uploads/2018/08/import-vuepress-2.png)

***

## 2. Configure Your Sidebar

After your import we'll ask you to configure your sidebar. Simply click the link after your import or go to `Settings` and select the `Sidebar` tab. Check out the [docs](https://forestry.io/docs/settings/content-sections/) for more information.

![](/uploads/2018/08/docs-configuration.png)

***

## Add Headings and Single Documents to Your Sidebar

With our latest update we've also included the ability to add section headings such as (`data` or `site`) and single files to your sidebar.

![](/uploads/2018/08/sidebar-headings-1.png)

More than other times, it is important that we get feedback on what is working for you, what isn't and what features you will need.