---
published: true
title: "How to handle Images and make use of gatsby-image - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
cover_image: "https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/06/cover.png"
description: "The 'How to handle Images and make use of gatsby-image' part of a tutorial, explaining how to create an advanced Gatsby site with WordPress as a headless CMS."
tags: gatsby, wordpress, webdev, tutorial
series: "Guide to Gatsby WordPress Starter Advanced"
canonical_url:
---

After the [previous part](https://dev.to/nevernull/blog-with-pagination-guide-to-gatsby-wordpress-starter-advanced-with-previews-i18n-and-more-50g5) we have a blog ready with featured images. Now, how do we get these images to be local and do some more magic with them? Gatsby comes with a super helpful plugin called [gatsby-image](https://www.gatsbyjs.org/packages/gatsby-image/) for image processing at build time.

In this part I will show you, how you can make use of gatsby-images superpowers and deliver your images in a static fashion.

## Table of Contents

- [Add Images to WordPress pages :computer:](#add-images-to-wordpress-pages-computer)
- [Make use of gatsbyimage :camera:](#make-use-of-gatsby-image-camera)
   - [1.) Add Resolver to Gatsby](#1-add-resolver-to-gatsby)
   - [2.) Add fluid Image fragment](#2-add-fluid-image-fragment)
   - [3.) Add fragment to createPages.js and createPosts.js](#3-add-fragment-to-createpagesjs-and-createpostsjs)
      - [Update createPages.js](#update-createpagesjs)
      - [Add PageTemplateFragment](#add-pagetemplatefragment)
      - [Update createPosts.js](#update-createpostsjs)
      - [Update PostTemplateFragment and BlogPreviewFragment](#update-posttemplatefragment-and-blogpreviewfragment)
   - [4.) Update Image.js to FluidImage.js](#4-update-imagejs-to-fluidimagejs)
   - [5.) Update page, blog and post template](#5-update-page-blog-and-post-template)
- [Post Inline images :pencil:](#post-inline-images-pencil)
- [Final Thoughts :checkered_flag:](#final-thoughts-checkeredflag)
- [What's Next :arrow_right:](#whats-next-arrowright)


## Add Images to WordPress pages :computer:

After adding images to the posts, let's also add some featured images to our pages.

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

As you can see we get an absolute path to the featured images with `sourceUrl`. **What we want instead though, is a local image, with the abilities to use `gatsby-image`'s filters.**

> I already wrote [an article](https://dev.to/nevernull/gatsby-with-wpgraphql-acf-and-gatbsy-image-72m) about this, but for the sake of integrity of the tutorial series, I gonna add the information here too.

## Make use of gatsby-image :camera:

We gonna make use of Gatsby's abilities to add custom resolvers, to customize the schema, so that WordPress images get handled as local `File`s. Gatsby will then automatically treat the images as files, that are getting processed by `gatsby-image`.

### 1.) Add Resolver to Gatsby

```javascript
// gatsby-node.js

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

### 2.) Add fluid Image fragment

As in the previous part. We will add some fragments, to our queries. First let's add a `fragment.js` inside our template folder like this:

```javascript
// src/templates/fragments.js

const FluidImageFragment = `
    fragment GatsbyImageSharpFluid_tracedSVG on ImageSharpFluid {
        tracedSVG
        aspectRatio
        src
        srcSet
        sizes
    }
`

module.exports.FluidImageFragment = FluidImageFragment
```

- This fragment will serve for our fluid images with traced SVG abilities.


### 3.) Add fragment to createPages.js and createPosts.js

We need to add these fragments to our page and post queries.

#### Update createPages.js

```javascript
// create/createPages.js

const pageTemplate = require.resolve("../src/templates/page/index.js")

const {FluidImageFragment} = require("../src/templates/fragments")
const {PageTemplateFragment} = require("../src/templates/page/data")

const GET_PAGES = `
    ${FluidImageFragment}
    ${PageTemplateFragment}

    query GET_PAGES($first:Int $after:String) {
        wpgraphql {
            pages(
                first: $first
                after: $after
                # This will make sure to only get the parent nodes and no children
                where: {
                    parent: null
                }
            ) {
                pageInfo {
                    hasNextPage
                    endCursor
                }
                nodes {                
                  ...PageTemplateFragment
                }
            }
        }
    }
`

// This file has more content. Check the repo for the rest of the code.
```

- As you can see we add our `FluidImageFragment` and then parse it inside our query string.
- Also new is the `PageTemplateFragment`. We will add that fragment now.

#### Add PageTemplateFragment

In the last tutorial part we already added the posts data.js. Now we also do the same for pages with the addition of our image fragment.

```javascript
// src/templates/page/data.js

const PageTemplateFragment = `
    fragment PageTemplateFragment on WPGraphQL_Page {
        id
        title
        pageId
        content
        uri
        isFrontPage
        featuredImage {
            sourceUrl
            altText
            imageFile {
                childImageSharp {
                    fluid(maxHeight: 400, maxWidth: 800, quality: 90, cropFocus: CENTER) {
                        ...GatsbyImageSharpFluid_tracedSVG
                    }
                }
            }
        }
    }
`

module.exports.PageTemplateFragment = PageTemplateFragment
```

- `featuredImage` now has the `imageFile` field added, thats coming form our gatsby-image implementation with the resolver.
- We can make use of our `GatsbyImageSharpFluid_tracedSVG` fragment, as we added it before.

> Note: Usually gatsby-image would add these fragments already, but not in our build scripts. If you would only use the fragments inside page or static queries, you would already have these fragments available.

#### Update createPosts.js

```javascript
// create/createPosts.js

const {
  PostTemplateFragment,
  BlogPreviewFragment,
} = require("../src/templates/post/data.js")

const {FluidImageFragment} = require("../src/templates/fragments")

const { blogURI } = require("../globals")

const postTemplate = require.resolve("../src/templates/post/index.js")
const blogTemplate = require.resolve("../src/templates/post/blog.js")

const GET_POSTS = `
    # Here we make use of the imported fragments which are referenced above
    ${FluidImageFragment}
    ${PostTemplateFragment}
    ${BlogPreviewFragment}

    query GET_POSTS($first:Int $after:String) {
        wpgraphql {
            posts(
                first: $first
                after: $after
                # This will make sure to only get the parent nodes and no children
                where: {
                    parent: null
                }
            ) {
                pageInfo {
                    hasNextPage
                    endCursor
                }
                nodes {           
                    uri     

                    # This is the fragment used for the Post Template
                    ...PostTemplateFragment

                    #This is the fragment used for the blog preview on archive pages
                    ...BlogPreviewFragment
                }
            }
        }
    }
`

// This file has more content. Check the repo for the rest of the code.
```

- Same as above. We add the `FluidImageFragment` to our query.

#### Update PostTemplateFragment and BlogPreviewFragment

```javascript
// src/templates/post/data.js

const PostTemplateFragment = `
    fragment PostTemplateFragment on WPGraphQL_Post {
        id
        postId
        title
        content
        link
        featuredImage {
            sourceUrl
            altText
            imageFile {
                childImageSharp {
                    fluid(maxHeight: 400, maxWidth: 800, quality: 90, cropFocus: CENTER) {
                        ...GatsbyImageSharpFluid_tracedSVG
                    }
                }
            }
        }
        categories {
            nodes {
                name
                slug
                id
            }
        }
        tags {
            nodes {
                slug
                name
                id
            }
        }
        author {
            name
            slug
        }
    }
`

const BlogPreviewFragment = `
    fragment BlogPreviewFragment on WPGraphQL_Post {
        id
        postId
        title
        uri
        date
        slug
        excerpt
        content
        featuredImage {
            sourceUrl
            altText
            imageFile {
                childImageSharp {
                    fluid(maxHeight: 400, maxWidth: 800, quality: 90, cropFocus: CENTER) {
                        ...GatsbyImageSharpFluid_tracedSVG
                    }
                }
            }
        }
        author {
            name
            slug
        }
    }
`

module.exports.PostTemplateFragment = PostTemplateFragment
module.exports.BlogPreviewFragment = BlogPreviewFragment
```

- You might notice, that there is potential of abstraction here. For simplicity I'll leave it like this for now.

### 4.) Update Image.js to FluidImage.js

Now we will need to update our image component. For now we are only using fluid images and therefore we gonna rename our `Image.js` to `FluidImage.js` and update it like so:

```jsx
// src/components/FluidImage.js

const FluidImage = ({ image, withFallback = false, ...props }) => {
  const data = useStaticQuery(graphql`
      query {
          fallbackImage: file(relativePath: { eq: "fallback.svg" }) {
              publicURL
          }
      }
  `)

  /**
   * Return fallback Image, if no Image is given.
   */
  if (!image) {
    return withFallback ? <img src={data.fallBackImage.publicURL} alt={"Fallback"} {...props}/> : null
  }

  if (image && image.imageFile) {
    return <GatsbyImage fluid={image.imageFile.childImageSharp.fluid} alt={image.altText} {...props}/>
  }

  return <img src={image.sourceUrl} alt={image.altText} {...props}/>
}

export default FluidImage
```

- If now image, we show a fallBackImage or render no element at all.
- If we do have an image and it has the `imageFile` field, we know it has been processed by our resolver and therfore we can use `GatsbyImage` to handle it.
- If we do have an image, but no `imageFile`, then it will default to using a normal `img` tag with the `sourceUrl` as source.

> This will guarantee us, that if for some reason, we are using dynamic queries and don't have the preprocessing abilities, that we will still have a valid output of the images. This will come in handy, if we are creating previews later on.

### 5.) Update page, blog and post template

```jsx
// src/templates/page/index.js

import React from "react"

import Layout from "../../components/Layout"
import SEO from "../../components/SEO"
import FluidImage from "../../components/FluidImage"


const Page = ({ pageContext }) => {
  const {
    page: { title, content, featuredImage },
  } = pageContext

  return (
    <Layout>
      <SEO title={title}/>

      <FluidImage image={featuredImage} style={{ marginBottom: "15px" }}/>

      <h1> {title} </h1>
      <div dangerouslySetInnerHTML={{ __html: content }}/>
    </Layout>
  )
}

export default Page
```

```jsx
// src/templates/post/blog.js

import React from "react"
import Layout from "../../components/Layout"
import PostEntry from "../../components/PostEntry"
import Pagination from "../../components/Pagination"
import SEO from "../../components/SEO"

const Blog = ({ pageContext }) => {
  const { nodes, pageNumber, hasNextPage, itemsPerPage, allPosts } = pageContext

  return (
    <Layout>
      <SEO
        title="Blog"
        description="Blog posts"
        keywords={[`blog`]}
      />

      {nodes && nodes.map(post => <PostEntry key={post.postId} post={post}/>)}

      <Pagination
        pageNumber={pageNumber}
        hasNextPage={hasNextPage}
        allPosts={allPosts}
        itemsPerPage={itemsPerPage}
      />
    </Layout>
  )
}

export default Blog
```

```jsx
// src/templates/post/index.js

import React from "react"

import Layout from "../../components/Layout"
import SEO from "../../components/SEO"
import FluidImage from "../../components/FluidImage"


const Post = ({ pageContext }) => {
  const {
    post: { title, content, featuredImage },
  } = pageContext

  return (
    <Layout>
      <SEO title={title}/>

      <FluidImage image={featuredImage} style={{ marginBottom: "15px" }}/>

      <h1> {title} </h1>
      <div dangerouslySetInnerHTML={{ __html: content }}/>
    </Layout>
  )
}

export default Post
```

## Post Inline images :pencil:

One thing I wanted to mention shortly, that there is a nice plugin, that can help you to get inline images as local static images. So for example you are using the normal Gutenberg in your posts, then you want images, that you included in the post also to be downloaded at build time and stored locally.

I won't implement this here, as I won't rely on Gutenberg and rather use Advanced Custom Fields - Flexible Content field in the upcoming parts.

But checkout [gatsby-wpgraphql-inline-images](https://www.gatsbyjs.org/packages/gatsby-wpgraphql-inline-images) if you need this functionality.



## Final Thoughts :checkered_flag:

Checkout how [the site](https://gatsby-starter-wordpress-advanced.netlify.com/) looks now:

![Guide to Gatsby WordPress Starter Advanced - Screenshot](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/06/preview-screenshot.png)

> PS: That's actually me walking on a highline (slackline) in this picture :scream:. Such a fun sport.

With this part I would say, we have a very solid base for everything that is coming next. The basic functionalities of a WordPress site are implemented and we can now start think about how we dynamically design our content.

Find the code base here: [https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-6](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-6)


## What's Next :arrow_right:

We'll dive into Advanced Custom Fields Flexible Content field and create a page-builder like experience with it. This will open us the doors, to dynamically create UI elements filled with content.

**Part 7** - [PageBuilder with ACF Flexible Content](https://dev.to/nevernull/pagebuilder-with-acf-flexible-content-guide-to-gatsby-wordpress-starter-advanced-with-previews-i18n-and-more-2lf4)
