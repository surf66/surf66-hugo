+++
author = "Tom Pennington"
date = "2017-03-09T10:54:24+02:00"
draft = false
title = "React at Zuto"
tags = ["React","JavaScript"]
comments = false
+++

At Zuto we've been using React for around six months now. The repository which contains all our React components was created back in October 2016. At the time of writing it has seen 700 commits by 5 main contributors. This post aims to summarise some technical decisions we have made along the way.

When you start to learn React, you'll quickly find like with most JavaScript frameworks or libraries, there are ten million different ways of writing it. Or at least that's how I felt. On top of this there are a bunch of tools available to use alongside React and it's quite daunting. I found myself constantly creating mini react apps in all kinds of different ways, using a different set of tools each time.

A few of us at Zuto knew React was the right tech for an upcoming project and we were confident we could justify that decision. What we didn't know was what the ideal way to build and test React components was. We're very disciplined at Zuto, everything we write is unit and acceptance tested and we keep our code to high standards so it was important to get this right.

We're a small squad and we had to move quickly as we needed to start this new project. We settled on the following structure for building our React components.

- NPM (for managing our dependencies, pretty standard)
- NPM Scripts (our build tool. No more Gulp.)
- Webpack (our script bundler)
- Babel (for transpilling es6 to es5)
- Jasmine (for writing unit tests)
- Karma (for running unit tests)

Whether this setup is right or wrong is irrelevant. This is working well for us at the moment and to say that at the beginning none of us had any real experience with React, we are pretty happy with it.

## Possible improvements
I think there are some aspects of this we could definitely improve in the future. I'd like to look at [Jest](https://facebook.github.io/jest/) for running our tests. We went with the Jasmine/Karma setup as it was what we were used to from other projects and wanted to get up and running quickly.

Maybe we could move away from Webpack? That's up for debate too!

## Next steps
At first we started writing React components very quickly and have since refactored some areas several times. The scope and requirements of our project have also grown over time.

We began to realise we were no longer writing small, individual React components. We were creating a JavaScript application. There is now lots of complex logic in our application. Some components are starting to handle data and perform controller logic. I've seen React described as the 'V' in MVC. We now have an application that is becoming more than just the 'V'.

Currently we are looking at introducing [Redux](http://redux.js.org/docs/introduction/CoreConcepts.html), a state management tool for JavaScript applications to cope with this.

### Conclusion
We've learnt a hell of a lot in the last 6 months and we've come a long way from where we were. We have moved very fast while constantly questioning the decisions we are making. As a squad, we are all still new to React as a technology and have a lot to learn. We believe in our approach. Stay committed to producing high quality, testable code and keep learning and iterating.

### Airbnb
Quick shout just to check out [Airbnb's github](https://github.com/airbnb) page. We have taken some inspiration from them when writing React. They also have a really good React/JavaScript [code style guide](https://github.com/airbnb/javascript/tree/master/react).

### Create React App

Also a quick mention that Facebook have a tool called Create React App.

> https://github.com/facebookincubator/create-react-app

In the past I've always avoided using scaffolding generators. I feel like they can add bloat to your dev environment and sometimes you may not fully understand what's going on underneath it all. Although, I have to say this one is pretty nice.

There are several reasons why I would recommend this tool. Firstly, it gets you started quickly and makes certain decisions for you so you can focus on writing JavaScript. Here are some of those decisions.

- It sets up a build process using NPM scripts, which is inline with what we like to use
- It gives you a local dev server
- It sets up a unit test runner for you
- It does very little. Sounds strange but it just sets up a basic React application with one component and a unit test for it. It's bare bones and I like it.

> Apologies if you thought this post was going to go into further technical detail :)