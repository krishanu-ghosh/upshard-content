---
title: Node.js Basics
description: Baseline Node.js literacy — modules, npm, the file system, and a basic HTTP server — for reading an AI-generated backend.
roadmap: ai-product-builder
node: node-js-basics
group: core-skills-you-still-need
video:
  provider: youtube
  id: "32M1al-Y6Ag"
  start: 0
  end: null
creator:
  name: Traversy Media
  channel_url: https://www.youtube.com/@TraversyMedia
duration_seconds: 7595
level: beginner
prerequisites:
  - html-css-and-javascript-fundamentals
takeaways:
  - Node.js runs JavaScript outside a browser, which is what lets JavaScript act as a server language instead of only a page-scripting one.
  - A module is a single file of code that exports specific values for other files to import, and a real backend is built from many small modules wired together.
  - package.json declares a project's dependencies and scripts; npm installs the former and runs the latter, and node_modules is where the installed code actually lives.
  - Node's file system module lets code read, write, and watch files on disk, which is how a server persists or serves data without a database.
  - A basic HTTP server is just a function that receives a request object and decides what response to send back — everything a framework like Express adds is convenience on top of that one idea.
check_questions:
  - q: What does Node.js add that lets JavaScript run a backend at all?
    a: Node.js provides a JavaScript runtime outside the browser, along with APIs for things like the file system and networking that a browser deliberately doesn't expose, which together let JavaScript act as a server language.
  - q: What's the difference between package.json and node_modules?
    a: package.json is the project's manifest — it declares which dependencies the project needs and defines npm scripts; node_modules is the folder where npm actually downloads and stores the code for those dependencies after install.
  - q: At its core, what does a basic Node HTTP server do?
    a: It registers a callback function that runs every time a request comes in, receiving a request object describing what was asked for and a response object the code uses to decide what status, headers, and body to send back.
deeper:
  - label: Node JS Tutorial | Crash Course
    url: https://www.youtube.com/watch?v=vJEO57B05Sg
related_formulas: []
---

## JavaScript, now off the page

Every JavaScript you'd written so far probably ran inside a browser tab, reacting to clicks and updating a page. Node.js is a separate runtime that executes that same language outside the browser entirely — on a server, on your own laptop, in a deployment container — with no page or DOM in sight. That single shift is what makes a JavaScript backend possible at all: Node adds APIs for talking to the file system, opening network connections, and listening for incoming requests, none of which a browser would ever let a web page do for security reasons. When an AI tool scaffolds a "Node backend" for you, this is the environment that code is written to run in.

## Modules: how a backend is actually organized

A real Node project is never one giant file. It's broken into modules — individual files that each do one job and explicitly `export` the pieces other files are allowed to use, which those other files then `import` or `require`. A route handler module might import a database-helper module, which itself imports a configuration module. When you're tracing through generated backend code, following these import lines is literally following the map of how the pieces fit together; a function behaving strangely is often actually defined three files away from where you're looking; reading the imports at the top of a file tells you exactly where to go next.

## npm and package.json: the project's inventory

Almost no Node project is built from scratch — it pulls in other people's published packages for things like parsing data, talking to a database, or handling authentication. `package.json` is the file that records which packages a project depends on, at which versions, plus a set of named `scripts` for common tasks like starting the server or running tests. npm is the tool that reads that file: `npm install` downloads everything listed into a folder called `node_modules`, and `npm run <script-name>` executes one of those named scripts. If generated code imports something and it isn't working, the first two things worth checking are whether that package is actually listed in `package.json` and whether `npm install` has actually been run.

## Touching the file system

Browsers deliberately can't read or write arbitrary files on a visitor's disk, but a server legitimately needs to — reading a configuration file, writing a log, serving a stored image. Node's built-in `fs` module provides exactly this: functions to read a file's contents, write new content to one, check whether a path exists, or watch a directory for changes. You'll see this show up constantly in generated backend code wherever persistence doesn't go through a full database, and recognizing an `fs.readFile` or `fs.writeFile` call tells you immediately that the code is reading or changing something on disk, not in memory.

## The HTTP server underneath every framework

Strip away every convenience a framework like Express adds, and a Node server is just a function registered to run every time a network request arrives. That function receives a request object — method, URL, headers, body — and a response object it uses to decide what to send back: a status code, some headers, a body. Frameworks exist to make routing and that response-building less repetitive, but the underlying model never changes. Once you can picture "a request comes in, some code decides what to send back," reading any Express route handler an AI tool generates becomes a matter of finding that same shape dressed up in slightly friendlier syntax.
