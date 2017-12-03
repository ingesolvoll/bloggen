---
layout: post
title:  "Leiningen: A new contender in the JS build tool battle?"
date:   2017-08-25 12:00:00 +0100
---

We have come a very long way in web development. With a bit of effort, you can set up a professional
and higly productive build environment for your JS project. And new tools and ideas keep popping up.
Some people are tired of the constantly moving targets of JS best practices, arguing that it's impossible to keep up.
Other people might say that those who complain are spoiled brats.

### New kid in town
Today I would like to reach out to those who feel that setting up a JS build is simply too much work.
After the amazing work done by the clojurescript team this summer, the clojurescript compiler is ready
to compete with Gulp and Webpack.

### Batteries included
This is the **Clojurescript** compiler. Why would you want to use that for a pure JS project,
when there are tons of pure JS/Node alternatives around?

*The main reason is simplicity.*

Webpack is excellent and powerful, but it's not trivial to set up. A good working build can take
days and weeks to set up.

- Live reload during development
- NPM dependencies handled for you
- Minification via Google Closure compiler