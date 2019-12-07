---
published: true
title: "Deployment - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
cover_image: "https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/cover.png"
description: "The Deployment part of a tutorial, explaining how to create an advanced Gatsby site with WordPress as a headless CMS."
tags: gatsby, wordpress, webdev, tutorial
series: "Guide to Gatsby WordPress Starter Advanced"
canonical_url:
---

Finally we have the basics and it's time to see what we've done in production. I'll quickly go through some ways to deploy your site. I will mainly focus on Netlify for this, but also give an example for GitLab pages.

## Table of Contents

* [Netlify Setup :zap:](#netlify-setup-zap)
* [WordPress Plugin :floppy_disk:](#wordpress-plugin-floppydisk)
* [GitLab Setup :cat:](#gitlab-setup-cat)
   * [Gatsby Prefix Configuration](#gatsby-prefix-configuration)
   * [GitLab CI Configuration](#gitlab-ci-configuration)
* [Final Thoughts :checkered_flag:](#final-thoughts-checkeredflag)
* [What's Next :arrow_right:](#whats-next-arrowright)

## Netlify Setup :zap:

1. First of all, make sure you have a repository, with all your work so far pushed to master. Preferably on GitHub. Now get yourself a Netlify account and connect it with your GitHub repository. I won't go into details for that. There is plenty of documentation out there.
![New Site from Git Button](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/new-site-from-git.png)


2. If you create a project from your Gatsby repository, Netlify should recognize, that it is a Gatsby project and you should see something like this:
![Netlify Build Options](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/netlify-build-options.png)

3. Now go to the **Build & Deploy** settings and **add a build hook**.
![Netlify Build Hook](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/netlify-build-hook.png)

4. **Copy the build hook URL for later use**.

5. By the time you got your build hook, Netlify should already be finished building your site. **Checkout your Netlify domain**.
![Netlify Build Hook](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/netlify-site.png)


## WordPress Plugin :floppy_disk:

We will setup a WordPress plugin, to be able to make use of the build hook. With this, we can decide when to trigger a build. For example, you can build only when pressing the build button, or after every post/page update. There is plenty more options.

- [WP JAMstack Deployments](https://github.com/crgeary/wp-jamstack-deployments) - To add your Netlify (or other webhook) build and to show a build status badge.
  - Also available in the WordPress Plugin registry: [here](https://wordpress.org/plugins/wp-jamstack-deployments/)

While I would suggest this plugin. There is also other plugins out there.

You can download the .zip files of these repositories and install them through WP-Admin or just **navigate to your plugin folder** and do a `git clone` like so:

```
git clone https://github.com/crgeary/wp-jamstack-deployments
```

1. Activate the plugin.

2. Now **go to Settings->Deployments** and fill in the form:
![WP JAMstack Deployments Settings](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/wp-plugin-settings.png)
  - **Build Hook URL**: The Netlify build hook we created before.
  - **Badge Image URL**: We can find the badge URL in the **General->Status badges** settings of Netlify. It consists of 2 URLs. The first one is the one we need for the status badge.
  - **Badge Link**: When you click on the Netlify badge, the link you set here will open. You can either set it to your Netlify site domain, or you use the second URL from the status badge, which will link to your Netlify deploy status page.
3. **Save Settings** and **Test Deployment**. We should see the badge in the admin toolbar being in `Building` mode.


Now we can trigger our rebuilds with the **Deploy Website** button in the admin toolbar. I like to only rebuild when I changed more then just a post, especially if there is not incremental builds in place (yet), so I'll leave all the checkboxes blank and rebuild my site as I see fit.

:ballot_box_with_check: Easy deployments done

## GitLab Setup :cat:

We really like working with GitLab on our client projects. With GitLab Pages you can have an easy way to setup a preview page, that is only accessible for authorized users, that are part of the project. you can also make it public if you want so.

Checkout the [docs on GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/#gitlab-pages) for some more insight.

### Gatsby Prefix Configuration

Go to your `gatsby-config.js` and add the following line inside `module.exorts = {...}`:

```javascript
  pathPrefix: `${process.env.PATH_PREFIX}`,
```

Now this way we can define the path prefix either in our .env files, our we use GitLab Page environment variables for it. Let's use the latter, to be as flexible as possible.

### GitLab CI Configuration

Depending on where your project is placed, GitLab will host your page on a domain, that will look like so:
![GitLab Page Domain Naming](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/gitlab-page-names.png)

> Checkout: [GitLab Page Domain Naming](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_one.html#gitlab-pages-default-domain-names)

As you can see, you will have `/projectname` in your URL. Therefore, we need to tell Gatsby to prefix your routing.

---

1.) **Head over to your GitLab** project, go to **Settings->CI/CD->Variables** and add your `PATH_PREFIX` like so:

![GitLab CI Variables](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/04/gitlab-vars.png)


Now the only thing missing is the CI config file inside your Gatsby projects root.

2.) **Add following snippet** to your `.gitlab-ci.yml`:

```yaml
# .gitlab-ci.yml

image: node:12.13.0

pages:
  script:
    - npm install
    - npm install gatsby-cli
    - ./node_modules/.bin/gatsby build --prefix-paths
  artifacts:
    paths:
      - public
  cache:
    paths:
      - node_modules
  only:
    - master
```

As you can see in the third line under `script:`, we call Gatsby like `./node_modules/.bin/gatsby build --prefix-paths`. This will make sure, that Gatsby is actually using the prefix in the build.

Thats it. You should be able to find your page under the domain as described above. **Sometimes it needs a few minutes, if it is the first time you deploy the page. Be patient.** :wink:


## Final Thoughts :checkered_flag:

Yaaah, our site is online now! :rocket: There is many more ways to deploy your site. But I guess you get a good idea of how things can work with the example of these two common approaches.


Find the code base here: [https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-4](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-4)

Find the GitLab code base in this extra branch: [https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-4-gitlab](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-4-gitlab)


## What's Next :arrow_right:

In the upcoming part we will introduce a blog page with pagination, which will serve as overview for our blog post previews.

**Part 5** - [Blog with Pagination](https://dev.to/nevernull/blog-with-pagination-guide-to-gatsby-wordpress-starter-advanced-with-previews-i18n-and-more-50g5)
