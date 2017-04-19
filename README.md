# Rosid

[![Travis Build Status](https://travis-ci.org/electerious/Rosid.svg?branch=master)](https://travis-ci.org/electerious/Rosid) [![Coverage Status](https://coveralls.io/repos/github/electerious/Rosid/badge.svg?branch=master)](https://coveralls.io/github/electerious/Rosid?branch=master) [![Dependencies](https://david-dm.org/electerious/Rosid.svg)](https://david-dm.org/electerious/Rosid#info=dependencies) [![Donate via Flattr](https://img.shields.io/badge/flattr-donate-009cde.svg)](https://flattr.com/profile/electerious) [![Donate via PayPal](https://img.shields.io/badge/paypal-donate-009cde.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CYKBESW577YWE)

Just-in-time development server and static site generator written in [Node.js](https://nodejs.org/). Rosid invokes functions before serving files to the browser. This allows you to transform anything on-the-fly, without saving.

## Contents

- [Description](#description)
	- [What is Rosid?](#what-is-rosid)
	- [Why Rosid?](#why-rosid)
	- [How does it work?](#how-does-it-work)
- [Requirements](#requirements)
- [Setup](#setup)
- [Routes](#routes)
	- [Name](#name)
	- [Path](#path)
	- [Handler](#handler)
	- [Opts](#opts)
- [Handlers](#handlers)
- [Execute](#execute)
	- [Initialize](#initialize)
	- [Serve](#serve)
	- [Compile](#compile)
	- [CLI](#cli)
- [Options](#options)
- [Tips](#tips)

## Description

### What is Rosid?

Rosid is a framework that focus on two features:

1. A **development server with live reloading**, which transforms files as soon as you request them.
2. A **static site generator**, which transforms files using defined transform functions.

### Why Rosid?

- It doesn't force you to use a defined directory structure
- It's built on popular modules like [Browsersync](https://www.browsersync.io)
- It integrates nicely with tools you are are already using to transform your files (e.g. [Gulp](http://gulpjs.com), [Grunt](http://gruntjs.com) or Vanilla JS)
- It's lightweight and only includes what it really needs
- Transformed files don't need to be saved along their source files
- It lets you compile code to static files to host them anywhere

### How does it work?

Rosid starts a server and compares requested URLs with user-defined patterns. An associated file handler will be executed when a pattern matches. The handler receives information about the request and can transform the file, which will be sent to the browser.

## Requirements

Rosid depends on...

- [Node.js](https://nodejs.org/en/) (v6.2.0 or newer)
- [npm](https://www.npmjs.com)

Make sure to install and update all dependencies before you setup Rosid.

## Setup

Install Rosid using [npm](https://npmjs.com).

```sh
npm install rosid
```

## Routes

The route-configuration is an array of objects. Each object must contain a name, path and handler.

```js
const routes = [
	{
		name    : 'JS',
		path    : 'assets/scripts/**/*.js',
		handler : 'rosid-handler-js'
	},
	{
		name    : 'SCSS',
		path    : 'assets/styles/**/[^_]*.{css,scss}',
		handler : transfromSCSS,
		opts    : { custom: 'data' }
	},
	{
		name    : 'EJS',
		path    : '[^_]*.{html,ejs}',
		handler : require('rosid-handler-ejs')
	}
]
```

Store the routes in a variable or save them in a JSON file called `rosidfile.json`. This file must be placed in your current working directory when using the [CLI](#cli). Here's how it might look like:

```json
[
	{
		"name"    : "JS",
		"path"    : "assets/scripts/**/[^_]*.js",
		"handler" : "rosid-handler-js"
	},
	{
		"name"    : "SASS",
		"path"    : "assets/styles/[^_]*.{css,sass}",
		"handler" : "rosid-handler-sass"
	},
	{
		"name"    : "EJS",
		"path"    : "[^_]*.{html,ejs}",
		"handler" : "rosid-handler-ejs"
	}
]
```

### Name

Type: `String` Default: `null` Optional: `false`

Name of the route.

### Path

Type: `String` Default: `null` Optional: `false`

Rosid compares all requested URLs (when running the [development server](#serve)) and all existing files (when [compiling a project](#compile)) with the path. It executes the handler when the pattern matches. The path must be a relative. Query strings will be ignored. Rosid uses the same [patterns the shell uses](https://github.com/isaacs/node-glob).

### Handler

Type: `Function|String` Default: `null` Optional: `false` Signature: `filePath, opts`

Should be a function which transforms and returns the content of a file. When a string is specified, Rosid tries to require the given module. [More about handlers...](#handlers)

### Opts

Type: `Object` Default: `{}` Optional: `true`

A save place to store route-specific properties, settings or data. All data is accessible inside the corresponding handler. It's the second parameter passed to the handler.

You can choose your property names freely. Only the following names are reserved by Rosid:

- Rosid automatically appends a `optimize` property. `optimize` is `false` when running the [development server](#serve) *or* `true` when [compiling a project](#compile). This option can be used to optimize the output of handlers depending on how Rosid has been executed. Set a custom `optimize` to use your option instead.
- Each handler has a predefined load (`in`) and save (`out`) extension. Specify a custom extension to overwrite the default of the handler.

`rosid-handler-sass` loads `.sass` files by default and only optimizes the output when necessary. The following `rosidfile.json` shows reserved options in action, forces a optimization and sets the `in` extension to `.scss`.

```json
[
	{
		"name"    : "SASS",
		"path"    : "assets/styles/[^_]*.{css,scss}",
		"handler" : "rosid-handler-sass",
		"opts"    : {
			"in"       : ".scss",
			"optimize" : true
		}
	}
]
```

## Handlers

Handlers are functions which load and transform files. Rosid doesn't care about how they transform them, but requires them to return a promise. The promise should resolve a string that contains the content of the transformed file.

Existing handlers:

| Type | Description | Link |
|:-----------|:------------|:------------|
| JS => JS | Load, transform, bundle and compress JS. | [GitHub](https://github.com/electerious/rosid-handler-js) |
| JS => HTML | Load JS and transform to HTML. | [GitHub](https://github.com/electerious/rosid-handler-node) |
| SASS => CSS | Load SASS and transform to CSS, add vendor prefixes and minify. | [GitHub](https://github.com/electerious/rosid-handler-sass) |
| EJS => HTML | Load EJS templates and render them. | [GitHub](https://github.com/electerious/rosid-handler-ejs) |
| Nunjucks => HTML | Load Nunjucks templates and render them. | [GitHub](https://github.com/electerious/rosid-handler-njk) |

Example:

```js
/*
 * The following handler transforms SASS to CSS.
 */
const transfromSASS = function(filePath, opts) {

	/*
	 * 1. Load requested file (filePath)
	 * 2. Transform the file
	 * 3. Return the transformed contents of the file
	 */

	return Promise.resolve(`
		.css { display: none; }
	`)

}
```

Parameters:

- `filePath` `{String}` Absolute path to file.
- `route` `{Object}` Options from the route.

Returns:

- `{Promise}({String|Buffer})` The transformed file content.

## Execute

### Initialize

You must require and initialize Rosid before you can use the `serve` and `compile` functions.

Syntax:

```js
Rosid = require('rosid')(routes)
```

Parameters:

- `routes` `{Array}` An array of [routes](#routes).

### Serve

Start a static site server and compile requested files on-the-fly. The site will reload automatically when files change.

Syntax:

```js
Rosid.serve(srcPath, opts, callback)
```

Example:

```js
Rosid.serve('src/', (err) => {})
```

Parameters:

- `srcPath` `{String}` Path to the folder containing your site and untransformed files.
- `opts` `{?Object}` An object of [options](#options).
- `callback` `{?Function}`
	- `err` `{?Error}`

### Compile

Export your site to a folder.

Syntax:

```js
Rosid.compile(srcPath, distPath, opts, callback)
```

Example:

```js
Rosid.compile('src/', 'dist/', (err) => {})
```

Parameters:

- `srcPath` `{String}` Path to the folder containing your site and untransformed files.
- `distPath` `{String}` Path where Rosid should save your site and transformed files. The folder is automatically created and is assumed to be empty.
- `opts` `{?Object}` An object of [options](#options).
- `callback` `{?Function}`
	- `err` `{?Error}`

### CLI

Rosid can be used as a library or as a command line utility. The tool is located in the `bin` folder and allows you to run the `serve` and `compile` functions without adding JS files to your project. Only a single `rosidfile.json` is required in your current working directory. Execute `rosid --help` for additional information.

## Options

If you want more control over the `serve` or `compile` function, pass an object with the following parameters to them:

```js
{
	/*
	 * Option for the copy-module which will only run when compiling your site.
	 * [] = Ignore the following files when copying.
	 *      Must be an array of strings, which will be matched against absolute paths.
	 */
	ignore: [],
	/*
	 * Increase verbosity.
	 * true  = Log additional messages.
	 * false = Only log important messages.
	 */
	verbose: false,
	/*
	 * Option for the deliver-module which will only run when serving your site.
	 * It is typically necessary to set this to true to successfully watch files over a network,
	 * and it may be necessary to successfully watch files in other non-standard situations.
	 * true  = Use fs.watch
	 * false = Use fs.watchFile (backed by polling)
	 */
	polling: false,
	/*
	 * Decide if Rosid should open the URL automatically in your default browser.
	 * true  = Open URL
	 * false = Don't open URL
	 */
	open: true
}
```

## Tips

- Install Rosid without optional dependencies using npm's `--no-optional` flag. This speeds up the installation and skips a lot of dependencies. It's perfect when used in production. The downside: Running the `serve` function isn't possible anymore.
