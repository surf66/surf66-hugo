+++
author = "Tom Pennington"
date = "2015-07-16T10:54:24+02:00"
draft = false
title = "Gulp Stubby Server Contribution"
comments = false
+++

This week I made my first contribution to an open source project on GitHub! Recently I've been running acceptance tests against mocks (mock responses as opposed to hitting an actual API). The tests are written in JavaScript and run via Gulp.

To create mocks for the tests, I've been using `npm gulp-stubby-server` which is essentially a wrapper around [Stubby](http://www.stub.by). When running the tests in a CI environment I found that the stubby server wasn't shutting down properly. This meant that the next time the tests ran, the stubby server wouldn't start up as it was already running.

Anyway, my first contribution to open source was adding a `stop()` method into `gulp-stubby-server` so you can programmatically stop the stubby server when your done running tests.

Sounds very simple doesn't it? But this is great! I found an issue at work and rather than just fixing it for me, as it's an open source project, I was able to submit my code and fix it for anyone else having the same issue.

This is cool! :)

**Gulp Stubby Server**

- [Git Repo](https://github.com/felixzapata/gulp-stubby-server)
- [npm gulp-stubby-server](https://www.npmjs.com/package/gulp-stubby-server)
- [My Contribution](https://github.com/felixzapata/gulp-stubby-server/pull/5)