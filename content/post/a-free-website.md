+++
author = "Michael Collins"
title = "A Free Website!"
date = "2023-02-24"
description = "How I created this free website with Hugo, GitLab, CloudFlare and Calendy!"
tags = [
    "web hosting",
    "business",
    "meta"
]
+++

This is how I created this very website for FREE using Hugo and other technologies to host a "static website". A lot of websites today have server-side scripting, databases or other complex systems (like WordPress) powering their websites. But web hosting can be simpler and more affordable by opting out of these heavyweight features and creating a website that's just simple static content. Since hosting a static website is so affordable, many large tech companies (GitHub, GitLab, Amazon, CloudFlare) will offer to host your static website for free to encourage you to use their platform.

### It All Starts With a Domain Name

---

Disclaimer: before we start, it's important to note that creating a static website is not entirely free. In this post, we will assume that you have already rented a domain name, which can cost anywhere from a few dollars to hundreds of dollars per year. In our case, we rented the perthserverplus.com domain from [www.namesilo.com](https://www.namesilo.com/) for a yearly fee of ~$16 AUD.

![NameSilo Logo](https://upload.wikimedia.org/wikipedia/commons/thumb/1/12/NameSilo_Logo.svg/798px-NameSilo_Logo.svg.png)

### Hugo Themes

---

Now that we have our domain name, we need a way to build our website. For this, we will use Hugo, a static website development framework that is easy to use and allows us to create beautiful websites quickly. Hugo is open-source, which means that it is free to use and has a large community of developers who contribute to its development.

![Hugo Themes](https://perthserverplus.com/images/hugo-themes.png)

https://themes.gohugo.io/


To start, we need to select a Hugo themes that fits our needs. Hugo has a wide variety of free and premium themes available, so we can choose one that fits our brand or services. Once we have selected our theme, we can start editing our website to our liking. Hugo uses Markdown, a simple markup language, to create content, which makes it easy for anyone, even those with no coding experience, to create beautiful websites.

### Gitlab Pages

---

![GitLab Pages](https://perthserverplus.com/images/gitlab-pages.png)

https://docs.gitlab.com/ee/user/project/pages/getting_started/pages_new_project_template.html

https://docs.gitlab.com/ee/user/project/pages/custom_domains_ssl_tls_certification/

Once we have created our website, we need to host it somewhere. For this, we will use GitLab Static Pages, a free hosting service that is perfect for static websites. GitLab Static Pages offers free hosting, unlimited bandwidth, and allows us to use custom domains, which means that we can use our perthserverplus.com domain name. To deploy our website to GitLab Static Pages, we can use a CI/CD pipeline that compiles our Hugo website automatically. 

The following YAML code can be used in your GitLab repository to create the pipeline:
```
image: registry.gitlab.com/pages/hugo/hugo_extended:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
  - hugo
  artifacts:
    paths:
    - public
  rules:
  - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### CloudFlare Email Forwarding and CDN

---

With our website hosted, we need to make sure that it is fast and secure. For this, we will use Cloudflare, a free Content Delivery Network (CDN) that accelerates and protects our website. Cloudflare caches our website's content, which makes it faster to load for visitors from different parts of the world.

I'll also be using CloudFlare to setup free email forwarding from [michael@perthserverplus.com](mailto:michael@perthserverplus.com) to my email account.

![GitLab Pages](https://perthserverplus.com/images/cloudflare.png)

### Calendy Scheduler

---

![GitLab Pages](https://perthserverplus.com/images/calendy.png)

Finally, we need a way for our clients to book appointments with us. For this, we will use Calendly, a free scheduling system that integrates with our website. Calendly allows us to set up our availability and create custom booking links that we can share with our clients. With Calendly, our clients can easily book appointments with us without having to call or email us.

---

In conclusion, building a website for free, or at a very low cost, is possible with the right tools and a bit of creativity. With Hugo, GitLab Static Pages, Cloudflare, and Calendly, we can create a beautiful and functional website that promotes our brand or services online. Remember, the monthly ongoing costs, besides the domain name, **is $0**, which makes it an excellent option for anyone who wants to create a website on a budget.