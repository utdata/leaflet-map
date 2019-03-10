# ICJ Project Template

A Node-based template system with a Gulp workflow set up for Github Pages publishing.

Features:

- [Bootstrap 4.1](https://getbootstrap.com/).
- [Sass](https://sass-lang.com/) with [autoprefixer](https://github.com/postcss/autoprefixer).
- Nunjucks templates with [`journalize`](https://www.npmjs.com/package/journalize) filters. Data and be made available to templates through the `project.config.json` file.
- Browsersync server.
- Image compression.
- Publishing to `docs/` for [Github Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/#publishing-your-github-pages-site-from-a-docs-folder-on-your-master-branch).

## Setup

If you are in my ICJ class, this is already done, but others will need to make sure you have gulp installed globally: `npm install -g gulp-cli`.

- Create your project folder, called `yourname-final`.
- Open VS Code into that folder and open the Terminal.
- Run `degit utdata/icj-project-template`.
- Create your Github repo and connect them.

## Understanding this project

Most of the files you edit for this project are in the `src` directory. The Gulp production process will generate the publishable files into the `docs` folder, which you shouldn't have to touch.

```pre
src
|-- assets
|   |-- img (Images go in here)
|-- js
|-- njk
|   |-- _data
|      |-- data.json (Any data to be parsed by Nunjucks)
|   |-- _layouts (Templates to be used by your pages)
|      |-- base.njk
|   |-- _partials (Reusable code for Templates)
|      |-- nav.njk
|   |-- index.njk (Pages that become your site)
|-- scss
|   |-- main.scss (Sass files and partials)
|   |-- _partials.scss
```

## Nunjucks templates

This project is configured to use Bootstrap 4 HTML/Sass as its core.

[Nunjucks](https://mozilla.github.io/nunjucks/templating.html) allows you to break your HTML into templates for the main structure of your site so you don't have to repeat that code for each page on your site.

Templates work off several basic concepts:

- Template inheritance using _extends_.
- _blocks_ which serve as variables or replaceable code.
- _include_ which pulls in code from other files.

With these tools, you can build a site framework once as a Layout, and then _extend_ or "use" that layout and all it's code, but swap out predefined _blocks_ specific to your new page.

### Layouts

**Layouts** and **partials** are parts of files used and extended elsewhere.

- The layout `src/njk/_layouts/base.njk` is an example base template for a site. The idea is to build the framework of the site only once, even though you have many pages.
- The layout `src/njk/_layouts/detail.njk` is an example of a layout that _extends_ the base layout, but then allows the user to insert different content through the _content_ block.
- Anything in `src/njk/_partials/` are snippets of code used by other layouts through a Nunjucks tag called _include_.

The Nunjucks community has adopted `.njk` as the file extension for templates. Because these files are inside a folder  and not at the `src/njk/` level, they do NOT become actual webpages on your site.

### Pages

All **pages** are kept in the root of the `src/njk/` folder. Each `.njk` file created here becomes an HTML page in `docs/`, and therefore a page on your website.

- The page `src/njk/index.njk` is the main website page that _extends_ `src/njk/_layouts/base.njk`. You are coding only the main content of the page, and inheriting all the nav and other framework.
- The page `src/njk/detail-page.njk` _extends_ the `src/njk/_layouts/detail.njk` layout, which is already extending `base.njk`. It allows you to have a different structure than the index file, yet still reuse it for many other pages.

### Using data in Nunjucks templates

Nunjucks has special [tags to apply logic](https://mozilla.github.io/nunjucks/templating.html#tags), like looping through data within templates.

Most data should be saved as key-value pairs or as an array in the `src/njk/_data/data.json`. An example might be this:

```json
  "books": [
    {
      "title": "The Clown",
      "author": "Heinrich Böll"
    },
    {
      "title": "The Shipping News",
      "author": "Annie Proulx"
    }
  ]
}
```

- You can access this data in a loop as `data.books.title`. There is an example in the `index.njk` file.
- You can add new `*.json` files into `src/njk/_data/` and they will be added to the Nunjucks context as `filename.arrayname.property`.
- You can also set global variables in `project.config.json` as key-value pairs or arrays.

> IMPORTANT: If you add/change/delete data in JSON files, you must re-run the `gulp dev` command to make it available to Nunjucks.

Have a spreadsheet of data that you need to convert to JSON? Try [csvjson.com](https://www.csvjson.com/csv2json).

### Filtering data for detail pages

It is possible to select a single node or "row" from an array in `data.json` by its position to use in a detail page using the Nunjucks [set](https://mozilla.github.io/nunjucks/templating.html#set) tag. The position order starts at zero, so using the data example above, you could access "The Shipping News" author (and similar properties) like this:

```html
{% set book = data.books[1] %}
<h1>{{ book.title }}</h1> # gets "The Shipping News" in data above
```

Using this method, you can create a single detail layout that can be extended to multiple detail pages, each using a single "row" from the JSON array. There is an example in `detail-page.html`.

### Sass/scss

The `src/scss/` folder holds all the SCSS files. It is configured for Bootstrap and gets compiled into the `docs` folder for publication.

There is also and example of a Sass partial with the `src/scss/_nav.scss` file, which is imported into `src/scss/main.scss`.

### Deployment

This project is designed to bundle the finished website into the `docs` folder, which can then be published anywhere you have a server.

By default, the `docs/` folder is committed to Github because we are using [Github Pages](https://help.github.com/categories/github-pages-basics/) for free hosting of our site.

Review [Github Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/#publishing-your-github-pages-site-from-a-docs-folder-on-your-master-branch) for specific directions on deployment.

## Technical bits on how this project is structured

### Gulp

Gulp is task runner and is configured in `gulpfile.js`. Individual tasks live in `tasks`.

- The default task `gulp` runs the `styles`, `lint`, `scripts`, `images` and `nunjucks` tasks to create the production files.
- Running `gulp dev` runs the default tasks above plus `serve` for the BrowserSync server.
- To run any specific gulp task: `gulp <name of task>`, e.g. `gulp clean`.

#### Tasks

- `clean.js`: Deletes the contents of the `docs` directory using [`del`](https://www.npmjs.com/package/del).
- `clear.js`: Clears out the gulp cache. Useful to reprocess images of the same name stuck in cache. Run `gulp clear` then re-run `gulp`.
- `images.js`: Optimize images using [`gulp-imagemin`](https://www.npmjs.com/package/gulp-imagemin) and [`imagemin-mozjpeg`](https://www.npmjs.com/package/imagemin-mozjpeg) packages.
- `lint.js`: Checks syntax of your (optionally ES6) javascript in `/src/js/` using [`gulp-eslint`](https://www.npmjs.com/package/gulp-eslint) -- it's a good idea to have an eslint package installed in your text editor of choice, as well.
- `nunjucks.js`: Builds out html pages using [`gulp-nunjucks-render`].(https://github.com/carlosl/gulp-nunjucks-render) (see notes below).
- `scripts.js`: Babel/concat/uglify javascript in `/src/js/` using [`gulp-babel`](https://www.npmjs.com/package/gulp-babel), [`gulp-concat`](https://www.npmjs.com/package/gulp-concat) and [`gulp-uglify`](https://www.npmjs.com/package/gulp-uglify).
- `serve.js`: Spins up a [BrowserSync](https://browsersync.io/docs/gulp) server at `localhost:3000`. Bundled with watch tasks for css/js/template changes.
- `styles.js`: Processes Sass files from `/src/scss/` into minified css using [`gulp-sass`](https://www.npmjs.com/package/gulp-sass), [`gulp-sourcemaps`](https://www.npmjs.com/package/gulp-sourcemaps), [`gulp-autoprefixer`](https://www.npmjs.com/package/gulp-autoprefixer) and [`gulp-cssnano`](https://www.npmjs.com/package/gulp-cssnano).

### More on Nunjucks

You can add [custom filters](https://mozilla.github.io/nunjucks/api.html#custom-filters) and [global variables](https://mozilla.github.io/nunjucks/api.html#addglobal) in the `manageEnv` function inside `tasks/nunjucks.js`.

A collection of functions useful for making prose reader friendly is already included with [`journalize`](https://www.npmjs.com/package/journalize).

### Future development

- I'd like to add a Nunjucks Markdown package of some sort to allow adding/editing of basic text in Markdown, perhaps with front-matter.
- I'd like to loop through data to create detail pages.
- I'd like to store everything in Google Docs and Sheets. Perhaps using Markdown in the Google docs.
