+++
author = "Tom Pennington"
date = "2014-07-11T10:54:24+02:00"
draft = false
title = "Generating multiple static HTML pages with Gulp and Handlebars"
tags = ["Gulp","Handlebars","front end"]
comments = false
+++

Creating a static HTML pages is easy. Creating multiple copies of that same web page manually and changing content is again easy, however this creates a maintenance overhead. Any web developer will understand how frustrating it can be when you're working on a project and there is that one part that you keep putting off until you absolutely have to because it’s  so laborious and painstaking to complete.

While working on a project recently at work I was faced with this exact problem. At least I could see it coming. We needed tens of copies of the same static HTML page, all with different content. Of course we could have drafted a HTML template and, once we were happy, simply duplicate the template and go through each one and change the content. Like I said before, this creates a horrible maintenance overhead. A job for one of our developers each time there is a change to these pages, that same change would have to be made across every single version of this static page. Not ideal.

In this post I will walk you through the solution I came up with in order to reduce the potential maintenance overhead this could have caused for our dev team.

## Automation to the rescue

If you're a front-end developer, I'm going to assume you know what GulpJS is so I won't waste time explaining it, so based on this assumption - I'll crack on.

To prevent this future maintenance overhead for my dev team, I came up with a solution using GulpJS and Handlebars.

*Hang on, what the hell are Handlebars?*

HandlebarsJS is a small JavaScript library for generating HTML by accepting data in JSON format.

## What's the Solution Already?

In short my solution was to have one *.handlebars* template which contained the structure of my HTML page, links to JS and CSS etc and a JSON config file. This contained the content I wanted to change on each of the generated HTML pages. I generated the HTML files using the *gulp-compile-handlebars* plugin.

## The Code

Here’s an example of how I did it. I'll show you what I did by creating a small static web page about each of the speakers at a front-end conference. First, before I get into it, let me show you my folder structure so things are clear from the off.
```
+-- templates
|   +-- speaker-profile.handlebars
+-- speaker-profile-config.json
+-- package.json
+-- gulpfile.js
```
### Install Dependencies

Install a Gulp plugin for compiling the handlebars template and gulp-rename. I've used gulp-rename so that when the output is saved as HTML, we can rename the file to whatever we like.

```
npm install --save-dev gulp-compile-handlebars
npm install --save-dev gulp-rename
```

### Add Gulp File

Next we need to create a gulp file in the root of our project and include the two gulp plugins we just installed.
```
var gulp = require('gulp'),
    handlebars = require('gulp-compile-handlebars'),
    rename = require('gulp-rename');
```
### Adding the Handlebars Template

We now need a handlebars template containing our HTML content and placeholders for our data, which we will later populate from a JSON file.
```
<!DOCTYPE html>
    <head>
        <title>Front-End Dev Conference</title>
    </head>
    <body>
        <h1>{{fullName}}</h1>
        <p>{{bio}}</p>
        <p>
            You can find me on <a href="http://www.twitter.com/{{twitterUsername}}">Twitter</a>,
            <a href="http://www.github.com/{{githubUsername}}">GitHub</a> or my <a href="http://{{website}}">Website</a>.
        </p>
    </body>
</html>
```

### Add Handlebars Gulp Task

Now back in your gulpfile, we can create an empty gulp task for compiling the handlebars template. Our gulpfile should now look something like this.
```
var gulp = require('gulp'),
    handlebars = require('gulp-compile-handlebars'),
    rename = require('gulp-rename');

gulp.task('handlebars', function() {

});

gulp.task('default', ['handlebars']);

```

### Add the Speaker Config File

We need to compile our `speaker-profile.handlebars` template for each of the speakers in our `speaker-profile-config.json`, but first we need to create this config file.

The config file below contains the details of three of our speakers. Feel free to add more or change the data.

