
Javascript, Node, web apps and ecosystem
========================================

__UNDER CONSTRUCTION__

TODO:

- describe CommonJS, UMD, require...
- a few words about Immutable

For faster `npm`: `npm set progress=false`

---

Things you need to know:

  - __Javascript__ and its "current" version, ES6.

    [ES6 (ECMAscript 6)](http://es6-features.org/) is not yet supported in web browsers, but __transpilers__ (source code to source code compilers) like [Babel](https://babeljs.io/) are used to convert ES6 code to ES5 code, so it's OK :) These transpilers can run client-side (good for development) or server side (good for production and for node.js).

    Where to learn Javascript:

    - [JavaScript at MDN - Mozilla Developer Network](https://developer.mozilla.org/cs/docs/Web/JavaScript)
    - [Learn ES2015](https://babeljs.io/docs/learn-es2015/)

    How to learn Javascript:

    - learn how to make and work with classes
    - please forget about jQuery :)

  - __[Node.js](https://nodejs.org/)__ is a Javascript interpreter. It iss used to run scripts in terminal or web app servers.

    Node.js doesn't support ES6, but ES6 can be used with Node using Babel.

    There is also Node fork [io.js](https://iojs.org/) with native ES6 support, but it will be (hopefully) merged back to Node.  

    Debian Linux note: `aptitude install nodejs-legacy`, because most scripts expect Node to be called `node` and not `nodejs`.

  - __[NPM](https://www.npmjs.com/)__ is package repository for Node.

    But not only for Node - you can for example `npm install google-closure-library` so you don't have to download Closure from web manually and serve static files directly from `node_modules/google-closure-library/closure`.

    NPM packages can be installed:
    - into a project's `node_modules`
    - globally, into `/usr/local/lib/node_modules` (on Debian Linux)

    You can find NPM packages for almost anything. Be prepared that NPM packages are usually fragmented - they are small, there is lot of them and they use (depend on) each other :) Each package installed in `node_modules` has its own `node_modules` subdirectory, so there are no version conflicts, but it can take some disk space.

    Tip: `npm install --save something` will not just install `something`, but also save `something` dependency into file `package.json`.

- __[Express.js](http://expressjs.com/)__ is a widely used, minimalistic Node web framework.

  Similar to Python Flask (I think), but in Javascript, so it's asynchronous.

- __Workflow automation, asset management, etc.__

    When creating a web application, you need to solve a few problems:

    - you have a bazillion of Javascript and CSS client code files
        - you write them in ES6, but browsers need to get them transpiled to ES5
        - similarly, you write LESS/SASS/whatever, but it needs to be translated to CSS
        - not mentioning image spriting...
        - you want to be agile (work fast) in __development node__
            - no lengthy (re)compilation
            - automatic reloading of changed things (JS, CSS...) in browser if possible
            - static code analysis and validation ("linting")
            - automated testing
        - you want the exact opposite in __production mode__
            - lengthy compilation is ok
            - everything must be compressed into a few files
            - code should be minimized and possibly obfuscated, unused code ideally removed
            - no dev/debugging features (no unit tests, no live reloading etc.)
