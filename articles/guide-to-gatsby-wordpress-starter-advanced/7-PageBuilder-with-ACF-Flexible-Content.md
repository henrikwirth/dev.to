---
published: false
title: "PageBuilder with ACF Flexible Content - Guide to Gatsby WordPress Starter Advanced with Previews, i18n and more"
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

- [Install WordPress Plugins :floppy_disk:](#install-wordpress-plugins-floppydisk)
- [Create ACF fields and Content :pencil2:](#create-acf-fields-and-content-pencil2)
   - [1.) Create Flexible Content field](#1-create-flexible-content-field)
   - [2.) Create Layouts inside the Flexible Content field](#2-create-layouts-inside-the-flexible-content-field)
   - [3.) Update your Pages](#3-update-your-pages)
   - [4.) Checkout the Schema](#4-checkout-the-schema)
- [Add layouts to Gatsby :newspaper:](#add-layouts-to-gatsby-newspaper)
   - [1.) Add Hero layout](#1-add-hero-layout)
   - [2.) Add TextBlock layout](#2-add-textblock-layout)
   - [3.) Add data to our page queries](#3-add-data-to-our-page-queries)
      - [Update pages data.js](#update-pages-datajs)
      - [Update createPages.js](#update-createpagesjs)
      - [Create utility function getAllLayoutsData](#create-utility-function-getalllayoutsdata)
   - [4.) Add AllLayouts component](#4-add-alllayouts-component)
   - [5.) Update Page template](#5-update-page-template)
- [Improve performance through Template-String technique :tophat:](#improve-performance-through-template-string-technique-tophat)
   - [1.) Add more utiliy functions](#1-add-more-utiliy-functions)
   - [2.) Add `lodash.uniqby` and `lodash.isempty`](#2-add-lodashuniqby-and-lodashisempty)
   - [3.) Add/update createPages.js](#3-addupdate-createpagesjs)
   - [4.) Add string based template](#4-add-string-based-template)
- [Explanation Video :movie_camera:](#explanation-video-moviecamera)
- [Final Thoughts :checkered_flag:](#final-thoughts-checkeredflag)
- [What's Next :arrow_right:](#whats-next-arrowright)


## Install WordPress Plugins :floppy_disk:

- [Advanced Custom Fields - PRO](https://www.advancedcustomfields.com/pro/)
- [WPGraphQL for Advanced Custom Fields](https://github.com/wp-graphql/wp-graphql-acf)

**Install Advanced Custom Fields PRO and activate your license**. We **need the PRO** version if we want to use the flexible content field.

You can download the .zip files of the `wp-graphql-acf` repository and install it through WP-Admin or just **navigate to your plugin folder** and do a `git clone` like so:

```shell
git clone https://github.com/wp-graphql/wp-graphql-acf
```

The WPGraphQL for Advanced Custom Fields is the plugin, that exposes the ACF fields to the GraphQL Schema. There is some good [documentation of it here](https://docs.wpgraphql.com/extensions/wpgraphql-advanced-custom-fields/).

## Create ACF fields and Content :pencil2:

We will add a flexible content field and 2 layouts to start with.

> **Note**: If you want to skip this parts 1.) and 2.), I prepared a `.json` export for the 2 layouts. Find it [right here](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/blob/tutorial/part-7/wordpress/acf-export.json).


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

## Add layouts to Gatsby :newspaper:

Let's start by adding our layout components. The folder structure for our layouts will look like so:

![Folder Structure](https://raw.githubusercontent.com/henrikwirth/dev.to/master/articles/guide-to-gatsby-wordpress-starter-advanced/images/07/folder-structure.png)

### 1.) Add Hero layout

```jsx
// src/layouts/Hero/Hero.js

import React from "react"
import FluidImage from "../../components/FluidImage"

const Hero = ({ image, text, textColor }) => {

  return (
    <section style={{ position: "relative" }}>
      <FluidImage image={image} style={{marginBottom: '15px'}}/>
      <p style={{
        color: textColor,
        position: "absolute",
        top: "50%",
        left: "50%",
        transform: "translate(-50%, -50%)",
        fontSize: '40px',
        textAlign: 'center',
        paddingTop:'80px',
        lineHeight: 1,
      }}>{text}</p>

    </section>
  )
}

export default Hero
```

- A simple React component with some styling.

```javascript
// src/layouts/Hero/Hero.data.js

module.exports = () => {
  return `
      ... on WPGraphQL_Page_Pagebuilder_Layouts_Hero {
          fieldGroupName
          image {
              sourceUrl
              altText
              imageFile {
                  childImageSharp {
                      fluid(maxHeight: 400, quality: 90, cropFocus: CENTER) {
                          ...GatsbyImageSharpFluid_tracedSVG
                      }
                  }
            }
          }
          text
          textColor
      }
  `
}
```

- `WPGraphQL_Page_Pagebuilder_Layouts_Hero` is the UnionType name of this layout. As we can have multiple layouts, GraphQL union types ([more here](https://graphql.org/learn/schema/#union-types)), help us to say: "If the data is of a certain type, then query with the following fields".
- As you can see this time we use a function to export the data. This will come in handy later, if we need to adjust the query, based on certain variables.

```javascript
// src/layouts/Hero/index.js

export { default } from './Hero';
```

- This simply helps us to make our component call a bit more elegant.
- Instead of importing like `import Hero from "../layouts/Hero/Hero"`, we can omit the last `/Hero` and do `import Hero from "../layouts/Hero` instead.

### 2.) Add TextBlock layout

```jsx
// src/layouts/TextBlock/TextBlock.js

import React from "react"

const TextBlock = ({ text, textColor, backgroundColor }) => {

  return (
    <section style={{backgroundColor: backgroundColor}}>
      <div style={{
        color: textColor,
        padding: '30px'
      }}>
        <div style={{
          color: textColor,
          textAlign: 'left',
        }} dangerouslySetInnerHTML={{__html: text}} />
      </div>

    </section>
  )
}

export default TextBlock
```

```javascript
// src/layouts/TextBlock/TextBlock.data.js

module.exports = () => {
  return `
      ... on WPGraphQL_Page_Pagebuilder_Layouts_TextBlock {
          fieldGroupName
          text
          textColor
          backgroundColor
      }
  `
}
```

```javascript
// src/layouts/TextBlock/index.js

export { default } from './TextBlock';
```

- TextBlock should be self-explanatory.

### 3.) Add data to our page queries

Now that we defined the data for the different layouts, we need to add it to our main query.

#### Update pages data.js

```javascript
// src/templates/page/data.js

const PageTemplateFragment = (layouts) => `
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
        pageBuilder {
            layouts {
                ${layouts}
            }
        }
    }
`

module.exports.PageTemplateFragment = PageTemplateFragment
```

- We update our data to be a function and pass it the layouts strings.
- This helps us to dynamically render all the queries into our main query.

#### Update createPages.js

```javascript
// create/createPages.js

const { getAllLayouts } = require("./utils")

... 1.)

const GET_PAGES = (layouts) => `
    ${FluidImageFragment}
    ${PageTemplateFragment(layouts)}

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

... 2.)

module.exports = async ({ actions, graphql, reporter }, options) => {

  /**
   * Get all layouts data as a concatenated string
   */
  const layoutsData = getAllLayoutsData()

... 3.)

  const fetchPages = async (variables) =>
    /**
     * Fetch pages using the GET_PAGES query and the variables passed in.
     */
    await graphql(GET_PAGES(layoutsData), variables).then(({ data }) => {

...      

```

> This snippet is not complete. Refer to this file ([createPages.js](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/blob/179b561fb83350964a17860ac11f6a323681eb9c/create/createPages.js)), if you want to see the whole file.

**I divided the file in 3 parts, where changes take place.**

1. We rewrite `GET_PAGES` to be a function and pass it `layouts`. Then we pass layouts further down to our template fragment like: `PageTemplateFragment(layouts)`.
2. In the second part we call our utility function `getAllLayoutsData()`. We will add this utility function in the next step.
3. In part 3 we change the `GET_PAGES` call to `GET_PAGES(layoutsData)` to pass down the layout data string.

#### Create utility function getAllLayoutsData

While we could import all the layouts data by hand and then combine the string before we pass it to our query, I wanted a way to **automate/abstract the creation of the query string**. Therefore this function **uses the glob pattern** to get all `layouts/**/*.data.js` and combines them to have a query ready string.

First, add the [glob module](https://www.npmjs.com/package/glob):

```shell
yarn add glob
```

Then add utils.js to our create folder:

```javascript
// create/utils.js

const path = require("path")

module.exports.getAllLayoutsData = () => {
  const glob = require("glob")

  let allLayoutsString = ""

  const fileArray = glob.sync("./src/layouts/**/*.data.js")

  fileArray.forEach(function(file) {
    let queryStringFunction = require(path.resolve(file))
    allLayoutsString = allLayoutsString + " \n " + queryStringFunction()
  })

  return allLayoutsString
}
```

- So we create a file array with the `glob.sync` call at first.
- Then, for each file, we require the file which essentially is an exported function (our `*.data.js`).
- At last, we concatenate all the strings and return `allLayoutsString`. This will hold all our layouts data query strings.

**I find, this comes in so handy!!!**


### 4.) Add AllLayouts component

Now we need a way to decide, which component should be rendered. To make this simple first, we introduce an AllLayouts component, that depending on the `fieldGroupName` (type) of the layout, decides what to render.

```jsx
// src/components/AllLayouts.js

import React from "react"
import Hero from "../layouts/Hero"
import TextBlock from "../layouts/TextBlock"

const AllLayouts = ({ layoutData }) => {

  const layoutType = layoutData.fieldGroupName

  /**
   * Default component
   */
  const Default = () => <div>In AllLayouts the mapping of this component is missing: {layoutType}</div>

  /**
   * Mapping the fieldGroupName(s) to our components
   */
  const layouts = {
    page_Pagebuilder_Layouts_Hero: Hero,
    page_Pagebuilder_Layouts_TextBlock: TextBlock,
    page_default: Default
  }

  /**
   * If layout type is not existing in our mapping, it shows our Default instead.
   */
  const ComponentTag = layouts[layoutType] ? layouts[layoutType] : layouts['page_default']

  return (
    <ComponentTag {...layoutData} />
  )
}

export default AllLayouts
```

- This component simply takes in the `layoutData` and depending on the `fieldGroupName` renders the component, that we have assigned in the mapping.
- **Note:** The fieldGroupName starts with a lowercase letter.

### 5.) Update Page template

Now that our data pipeline has been handled and we have a component, that dynamically renders the right component, we only need to add it to our template.

```jsx
// src/templates/page/index.js

import React from "react"

import Layout from "../../components/Layout"
import SEO from "../../components/SEO"
import AllLayouts from "../AllLayouts"


const Page = ({ pageContext }) => {
const {
  page: { title, pageBuilder },
} = pageContext

const layouts = pageBuilder.layouts || []

return (
  <Layout>
    <SEO title={title}/>
    <h1> {title} </h1>

    {
      layouts.map((layout, index) => {
        return <AllLayouts key={index} layoutData={layout} />
      })
    }

  </Layout>
)
}

export default Page
```

- We simply add the AllLayouts component and add the variable `pageBuilder` coming from our `pageContext`.
- `const layouts = pageBuilder.layouts || []` helps to have a fallback array, if there are no layouts defined. Then our mapping we just render to nothing and we don't end up with errors.
- Then we only need to map through our `layouts` and render the `AllLayouts` component, passing down the `layoutData`.

---

**Eh voilà, we have a page builder. We can define layouts/components in our WordPress backend with ACF and dynamically render React components, depending on the type.**

## Improve performance through Template-String technique :tophat:

> I won't explain too much detail in this written part. Please watch [the video]() for it.

One problem with our `AllLayouts` component we have, is that the way we implemented it now, on every page, **we import all layout components we have, even if we don't need them.**

This is fine if we have just a few components, but **imagine with have 50 different layouts in our page builder. We don't want them all imported on every page.**

For that we will use a little hacky, but in my opinion efficient way to **generate templates out of template strings**.

### 1.) Add more utiliy functions

```javascript
// create/utils.js

/**
 * Creates files based on a template string.
 *
 * @param {string} templateCacheFolderPath - Path where the temporary files should be saved.
 * @param {string} templatePath - Path to the file holding the template string.
 * @param {string} templateName - Name of the temporary created file.
 * @param {object[]} imports - An array of objects, that define the layoutType, componentName and filePath.
 * @returns {Promise<>}
 */
module.exports.createTemplate = ({ templateCacheFolderPath, templatePath, templateName, imports }) => {
  return new Promise((resolve) => {
    const fs = require("fs")

    const template = require(templatePath)
    const contents = template(imports)

    fs.mkdir(templateCacheFolderPath, { recursive: true }, (err) => {
      if (err) throw "Error creating template-cache folder: " + err

      const filePath = templateCacheFolderPath + "/" + templateName + ".js"

      fs.writeFile(filePath, contents, "utf8", err => {
        if (err) throw "Error writing " + templateName + " template: " + err

        console.log("Successfully created " + templateName + " template.")
        resolve()
      })
    })
  })
}

/**
 * Creates pages out of the temporary created templates.
 */
module.exports.createPageWithTemplate = ({ createTemplate, templateCacheFolder, pageTemplate, page, pagePath, mappedLayouts, createPage, reporter }) => {

  /**
   * First we create a new template file for each page.
   */
  createTemplate(
    {
      templateCacheFolderPath: templateCacheFolder,
      templatePath: pageTemplate,
      templateName: "tmp-" + page.uri,
      imports: mappedLayouts,
    }).then(() => {

    /**
     * Then, we create a gatsby page with the just created template file.
     */
    createPage({
      path: pagePath,
      component: path.resolve(templateCacheFolder + "/" + "tmp-" + page.uri + ".js"),
      context: {
        page: page,
      },
    })

    reporter.info(`page created: ${pagePath}`)
  })
}
```

- Most of the things should be explained with the comments I added. Basically we create new template files based on a template-string file.
- Then we create the pages, based on that generated file.


### 2.) Add `lodash.uniqby` and `lodash.isempty`

```shell
yarn add lodash.uniqby lodash.isempty
```

### 3.) Add/update createPages.js

First the top part:

```javascript
// create/createPages.js

const _uniqBy = require("lodash.uniqby")
const _isEmpty = require("lodash.isempty")

const { getAllLayoutsData, createTemplate, createPageWithTemplate } = require("./utils")

const filePathToComponents = "../src/layouts/"
const templateCacheFolder = ".template-cache"
const layoutMapping = require("./layouts")
const pageTemplate = require.resolve("../src/templates/page/template.js")
```

Then further down:

```javascript
// create/createPages.js

wpPages && wpPages.map((page) => {
      let pagePath = `/${page.uri}/`

      /**
       * If the page is the front page, the page path should not be the uri,
       * but the root path '/'.
       */
      if (page.isFrontPage) {
        pagePath = "/"
      }

      /**
       * Filter out empty objects. This can happen, if for some reason you
       * don't query for a specific layout (UnionType), that is potentially
       * there.
       */
      const layouts = page.pageBuilder.layouts.filter(el => {
        return !_isEmpty(el)
      })

      let mappedLayouts = []

      if (layouts && layouts.length > 0) {
        /**
         * Removes all duplicates, as we only need to import each layout once
         */
        const UniqueLayouts = _uniqBy(layouts, "fieldGroupName")

        /**
         * Maps data and prepares object for our template generation.
         */
        mappedLayouts = UniqueLayouts.map((layout) => {
          return {
            layoutType: layout.fieldGroupName,
            componentName: layoutMapping[layout.fieldGroupName],
            filePath: filePathToComponents + layoutMapping[layout.fieldGroupName],
          }
        })
      }

      createPageWithTemplate({
        createTemplate: createTemplate,
        templateCacheFolder: templateCacheFolder,
        pageTemplate: pageTemplate,
        page: page,
        pagePath: pagePath,
        mappedLayouts: mappedLayouts,
        createPage: createPage,
        reporter: reporter,
      })
    })
```

### 4.) Add string based template

In JavaScript you can use template strings to pass down arguments or logic in general to your string. We make use of that to literally create a template based on our `src/templates/page/index.js` component.

```
module.exports = (imports) => {
  return`
// This is a temporary generated file. Changes to this file will be overwritten eventually!
import React from "react"

import Layout from "../src/components/Layout"
import SEO from "../src/components/SEO"

// Sections
${imports.map(({ componentName, filePath }) => `import ${componentName} from '${filePath}';`).join('\n')}

const Page = ({ pageContext }) => {
  const {
    page: { title, pageBuilder },
  } = pageContext

  const layouts = pageBuilder.layouts || []

  return (
    <Layout>
      <SEO title={title}/>
      <h1> {title} </h1>

      {
        layouts.map((layout, index) => {
          ${imports.map(({ componentName, layoutType }) => {
            return `
              if (layout.fieldGroupName === '${layoutType}') {
                  return <${componentName} {...layout} key={index} />;
              }
            `
          }).join('\n')}
        })
      }

    </Layout>
  )
}

export default Page
  `
}
```

- What we do here, is to decide **what components actually get imported**. We only want to import components, that are part of the created page. Not more not less.

## Explanation Video :movie_camera:

This tutorial can be a lot to take in and it is hard for me to write a super detailed explanation of everything down. So I decided to record a video, where I would go through the steps mentioned in this tutorial and verbally explain what my thoughts to the process are.

I hope this will make it more accessible for everyone.

{% youtube somecode %}


## Final Thoughts :checkered_flag:

Checkout how [the site](https://gatsby-starter-wordpress-advanced.netlify.com/) looks now:

...

Find the code base here: [https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-7](https://github.com/henrikwirth/gatsby-starter-wordpress-advanced/tree/tutorial/part-7)


## What's Next :arrow_right:


Coming up soon: **Part 8** - Internationalization
