---
title: Getting started with Gatsby
date: '2019-01-02'
---

## Creating a new project

Make sure you have gatsby CLI globally installed in you machine

If not:

```sh
npm install --global gatsby-cli
```

Now create a folder for a project, for example `blogs` and inside that folder create a new gatsby project with the following command:

```sh
gatsby new SITE_NAME_HERE
```

Make sure to change SITE_NAME_HERE with a custom name for your website

Now that you have your new project built, head to that folder and run

```sh
gatsby develop
```

By running `gatsby develop` you'll start a live server in the port 8000, where you'll be able to see your project up and running

Access http://localhost:8000 and you'll se something similar to this:

![Hello Gatsby](/img/Hello&#32;Gatsby.png)



## Working with styles and using Saas

We'll need to install a gatsby plugin to start working with Sass to style our static site.

To do this, in your project folder let's install the `gatsby-plugin-sass`

```sh
npm install --save gatsby-plugin-sass
```

After we installed the gatsby pluging, we need to explicitly add this to the plugins list in the `gatsby-config.js` file.

By default, this plugin requires `node-sass` package to work properly. So let's install it

```sh
npm install --save node-sass
```

Perfect! The plugins are good to go, and now we'll be able to use Sass inside our project

> It's important to note that if you had your server running (you probably had) while installing the plugins you'll have to restart your server in order for the plugin to work. Simply using `Ctrl+C` and then running `gatsby develop` again

So to test if our plugin is working as expected, let's change or css file to a scss file.

In my project, this file we are going to use for this firts test is located at:

```
|-- src
|   |-- components
|   |   |-- header.js
|   |   |-- image.js
|   |   |-- layout.js
|   |   |-- layout.scss ( <<< --- THIS FILE RIGHT HERE )
|   |   `-- seo.js
```



Inside components folder you'll probably have a file called `layout.css` or `index.css`. Something relating to stylesheet file. Change the extension from `.css` to `.scss`. All you have to do is find where this stylesheet file is being imported, in my case is inside the `layout.js` file

![Files](/img/Files.png)

Inside `layout.js` file you'll probably have a line importing our style file

```javascript
import './layout.css';

// simply change it to 

import './layout.scss';
```

Your server will reload, and if everything in the page remains the same, everything is working perfectly

## Building a blog

### Building a blog with markdown and graphQL

First things first, we will start by installing a source plugin in our gatsby project. Source plugins are used to create file nodes from our file system. We will use this because we'll be pulling files from our project to get our data, instead of an external API.

So let's install this source plugin:

```sh
npm install --save gatsby-source-filesystem
```

We then need to configure this plugin inside our `Gatsby-config.js` file

We'll initialize our plugin in a sort of different way than we did with our sass plugin.

So let's open an object `{}` and start our configuration:

```javascript
{
      resolve: 'gatsby-source-filesystem',
      options: {
        path: `${__dirname}/src/pages`,
        name: 'pages',
      },
    },
}
```

For `resolve` we pass the name of our installed plugin: `gatsby-source-filesystem`

We also have an `options object` that we'll configure the path of our file, where we'll store our blog posts, and a name for the plugin

Now we may install another plugin, that will allow us to query our data from our Markdown files

```sh
npm install --save gatsby-transformer-remark
```

This one won't require too much configuration, so let's just list it in our plugins.

Now let's create our folder inside the `pages` folder to keep our blog post's files.

I'll use a simple convention of naming the folder of our blog post with the date this post is being published and the name of the post. So our folder will be: `12-28-2018-first-post`. How you'll name the folder won't make too much difference, if you feel comfortable naming a differente way, just stick to it.

Inside our post folder, create a file called `index.md` that will contain the content of our post

```markdown
---
path: '/first-post'
title: 'First blog post'
---

# Hello everyone!
This is our first blog post, not exciting, sorry! 😢😢😢
```

This is our first post, just an example. 

One important thing I want to point to is the first 4 lines of this file. By opening with `---` and closing with `---` we created a special block called `FrontMatter`. This block allows us to set a number of variables in our file that we will be able to access once we querying our data.

Unfortunately we won't be able to see our blog post if we simply access http://localhost:8000/first-post because we still need to do some further configuration. If you try to access it, you'll get a 404 error:

![404 Error](/img/404&#32;Error.png)

So, let's begin by creating a template for our blog posts. Inside `src` folder, create a new folder called `templates` and inside it create a file called `post.js`. We will use this file as a default template that our blog posts will be rendered in.

In `post.js` let's create a React component:

```javascript
import React from 'react'
import Helmet from 'react-helmet'

