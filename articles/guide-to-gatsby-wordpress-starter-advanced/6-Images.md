---
published: false
title: "How to handle Images and make use of gatsby-image - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
cover_image: "https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/06/cover.png"
description: "The Deployment part of a tutorial, explaining how to create an advanced Gatsby site with WordPress as a headless CMS."
tags: gatsby, wordpress, webdev, tutorial
series: "Guide to Gatsby WordPress Starter Advanced"
canonical_url:
---

After the [previous part](https://dev.to/nevernull/deployment-guide-to-gatsby-wordpress-starter-advanced-with-previews-i18n-and-more-2g2o) we are able to deploy our site. Now what about images? Gatsby comes with a super helpful plugin called [gatsby-image](https://www.gatsbyjs.org/packages/gatsby-image/) for image processing at build time.

In this part I will show you, how you can make use of gatsby-images superpowers and deliver your images in a static fashion.

## Table of Contents

## Add Images to WordPress pages

Let's add some featured images to our pages.

![Add Featured Image](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/06/add-featured-image.png)

Now run `gatsby develop` and checkout http://localhost:8000/___graphql.

**Run**:

```graphql
query GETPAGES {
  wpgraphql {
    pages {
      nodes {
        title
        uri
        featuredImage {
          sourceUrl
        }
      }
    }
  }
}
```

**You should see something like this**:

![GraphQL Image Output](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/06/graphql-image-output.png)

As you can see we get an absolute path to the featured images with `sourceUrl`. **What we want instead though, is a local static image, with the abilities to use `gatsby-image`'s filters.**

> I already wrote [an article](https://dev.to/nevernull/gatsby-with-wpgraphql-acf-and-gatbsy-image-72m) about this, but for the sake of integrity of the tutorial series, I gonna add the information here too.

## Make use of gatsby-image

We gonna make use of Gatsby's abilities to add custom resolvers, to customize the schema, so that WordPress images get handled as local `File`s. Gatsby will then automatically treat the images as files, that are getting processed by `gatsby-image`.

### Add Resolver to Gatsby

```javascript
const { createRemoteFileNode } = require(`gatsby-source-filesystem`)

exports.createResolvers = (
  {
    actions,
    cache,
    createNodeId,
    createResolvers,
    store,
    reporter,
  },
) => {
  const { createNode } = actions
  createResolvers({
    WPGraphQL_MediaItem: {
      imageFile: {
        type: `File`,
        resolve(source, args, context, info) {
          return createRemoteFileNode({
            url: source.sourceUrl,
            store,
            cache,
            createNode,
            createNodeId,
            reporter,
          })
        },
      },
    },
  })
}
```
- `WPGraphQL_MediaItem`: This depends on your config. It starts with the `typeName` of your gatsby-source-graphql.
- [createRemoteFileNode](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/#createremotefilenode) gives you the ability to pull in remote files and automatically adds them to your schema.
- `imageFile`: will be the type you can query (see below).
- `type: 'File'`: will add the MediaItems as Files, which is great, because now gatsby-image can make use of it.
- `url: source.sourceUrl`: Is the images Url coming from your WordPress


## Pages Query
```graphql

# page.js

query {
  WPGraphQL {
    pages {
      nodes {
        featuredImage {
          sourceUrl
          imageFile {
            childImageSharp {
              fixed {
              ...GatsbyImageSharpFixed
              }
            }
          }
        }
      }
    }
  }
}
```

- WordPress Bilder anlegen
  - Featured images
  - Post Inline images.
- Gatsby image
  - gatsby resolver
  - Fallback image?
  -  



## What's Next :arrow_right:

Next we'll dive into Advanced Custom Fields Flexible Content field and create a page-builder like experience with it.

Coming up soon: **Part 6** - PageBuilder with ACF Flexible Content