```
[{
    "fullName": "James Burns",
    "bio": "Hey i'm James and i'm a front-end dev currently working in Manchester.",
    "twitterUsername": "@jburns",
    "githubUsername": "jburns",
    "website": "www.jburns.co.uk"
},
{
    "fullName": "Jasmine Hodgson",
    "bio": "Hey! I'm Jasmine and I love SASS!",
    "twitterUsername": "@jazziehodgson_93",
    "githubUsername": "jazziehodgson_93",
    "website": "www.jazzie-hodgson.co.uk"
},
{
    "fullName": "Alex Rimmer",
    "bio": "Hey i'm Al and i'm a Senior Front-End Dev based in London.",
    "twitterUsername": "@arimmer22",
    "githubUsername": "alex_rimmer",
    "website": "www.arimmer.co.uk/portfolio"
}]


```

### Compile Handlebars Template

Now we have our speakers, we need to configure our gulp task to compile our handlebars template and output HTML files to a build directory. First we need to include the speaker config at the top of the gulpfile.
```
var gulp = require('gulp'),
    handlebars = require('gulp-compile-handlebars'),
    rename = require('gulp-rename'),
    speakers = require('./speaker-profile-config.json');
```

Next we add the functionality into the gulp task. Our gulp task should now look something like this.

```
gulp.task('handlebars', function() {
    for(var i=0; i<speakers.length; i++) {
        var speaker = speakers[i],
            fileName = speaker.fullName.replace(/ +/g, '-').toLowerCase();

        gulp.src('templates/*.handlebars')
            .pipe(handlebars(speaker))
            .pipe(rename(fileName + ".html"))
            .pipe(gulp.dest('build/speaker-profiles'));
    }
});
```

Here we are simply looping through the speakers in our JSON config file and compiling our handlebars template with each of the speakers data. Once the template has compiled, we rename the file to something relevant to the speaker (their name hyphenated) and save it as a HTML file in a build directory.

That should be it! Run your gulp task in terminal/command prompt.
```
gulp handlebars
```

If we take a look at our project structure, we can see a build directory has appeared in the root of our project with the following structure.

```
+-- build
 +  |-- speaker-profiles
        +--alex-rimmer.html
        +--james-burns.html
        +--jasmine-hodgson.html 
```

Our three generated HTML files should now look something like this, all with different content of course.

```
<!DOCTYPE html>
    <head>
        <title>Front-End Dev Conference</title>
    </head>
    <body>
        <h1>Alex Rimmer</h1>
        <p>Hey i&#x27;m Al and i&#x27;m a Senior Front-End Dev based in London.</p>
        <p>
            You can find me on <a href="http://www.twitter.com/@arimmer22">Twitter</a>,
            <a href="http://www.github.com/alex_rimmer">GitHub</a> or my <a href="http://www.arimmer.co.uk/portfolio">Website</a>.
        </p>
    </body>
</html>
```

## I know what you're thinking

*"Why don't you just use a database and do everything server side?"*

For our dev team, this was the very beginning of a new project and we had a strict deadline. This worked best for us - it was a quick solution that I could knock up in a couple of hours and it meant we could go live with something. Building software is all about small iterations and only implementing things when you need them.

> YAGNI - You aren't gonna need it

All the business want to see is regular feedback on what we are doing. We can't go away for three months and develop a database, API and server side templating engine and come back and it's wrong. As a starting point we have a solution that works for our team, makes the rest of the business happy and we can now move forward and build on top of it.

## Conclusion

When working in a dev team, you may find yourself from time to time having to complete laborious tasks which seem to take you forever, and in the long run cause a headache for the rest of the team. As a developer, your time is far better spent architecting automated solutions to make your teams life easier, and save time and effort in the long run.

Yes, it will take longer in the short term to come up with a solution that works for your team. However that outweighs the time your team you will lose in the future if a clever solution is not implemented from the beginning.

Start small and work in iterations. What your building today may be completely different from what it will become over the coming months, but thats ok! During that time you will have released countless times and learnt so much about what it is you actually need to build for the people that are using it.

