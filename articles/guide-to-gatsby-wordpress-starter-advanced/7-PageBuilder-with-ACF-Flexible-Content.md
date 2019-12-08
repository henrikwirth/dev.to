---
published: false
title: "HPageBuilder with ACF Flexible Content - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
cover_image: "https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/cover.png"
description: "The 'PageBuilder with ACF Flexible Content' part of a tutorial, explaining how to create an advanced Gatsby site with WordPress as a headless CMS."
tags: gatsby, wordpress, webdev, tutorial
series: "Guide to Gatsby WordPress Starter Advanced"
canonical_url:
---

If you are still following this series, **you learned a lot of basics in the previous parts** and are pretty well informed about the creation of Gatsby sites with WordPress.

This part will introduce a lot of new functionality, ideas and concepts. Therefore, **I decided to make a video**, where I will talk you through all the steps mentioned in this written tutorial. I hope this will help you to understand what is happening and why.

> Note: We don't have any proper styling in place yet. So I will keep things as simple as possible. I want to refactor everything later on with theme-ui and emotion.

Now let's move forward and build a page builder with Advanced Custom Field's **Flexible Content** field.

## Table of Contents


## Install WordPress Plugins :floppy_disk:

- [Advanced Custom Fields - PRO](https://www.advancedcustomfields.com/pro/)
- [WPGraphQL for Advanced Custom Fields](https://github.com/wp-graphql/wp-graphql-acf)

**Install Advanced Custom Fields PRO and activate your license**. We **need the PRO** version if we want to use the flexible content field.

You can download the .zip files of the `wp-graphql-acf` repository and install it through WP-Admin or just **navigate to your plugin folder** and do a `git clone` like so:

```shell
git clone https://github.com/wp-graphql/wp-graphql-acf
```

The WPGraphQL for Advanced Custom Fields is the plugin, that exposes the ACF fields to the GraphQL Schema. There is some good [documentation of it here](https://docs.wpgraphql.com/extensions/wpgraphql-advanced-custom-fields/).

## Create ACF fields and Content

We will add a flexible content field and 2 layouts to start with.

> **Note**: If you want to skip this parts 1.) and 2.), I prepared a `.json` export for the 2 layouts. Find it [right here]().


### 1.) Create Flexible Content field

![ACF PageBuilder](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/acf-pagebuilder.png)

- As you can see we called the field group `Page Builder`
- We added one field of type `Flexible Content` with the name `layouts`
- In the `Location settings` we add the rule `Post type is equal to Page`, so the page builder will be only visible for our pages, but not our posts.

![ACF PageBuilder Settings](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/acf-pagebuilder-settings.png)

- When you scroll down you see `GraphQL Field Name` in the Settings tab. Make sure to give it a camelCase name. This will be the field name for the GraphQL schema.
- We call it `pageBuilder`

### 2.) Create Layouts inside the Flexible Content field

![ACF Layouts](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/acf-layouts.png)

- Always make sure to have `Show in GraphQL` to be turned on.
- Inside the flexible content field we add layouts with some fields like in the picture above. We add two for now: `Hero` and `TextBlock`
- The **name** of the layout usually is something with underscore, like `text_block`. This will then transform into `textBlock` in your GraphQL schema.

### 3.) Update your Pages

Now that we have the ACF fields setup, we need to update our pages.

![ACF Layouts](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/page-update.png)

- Delete the content in the normal Gutenberg blocks.
- Only leave the title at the top.
- At the bottom **open the Page Builder** tab and `Add Row`
  - Add your blocks and fill in some data.
- I'm using the image for the `Hero` block and still leave it in thew `featuredImage`. This will give me the option to use a different featuredImage for social previews if I want to do so, later on.

> **Note**: At this point we don't need the Gutenberg editor anymore for our pages. There is no option to disable it yet. There are some plugins I think, but what I usually do, is to just hide the content blocks through a custom plugin. I might introduce that maybe in another tutorial part.

### 4.) Checkout the Schema

Run your Gatsby with `yarn clean && yarn develop` and got to [http://localhost:8000/___graphql](http://localhost:8000/___graphql) to see what happend int the schema.

If you are using the same ACF layouts like me, you should be able to query the following:

```graphql
query GET_LAYOUTS {
  wpgraphql {
    pages {
      nodes {
        pageBuilder {
          layouts {
            ... on WPGraphQL_Page_Pagebuilder_Layouts_Hero {
              text
              fieldGroupName
              textColor
              image {
                sourceUrl
              }
            }
            ... on WPGraphQL_Page_Pagebuilder_Layouts_TextBlock {
              backgroundColor
              fieldGroupName
              textColor
              text
            }
          }
        }
      }
    }
  }
}
```

- Check the results and get familiar with your schema.

## Add layouts to Gatsby

... show how to add the queries and allcomponents

## Improve performance through Template-Strings technique

... explain why and how to improve performance with template-strings 
