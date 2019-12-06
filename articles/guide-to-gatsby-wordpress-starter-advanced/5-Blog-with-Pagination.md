---
published: true
title: "Blog with Pagination - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
cover_image: "https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/05/cover-05.png"
description: "The Blog with Pagination part of a tutorial, explaining how to create an advanced Gatsby site with WordPress as a headless CMS."
tags: gatsby, wordpress, webdev, tutorial
series: "Guide to Gatsby WordPress Starter Advanced"
canonical_url:
---

I figured, for further parts of the tutorial, it will make sense to implement the **Blog** now. So this part will show you how to create a Blog (post overview) with pagination.

> I renamed all components to start with a capital letter. Checkout [this](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/commit/35677e555338e315530b50877ff5b53a05ccd846) commit to see the refactoring I did and what changed. Or, just dive into the tutorial first and compare later.

## Table of Contents

- [Add content to WordPress :computer:](#add-content-to-wordpress-computer)
  - [1.) Create posts](#1-create-posts)
  - [2.) Add Menu Link](#2-add-menu-link)
- [Implement Blog components :floppy_disk:](#implement-blog-components-floppydisk)
  - [1.) Add globals.js](#1-add-globalsjs)
  - [2.) Create blog.js](#2-create-blogjs)
  - [3.) Create PostEntry.js](#3-create-postentryjs)
  - [4.) Create Image.js](#4-create-imagejs)
  - [5.) Create Pagination.js](#5-create-paginationjs)
- [Adjust Post/Blog Creation :notebook:](#adjust-postblog-creation-notebook)
  - [1.) Add data.js](#1-add-datajs)
  - [2.) Adjust createPosts.js](#2-adjust-createpostsjs)
- [Final Thoughts :checkered_flag:](#final-thoughts-checkeredflag)
- [What's Next :arrow_right:](#whats-next-arrowright)


## Add content to WordPress :computer:

### 1.) Create posts

Let's add some posts. Preferably more than 10, so that we can see our pagination in action later on. Make sure to have some posts with featured images.


### 2.) Add Menu Link

Now add a custom link to the Menu with `/blog/`


## Implement Blog components :floppy_disk:

Now, let's create the Blog page to preview all our posts.

### 1.) Add globals.js

First of all I added a file called `globals.js` to the root of the project. It will serve as our "options" for the project. Like what we later might wanna use if we use this starter as a theme.

```jsx
// globals.js
const Globals = {
  blogURI: 'blog'
}

module.exports = Globals
```

### 2.) Create blog.js

Now, create blog.js inside the **templates/post folder**:

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

- As you can see I already made use of some components we will create in the upcoming steps.

### 3.) Create PostEntry.js

```jsx
// src/components/PostEntry.js

import React from "react"
import { Link } from "gatsby"
import Image from "./Image"
import { blogURI } from "../../globals"

const PostEntry = ({ post }) => {

  const { uri, title, featuredImage, excerpt } = post

  return (
    <div style={{ marginBottom: "30px" }}>
      <header>
        <Link to={`${blogURI}/${uri}/`}>
          <h2 style={{ marginBottom: "5px" }}>{title}</h2>
          <Image image={featuredImage} style={{ margin: 0 }}/>
        </Link>

      </header>

      <div dangerouslySetInnerHTML={{ __html: excerpt }}/>
    </div>
  )
}

export default PostEntry
```

### 4.) Create Image.js

```jsx
// src/components/Image.js

import React from "react"
import { useStaticQuery, graphql } from "gatsby"

const Image = ({ image, withFallback = false, ...props }) => {
  const data = useStaticQuery(graphql`
      query {
          fallBackImage: file(relativePath: { eq: "fallback.svg" }) {
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

  return <img src={image.sourceUrl} alt={image.altText} {...props}/>
}

export default Image
```

The image component for now works simply with the absolute path to your WordPress installation. We will get more into image-processing in the next part of the tutotorial.

- `withFallback` gives you the option to show a fallback image if there is no image passed down. If `withFallback` is false, like in the default, then it will simply not render any dom element.
- Place the `fallback.svg` image inside your images folder. (Find my fallback image [here](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/blob/tutorial/part-5/src/images/fallback.svg))

### 5.) Create Pagination.js

Now if we want to only show a certain number of post previews on one page, we need to implement pagination.

```jsx
// src/components/Pagination.js

import React from "react"
import { Link } from "gatsby"
import { blogURI } from "../../globals"

const Pagination = ({ pageNumber, hasNextPage }) => {

  if (pageNumber === 1 && !hasNextPage) return null

  return (
    <div style={{ margin: "60px auto", textAlign: "center" }}>
      <h2>Posts navigation</h2>
      <div>
        {
          pageNumber > 1 && (
            <Link
              className="prev page-numbers"
              style={{
                padding: "4px 8px 5px 8px",
                backgroundColor: "rgba(0,0,0,.05)",
                borderRadius: "3px",
              }}
              to={pageNumber > 2 ? `${blogURI}/page/${pageNumber - 1}` : `${blogURI}/`}
            >
              <span>Previous page</span>
            </Link>
          )
        }
        <span aria-current="page" className="page-numbers current" style={{ padding: "5px 10px" }}>
          <span className="meta-nav screen-reader-text">Page </span>
          {pageNumber}
        </span>

        {
          hasNextPage && (
            <Link
              style={{
                padding: "4px 8px 5px 8px",
                backgroundColor: "rgba(0,0,0,.05)",
                borderRadius: "3px",
              }}
              className="next page-numbers"
              to={`${blogURI}/page/${pageNumber + 1}`
              }
            >
              <span>Next page </span>
            </Link>
          )
        }
      </div>
    </div>
  )
}

export default Pagination
```

- If `pageNumber === 1 && !hasNextPage` we can return null, as we only have one page and don't need any pagination.
- If the current pageNumber is more than 1, we show a **Previous Page** Link/Button
- If the current page `hasNextPage`, we show a Link/Button to the **Next page**


## Adjust Post/Blog Creation :notebook:

We have to adjust a couple of things in `createPosts.js`. I'll introduce a new concept of GraphQL fragments, that live in `data.js` files. This will help us to separate some concerns and make our code structure more structured.

### 1.) Add data.js

Let's add one fragment for the **Post** and one for the **Blog-Preview**.

If you are unfamiliar with the concepts of fragments in GraphQL checkout [this link](https://graphql.org/learn/queries/#fragments).

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

- Notice how we are using only strings that get exported?
  - We will use these strings inside the createPagesStatefully and not directly inside our React components. This gives us the freedom to define queries before the build starts and therefore we are able to concat multiple strings to one query string in the end.
  - This wouldn't be possible inside the components, when used with page queries or static queries.


### 2.) Adjust createPosts.js

**First part**

```javascript
// create/createPosts.js

const {
  PostTemplateFragment,
  BlogPreviewFragment,
} = require("../src/templates/post/data.js")

const { blogURI } = require("../globals")

const postTemplate = require.resolve("../src/templates/post/index.js")
const blogTemplate = require.resolve("../src/templates/post/blog.js")

const GET_POSTS = `
    # Here we make use of the imported fragments which are referenced above
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

// continues with second part
```

- We import the fragment strings and place them inside the `GET_PAGES` query string, but outside the actual `query GET_POSTS($first:Int $after:String) {...}`.
  - This will register the fragments, so they can be used in the upcoming query.
- Inside the query, you finally can make use of the standard `...` fragment syntax.

**Second Part**

```javascript
// create/createPosts.js

// Make sure to add the first part above

const allPosts = []
const blogPages = [];
let pageNumber = 0;
const itemsPerPage = 10;


/**
 * This is the export which Gatbsy will use to process.
 *
 * @param { actions, graphql }
 * @returns {Promise<void>}
 */
module.exports = async ({ actions, graphql, reporter }, options) => {

  /**
   * This is the method from Gatsby that we're going
   * to use to create posts in our static site.
   */
  const { createPage } = actions

  /**
   * Fetch posts method. This accepts variables to alter
   * the query. The variable `first` controls how many items to
   * request per fetch and the `after` controls where to start in
   * the dataset.
   *
   * @param variables
   * @returns {Promise<*>}
   */
  const fetchPosts = async (variables) =>
    /**
     * Fetch posts using the GET_POSTS query and the variables passed in.
     */
    await graphql(GET_POSTS, variables).then(({ data }) => {
      /**
       * Extract the data from the GraphQL query results
       */
      const {
        wpgraphql: {
          posts: {
            nodes,
            pageInfo: { hasNextPage, endCursor },
          },
        },
      } = data

      /**
       * Define the path for the paginated blog page.
       * This is the url the page will live at
       * @type {string}
       */
      const blogPagePath = !variables.after
        ? `${blogURI}/`
        : `${blogURI}/page/${pageNumber + 1}`

      /**
       * Add config for the blogPage to the blogPage array
       * for creating later
       *
       * @type {{
       *   path: string,
       *   component: string,
       *   context: {nodes: *, pageNumber: number, hasNextPage: *}
       * }}
       */
      blogPages[pageNumber] = {
        path: blogPagePath,
        component: blogTemplate,
        context: {
          nodes,
          pageNumber: pageNumber + 1,
          hasNextPage,
          itemsPerPage,
          allPosts,
        },
      }

      /**
       * Map over the posts for later creation
       */
      nodes
      && nodes.map((posts) => {
        allPosts.push(posts)
      })

      /**
       * If there's another post, fetch more
       * so we can have all the data we need.
       */
      if (hasNextPage) {
        pageNumber++
        reporter.info(`fetch post ${pageNumber} of posts...`)
        return fetchPosts({ first: itemsPerPage, after: endCursor })
      }

      /**
       * Once we're done, return all the posts
       * so we can create the necessary posts with
       * all the data on hand.
       */
      return allPosts
    })

  /**
   * Kick off our `fetchPosts` method which will get us all
   * the posts we need to create individual posts.
   */
  await fetchPosts({ first: itemsPerPage, after: null }).then((wpPosts) => {

    wpPosts && wpPosts.map((post) => {
      /**
       * Build post path based of theme blogURI setting.
       */
      const path = `${blogURI}/${post.uri}/`

      createPage({
        path: path,
        component: postTemplate,
        context: {
          post: post,
        },
      })

      reporter.info(`post created:  ${post.uri}`)
    })

    reporter.info(`# -----> POSTS TOTAL: ${wpPosts.length}`)

    /**
     * Map over the `blogPages` array to create the
     * paginated blog pages
     */
    blogPages
    && blogPages.map((blogPage) => {
      if (blogPage.context.pageNumber === 1) {
        blogPage.context.publisher = true;
        blogPage.context.label = blogPage.path.replace('/', '');
      }
      createPage(blogPage);
      reporter.info(`created blog archive page ${blogPage.context.pageNumber}`);
    });
  })
}
```

- Compared to the version from our previous parts, I did some constant name refactoring and added the `blogURI` coming from our globals file.


## Final Thoughts :checkered_flag:

Noooow, I hope I didn't miss anything. This part was a bit tricky, as I had to do some refactoring. I hope this did not mess up your setup. If so, please feel free to ask questions here or just clone this part of the tutorial to have a clean base for the next part.

Find the code base here: [https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-5](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-5)

## What's Next :arrow_right:

In the next part of the series, we will discover how to fetch images from WordPress in a way, that makes it possible to use the powerful [gatsby-image](https://using-gatsby-image.gatsbyjs.org/) plugin.

Coming up soon: **Part 6** - How to handle Images and make use of gatsby-image
