# able-gatsby-starter

## Gatsby's markdown blog starter with Able content webhooks

This is a base project that can be used to get started with a Gatsby markdown blog integrated with [Able](https://able.bio) content webhooks.

Want to customize the Gatsby part? Head over to their [gatsby-starter-blog](https://www.gatsbyjs.org/docs/gatsby-starters/) repo and amazing [documentation](https://www.gatsbyjs.org/docs).

Want to know more about the how the Able webhooks tie in all this and updates this repo with new content. Head over to the [able-webhook-handler](https://github.com/drenther/able-webhook-handler) express app repo.

## Getting Started

To get started first fork (by clicking the fork button on top right corner of this Github repo) and clone the repo

```
git clone https://github.com/drenther/able-gatsby-starter
```

Then, `cd` into the repository root directory and install dependencies

```
npm install
```

Once the dependencies are installed, you can run 

```
npm start
```

to start a development server and listens to changes in the files to auto update the build

### Updating metadata

To use this as the base for your own blog. You can start by updating the metadata for the site in the `gatsby-config.js` file.

Update the following `title`, `author.name`, `author.summary`, `description`, `siteUrl` and `social` field values to your own.

Then, update the asset files like the website logo and author profile pic in `content/assets` directory.

Search for reference of the old asset filenames across the project repo and replace them with your updated asset filenames.

For example, if you replaced `profile_pic.jpeg` with `my_photo.png`, make sure to replace all references of `profile_pic.jpeg` in the repo code to `my_photo.png`.

### Updating the base content

The current repo will have post files written in markdown under the `content/blog` directory. Delete them to start anew with your own posts.

## Deployment

You can use something like [Netlify](https://www.netlify.com/) or [Vercel](https://vercel.com/) to deploy a gatsby project like this. 

For auto updating blog enable auto deployment on push to master in your hosting service. This will enable a new gatsby build to be pushed out whenever a new commit is made to this repo by you manually or by the able webhook consumer API.