export default function({data}) {
    const { markdownRemark: post } = data;
    
    return (
		<div>
        	<h1>{post.frontmatter.title}</h1>
        </div>
    )
}
```

What we are doing here is, we are exporting a function that will receive a parameter called `data`. This `data` parameter will bring data brought by our graphql query.

With this, we are able to return a simple `html` structure rendering, for example, our post title. `<h1>post.frontmatter.title}</h1>`. 

But before we continue, let's create our graphql query that will pull the data from our markdown file.

```javascript
export const postQuery = graphql`
query BlogPostByPath($path: String!) {
  markdownRemark(frontmatter: { path: { eq: $path } }) {
    html
    frontmatter {
      path
      title
    }
  }
}
`;
```

Perfect! Just to check things up, your `post.js` file should look like this by now:

```javascript
import React from 'react';
import Helmet from 'react-helmet'
import { graphql } from 'gatsby';

export default function({data}) {
  const { markdownRemark: post } = data;

  return (
    <div>
      <h1>{post.frontmatter.title}</h1>
    </div>
  );
}

export const postQuery = graphql`
query BlogPostByPath($path: String!) {
  markdownRemark(frontmatter: { path: { eq: $path } }) {
    html
    frontmatter {
      path
      title
    }
  }
}
`;

```

For now, gatsby still don't know that our markdown files should be considered as pages, and then rendered in our site, so we'll need to tell gatsby to generate pages based on our markdown files, so we'll be able to see our blog posts

To do this, we need to create a file in the root of our project called `gatsby-node.js` (if you already have one, simply edit it). 

We'll need to use the `path` library, so let's import it

```javascript
const path = require('path');
```

And from this file we'll export a function called `createPages` that is part of Gatsby's API.

```javascript
exports.createPages = ({actions, graphql}) => {
  const { createPage } = actions;

  // bringing our template
  const postTemplate = path.resolve('src/templates/post.js');

  // querying all our blog posts (with graphQL)
  return graphql(`{
    allMarkdownRemark {
      edges {
        node {
          html
          id
          frontmatter {
            path
            title
          }
        }
      }
    }
  }`)
  .then(res => {
    if (res.errors) {
      return Promise.reject(res.errors);
    }

    // iterating over our pages
    res.data.allMarkdownRemark.edges.forEach(({node}) => {
      createPage({
        path: node.frontmatter.path,
        component: postTemplate,
      })
    })
  })
}
```

Let's start to break down the code above.

We start by passing to parameters to the `createPages()` function. 

- actions: As gatsby uses redux to manage state, we use this `actions` so (equivalent to boundActionCreators in Redux) to manipulate the state of our components in our gatsby application. This `actions` object contains a whole set of functions that you can use to manipulate state. In this case we'll be using `createPage()`.
- graphql: As we did before, we'll use graphql to query through our files to list all markdown files (used as blog posts) and create our pages from these files. 

So, as I just mentioned above, let's "extract" the `createPage()` function from our `actions`, using ES6 destructuring.

```javascript
const { createPage } = actions;
```

Let's also pull our post template from our templates folder:

```javascript
const postTemplate = path.resolve('src/templates/post.js')
```

Now we can start querying our markdown files containing our blog posts' content:

```javascript
return graphql(`{
    allMarkdownRemark {
      edges {
        node {
          html
          id
          frontmatter {
            path
            title
          }
        }
      }
    }
  }`)
```

And treat any sort of error that might occur:

```javascript
.then(res => {
    if (res.errors) {
      return Promise.reject(res.errors);
    }
```

If no error happens, then we are able to loop over our pulled files and create the pages from them, using `createPage()` function:

```javascript
res.data.allMarkdownRemark.edges.forEach(({node}) => {
      createPage({
        path: node.frontmatter.path,
        component: postTemplate,
```

In this case we'll need to specify to parameters to `createPage()` function.

- path: The path of the page. If you remember well we set this in our frontmatter inside our markdown file:

  - ```markdown
    ---
    path: '/first-post'
    title: 'First blog post'
    ---
    ```

  - It's important to note that's is mandatory that the path is a string starting with a forward slash ('/').

- component: This is the react component that will be used to render the blog post. In our case we are talking about the `post.js` file we created inside the `templates` folder

Perfect! 

Now, if you save this and start your server again, acces the address http://localhost:8000/first-post you'll be able to see our first post:

![First Post](/img/First&#32;Post.png)