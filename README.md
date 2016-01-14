# dts-bundle

[![Build Status](https://travis-ci.org/TypeStrong/dts-bundle.svg)](https://travis-ci.org/TypeStrong/dts-bundle) [![NPM version](https://badge.fury.io/js/dts-bundle.svg)](http://badge.fury.io/js/dts-bundle) [![Dependency Status](https://david-dm.org/TypeStrong/dts-bundle.svg)](https://david-dm.org/TypeStrong/dts-bundle) [![devDependency Status](https://david-dm.org/TypeStrong/dts-bundle/dev-status.svg)](https://david-dm.org/TypeStrong/dts-bundle#info=devDependencies)

> Export TypeScript .d.ts files as an external module definition

This module is a naïve string-based approach at generating bundles from the .d.ts declaration files generated by a TypeScript compiler.

The main use-case is generating definition for npm/bower modules written in TypeScript (commonjs/amd) so the TypeScript code should following the external-module pattern (using `import/export`'s and `--outDir`).

:warning: Experimental; use with care.

This module was born out of necessity and frustration. It is a little hacky but at least it seems to work..

- Original code was extracted from an ad-hoc Grunt-task so for now it is fully synchronous with no error feedback.
- It works by line-by-line string operations so please [report](https://github.com/grunt-ts/dts-bundle/issues) any edge-cases.


## Usage

1) Get it from npm:

````
npm install dts-bundle
````

2) Compile your modules with the TypeScript compiler of your choice with the `--declaration` and `--outDir` flags (and probably `--module commonjs` too).

Something like:

````shell
tsc src/index.ts --declaration --module commonjs --target es5 --outDir build
````

3) Run `dts-bundle`

Let's assume the project is called `cool-project` and the main module is `build/index.js` with a `build/index.d.ts`:

````js
var dts = require('dts-bundle');

dts.bundle({
	name: 'cool-project',
	main: 'build/index.d.ts'
});
````

This will traverse all references and imports for the .d.ts files of your sub-modules and write `build/cool-project.d.ts` with the bundle of all 'local' imports.

**Optional:**

4) Bonus points if you link the generated definition in your package.json's (or bower.json) `typescript` element:

````json
{
	"name": "cool-project",
	"version": "0.1.3",

	"typescript": {
		"definition": "build/cool-project.d.ts"
	}
}
````

Using this makes the definition findable for tooling, for example the [TypeScript Definitions package manager](https://github.com/DefinitelyTyped/tsd) (from v0.6.x) can auto-link these into `tsd.d.ts` bundle file.


### Wrappers

There is also a Grunt plugin [grunt-dts-bundle](https://www.github.com/grunt-ts/grunt-dts-bundle) that goes well with Grunt based compilers, like [grunt-ts](https://www.github.com/grunt-ts/grunt-ts) or [grunt-typescript](https://github.com/k-maru/grunt-typescript).


## Options

Example of all options:

````js
var opts = {

	// Required

	// name of module likein package.json
	// - used to declare module & import/require
	name: 'cool-project',
	// path to entry-point (generated .d.ts file for main module)
	// - either relative or absolute
	main: 'build/index.d.ts',

	// Optional

	// base directory to be used for discovering type declarations (i.e. from this project itself)
	// - default: dirname of main
	baseDir: 'build',
	// path of output file. Is relative from baseDir but you can use absolute paths. If start with "~" then is relative to current path
	// - default: "<baseDir>/<name>.d.ts"
	out: 'dist/cool-project.d.ts',
	// include typings outside of the 'baseDir' (i.e. like node.d.ts)
	// - default: false
	externals: false,
	// reference external modules as <reference path="..." /> tags
	// - default: false
	referenceExternals: false,
	// filter to exclude typings, either a RegExp or a callback. match path relative to opts.baseDir
	// - RegExp: a match excludes the file
	// - function: (file:String, external:Boolean) return true to exclude, false to allow
	// - always use forward-slashes (even on Windows)
	// - default: *pass*
	exclude: /^defs\/$/,
  	// delete all source typings (i.e. "<baseDir>/**/*.d.ts")
	// - default: false
	removeSource: false,
	// newline to use in output file
	newline: os.EOL,
	// indentation to use in output file
	// - default 4 spaces
	indent: '	',
	// prefix for rewriting module names
	// - default '__'
	prefix: '__',
	// separator for rewriting module 'path' names
	// - default: forward slash (like sub-modules)
	separator: '/',
	// enable verbose mode, prints detailed info about all references and includes/excludes
	// - default: false
	verbose: false,
};

// require module
var dts = require('dts-bundle');

// run it
dts.bundle(opts);
````


## Todo

- add feature to create a DefinitelyTyped header (using `definition-header` package)
- add feature to auto-update package.json/bower.json with the definition link
- find time to implement a parser based solution
- run IO asyncronous


# History

- 0.3.x - Support es6 module syntax & rewrite by TypeScript
- 0.2.x - Fixed bugs & added many options (thanks @poelstra)
- 0.1.x - First release


## Contributions

They are very welcome. Beware this module is a quick hack-job so good luck!

* Martin Poelstra (@poelstra): Exclude 'external' typings, optional debug output, improved configurability.


## License

Copyright (c) 2014 [Bart van der Schoor](https://github.com/Bartvds)

Licensed under the MIT license.
