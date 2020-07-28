# node-sass

#### Supported Node.js versions vary by release, please consult the [releases page](https://github.com/sass/node-sass/releases). Below is a quick guide for minimum support:

NodeJS  | Minimum node-sass version | Node Module
--------|--------------------------|------------
Node 14 | 4.14+                    | 83
Node 13 | 4.13+                    | 79
Node 12 | 4.12+                    | 72
Node 11 | 4.10+                    | 67
Node 10 | 4.9+                     | 64
Node 8  | 4.5.3+                   | 57

<table>
  <tr>
    <td>
      <img width="77px" alt="Sass logo" src="https://rawgit.com/sass/node-sass/master/media/logo.svg" />
    </td>
    <td valign="bottom" align="right">
      <a href="https://www.npmjs.com/package/node-sass">
        <img width="100%" src="https://nodei.co/npm/node-sass.png?downloads=true&downloadRank=true&stars=true">
      </a>
    </td>
  </tr>
</table>

[![Build Status](https://travis-ci.org/sass/node-sass.svg?branch=master&style=flat)](https://travis-ci.org/sass/node-sass)
[![Build status](https://ci.appveyor.com/api/projects/status/22mjbk59kvd55m9y/branch/master?svg=true)](https://ci.appveyor.com/project/sass/node-sass/branch/master)
[![npm version](https://badge.fury.io/js/node-sass.svg)](http://badge.fury.io/js/node-sass)
[![Dependency Status](https://david-dm.org/sass/node-sass.svg?theme=shields.io)](https://david-dm.org/sass/node-sass)
[![devDependency Status](https://david-dm.org/sass/node-sass/dev-status.svg?theme=shields.io)](https://david-dm.org/sass/node-sass#info=devDependencies)
[![Coverage Status](https://coveralls.io/repos/sass/node-sass/badge.svg?branch=master)](https://coveralls.io/r/sass/node-sass?branch=master)
[![Inline docs](http://inch-ci.org/github/sass/node-sass.svg?branch=master)](http://inch-ci.org/github/sass/node-sass)
[![Join us in Slack](https://libsass-slack.herokuapp.com/badge.svg)](https://libsass-slack.herokuapp.com/)

Node-sass is a library that provides binding for Node.js to [LibSass], the C version of the popular stylesheet preprocessor, Sass.

It allows you to natively compile .scss files to css at incredible speed and automatically via a connect middleware.

Find it on npm: <https://www.npmjs.com/package/node-sass>

Follow @nodesass on twitter for release updates: <https://twitter.com/nodesass>

## Install

```shell
npm install node-sass
```

Some users have reported issues installing on Ubuntu due to `node` being registered to another package. [Follow the official NodeJS docs](https://github.com/nodesource/distributions/blob/master/README.md#debinstall) to install NodeJS so that `#!/usr/bin/env node` correctly resolves.

Compiling on Windows machines requires the [node-gyp prerequisites](https://github.com/nodejs/node-gyp#on-windows).

Are you seeing the following error? Check out our [Troubleshooting guide](https://github.com/sass/node-sass/blob/master/TROUBLESHOOTING.md#installing-node-sass-4x-with-node--4).**

```
SyntaxError: Use of const in strict mode.
```

**Having installation troubles? Check out our [Troubleshooting guide](https://github.com/sass/node-sass/blob/master/TROUBLESHOOTING.md).**

### Install from mirror in China

```shell
npm install -g mirror-config-china --registry=http://registry.npm.taobao.org
npm install node-sass
```

## Usage

```javascript
var sass = require('node-sass');
sass.render({
  file: scss_filename,
  [, options..]
}, function(err, result) { /*...*/ });
// OR
var result = sass.renderSync({
  data: scss_content
  [, options..]
});
```

## Options

### file

* Type: `String`
* Default: `null`

**Special**: `file` or `data` must be specified

Path to a file for [LibSass] to compile.

### data

* Type: `String`
* Default: `null`

**Special**: `file` or `data` must be specified

A string to pass to [LibSass] to compile. It is recommended that you use `includePaths` in conjunction with this so that [LibSass] can find files when using the `@import` directive.

### importer (>= v2.0.0) - _experimental_

**This is an experimental LibSass feature. Use with caution.**

* Type: `Function | Function[]` signature `function(url, prev, done)`
* Default: `undefined`

Function Parameters and Information:

* `url (String)` - the path in import **as-is**, which [LibSass] encountered
* `prev (String)` - the previously resolved path
* `done (Function)` - a callback function to invoke on async completion, takes an object literal containing
  * `file (String)` - an alternate path for [LibSass] to use **OR**
  * `contents (String)` - the imported contents (for example, read from memory or the file system)

Handles when [LibSass] encounters the `@import` directive. A custom importer allows extension of the [LibSass] engine in both a synchronous and asynchronous manner. In both cases, the goal is to either `return` or call `done()` with an object literal. Depending on the value of the object literal, one of two things will happen.

When returning or calling `done()` with `{ file: "String" }`, the new file path will be assumed for the `@import`. It's recommended to be mindful of the value of `prev` in instances where relative path resolution may be required.

When returning or calling `done()` with `{ contents: "String" }`, the string value will be used as if the file was read in through an external source.

Starting from v3.0.0:

* `this` refers to a contextual scope for the immediate run of `sass.render` or `sass.renderSync`

* importers can return error and LibSass will emit that error in response. For instance:

  ```javascript
  done(new Error('doesn\'t exist!'));
  // or return synchronously
  return new Error('nothing to do here');
  ```

* importer can be an array of functions, which will be called by LibSass in the order of their occurrence in array. This helps user specify special importer for particular kind of path (filesystem, http). If an importer does not want to handle a particular path, it should return `null`. See [functions section](#functions--v300---experimental) for more details on Sass types.

### includePaths

* Type: `Array<String>`
* Default: `[]`

An array of paths that [LibSass] can look in to attempt to resolve your `@import` declarations. When using `data`, it is recommended that you use this.

### indentedSyntax

* Type: `Boolean`
* Default: `false`

`true` values enable [Sass Indented Syntax](https://sass-lang.com/documentation/file.INDENTED_SYNTAX.html) for parsing the data string or file.

__Note:__ node-sass/libsass will compile a mixed library of scss and indented syntax (.sass) files with the Default setting (false) as long as .sass and .scss extensions are used in filenames.

### indentType (>= v3.0.0)

* Type: `String`
* Default: `space`

Used to determine whether to use space or tab character for indentation.

### indentWidth (>= v3.0.0)

* Type: `Number`
* Default: `2`
* Maximum: `10`

Used to determine the number of spaces or tabs to be used for indentation.

### linefeed (>= v3.0.0)

* Type: `String`
* Default: `lf`

Used to determine whether to use `cr`, `crlf`, `lf` or `lfcr` sequence for line break.

### omitSourceMapUrl

* Type: `Boolean`
* Default: `false`

**Special:** When using this, you should also specify `outFile` to avoid unexpected behavior.

`true` values disable the inclusion of source map information in the output file.

### outFile

* Type: `String | null`
* Default: `null`

**Special:** Required when `sourceMap` is a truthy value

Specify the intended location of the output file. Strongly recommended when outputting source maps so that they can properly refer back to their intended files.

**Attention** enabling this option will **not** write the file on disk for you, it's for internal reference purpose only (to generate the map for example).

Example on how to write it on the disk

```javascript
sass.render({
    ...
    outFile: yourPathTotheFile,
  }, function(error, result) { // node-style callback from v3.0.0 onwards
    if(!error){
      // No errors during the compilation, write this result on the disk
      fs.writeFile(yourPathTotheFile, result.css, function(err){
        if(!err){
          //file written on disk
        }
      });
    }
  });
});
```

### outputStyle

* Type: `String`
* Default: `nested`
* Values: `nested`, `expanded`, `compact`, `compressed`

Determines the output format of the final CSS style.

### precision

* Type: `Integer`
* Default: `5`

Used to determine how many digits after the decimal will be allowed. For instance, if you had a decimal number of `1.23456789` and a precision of `5`, the result will be `1.23457` in the final CSS.

### sourceComments

* Type: `Boolean`
* Default: `false`

`true` Enables the line number and file where a selector is defined to be emitted into the compiled CSS as a comment. Useful for debugging, especially when using imports and mixins.

### sourceMap

* Type: `Boolean | String | undefined`
* Default: `undefined`

Enables source map generation during `render` and `renderSync`.

When `sourceMap === true`, the value of `outFile` is used as the target output location for the source map with the suffix `.map` appended. If no `outFile` is set, `sourceMap` parameter is ignored.

When `typeof sourceMap === "string"`, the value of `sourceMap` will be used as the writing location for the file.

### sourceMapContents

* Type: `Boolean`
* Default: `false`

`true` includes the `contents` in the source map information

### sourceMapEmbed

* Type: `Boolean`
* Default: `false`

`true` embeds the source map as a data URI

### sourceMapRoot

* Type: `String`
* Default: `undefined`

the value will be emitted as `sourceRoot` in the source map information

## `render` Callback (>= v3.0.0)

node-sass supports standard node style asynchronous callbacks with the signature of `function(err, result)`. In error conditions, the `error` argument is populated with the error object. In success conditions, the `result` object is populated with an object describing the result of the render call.

```javascript
var sass = require('node-sass');
sass.render({
  file: '/path/to/myFile.scss',
  data: 'body{background:blue; a{color:black;}}',
  importer: function(url, prev, done) {
    // url is the path in import as is, which LibSass encountered.
    // prev is the previously resolved path.
    // done is an optional callback, either consume it or return value synchronously.
    // this.options contains this options hash, this.callback contains the node-style callback
    someAsyncFunction(url, prev, function(result){
      done({
        file: result.path, // only one of them is required, see section Special Behaviours.
        contents: result.data
      });
    });
    // OR
    var result = someSyncFunction(url, prev);
    return {file: result.path, contents: result.data};
  },
  includePaths: [ 'lib/', 'mod/' ],
  outputStyle: 'compressed'
}, function(error, result) { // node-style callback from v3.0.0 onwards
  if (error) {
    console.log(error.status); // used to be "code" in v2x and below
    console.log(error.column);
    console.log(error.message);
    console.log(error.line);
  }
  else {
    console.log(result.css.toString());

    console.log(result.stats);

    console.log(result.map.toString());
    // or better
    console.log(JSON.stringify(result.map)); // note, JSON.stringify accepts Buffer too
  }
});
// OR
var result = sass.renderSync({
  file: '/path/to/file.scss',
  data: 'body{background:blue; a{color:black;}}',
  outputStyle: 'compressed',
  outFile: '/to/my/output.css',
  sourceMap: true, // or an absolute or relative (to outFile) path
  importer: function(url, prev, done) {
    // url is the path in import as is, which LibSass encountered.
    // prev is the previously resolved path.
    // done is an optional callback, either consume it or return value synchronously.
    // this.options contains this options hash
    someAsyncFunction(url, prev, function(result){
      done({
        file: result.path, // only one of them is required, see section Special Behaviours.
        contents: result.data
      });
    });
    // OR
    var result = someSyncFunction(url, prev);
    return {file: result.path, contents: result.data};
  }
});

console.log(result.css);
console.log(result.map);
console.log(result.stats);
```