+++
author = "Tom Pennington"
date = "2015-07-07T10:54:24+02:00"
draft = false
title = "Front-End Testing with JavaScript & WebdriverIO"
tags = ["WebdriverIO","JavaScript","GulpJS","BrowserStack"]
comments = false
+++

The one thing front-end developers aren't bred to do is write tests around their code. Why not? When working with 'back-end' developers, everything they write has tests around it, why aren't we doing that? Why is this not part of our workflow like it is theirs? We're the ones that have to make sure our sites work on this huge spectrum of devices and screen sizes.

Recently I've been writing automated tests for the front-end and I wanted to share what I've been doing and how I've gone about it.

## My Setup
Before I jump into the code and it all gets a bit messy, here's a brief description of how I've gone about it.

- [gulp-jasmine](https://www.npmjs.com/package/gulp-jasmine) for running the tests
- [Jasmine](http://jasmine.github.io/) for the test framework
- [selenium-standalone](https://www.npmjs.com/package/selenium-standalone) for the selenium server which the tests run against
- [run-sequence](https://www.npmjs.com/package/run-sequence) to run gulp tasks synchronously
- [WebdriverIO](http://webdriver.io/) to interface with the browser

I've put my setup on GitHub if you want to clone it and see how it works. https://github.com/surf66/frontend-tests

##### Folder Structure
![Folder Structure](http://i.imgur.com/roggPhA.png)

## Gulp Setup

In my setup, the tests are run via one gulp task which calls a sequence of three other tasks using `npm run-sequence`. These three tasks pretty much do what they say on the tin:

- selenium-start
- run-tests
- selenium-stop

#### Gulp Task 1: selenium-start
This gulp task is in charge of starting selenium server. It's pretty straight forward and `npm selenium-standlone` is well documented.
```
gulp.task('selenium-start', function (done) {
    selenium.install({
        version: '2.45.0',
        baseURL: 'http://selenium-release.storage.googleapis.com',
        drivers: {
            chrome: {
                version: '2.16',
                arch: process.arch,
                baseURL: 'http://chromedriver.storage.googleapis.com'
            }
        },
        logger: function (message) { }
    }, function (err) {
        if (err) return done(err);

        selenium.start(function (err, child) {
            if (err) return done(err);
            selenium.child = child;
            done();
        });
    });
});
```

#### Gulp Task 2: run-tests
This task is responsible for running the tests against the selenium server.
```
gulp.task('run-tests', function() {

    return gulp.src('test/*-spec.js')
        .pipe(jasmine(
            {
                verbose : true,
                includeStackTrace : true,
                reporter: [
                    new reporters.TerminalReporter({
                        verbosity: 3,
                        color: true,
                        showStack: true
                    }),
                    new reporters.JUnitXmlReporter({savePath: 'test-results'})
                ]
            }
        )).on("error", function(result) {
                selenium.child.kill();
                throw new Error("There are failed tests!");
                this.emit('exit');
                process.exit(-1);
                this.emit('end');
        });
});

```

#### Run Sequence Issue
I had some trouble with `npm run-sequence` when writing this task. Run Sequence is not very clever when it comes to a step in it's 'run sequence' failing. I had an issue where essentially if the any of the tests failed (which they will do), run sequence fell over and the node process stayed open.

This meant that when I deployed into our CI environment, the test stage never failed and just hung! This also lead to issues with selenium server not closing down properly and it all got a bit meh! I had to add the on error function in order to kill selenium off properly, throw and error and exit the node process if tests failed, which in turn caused the build to pass or fail properly.

anyway...

#### Gulp Task 3: selenium-stop
This task is simple, all we have to do is kill selenium server.
```
gulp.task('selenium-stop', function (done) {
    selenium.child.kill();
});
```

### Running the tests
You may know that GulpJS runs tasks asynchronously. This was rubbish as I needed the selenium server to be running while the tests were running. I used `npm run-sequence` to ensure these three gulp tasks ran synchronously.

```
gulp.task('test', function() {
    runSequence('selenium-start', 'run-tests', 'selenium-stop');
});
```
Now all I needed to do locally and in our CI environment was run `gulp test`

## Writing the tests
Writing the tests is very straight forward. Both **Jasmine** and **WebdriverIO** are very well documented and I was able to get some simple tests up and running quickly. The hardest part is getting setup. Once you've wrote a couple of tests your away!

Here is an example of one of my tests.

#### Test File: zuto-homepage-spec.js
```
var assert                 = require("assert"),
    testConfig             = require("./config/test-config.js"),
    zutoPageObject         = require("./page-objects/zuto.json"),
    zutoBaseUrl            = testConfig.getBaseUrl();

describe("Zuto", function() {
    jasmine.DEFAULT_TIMEOUT_INTERVAL = 9999999;

    beforeEach(function() {
        client = testConfig.getWebdriver();
        client.init();
    });
    
    it("user can change their credit score on the homepage calculator", function(done) {
        client
            .url(zutoBaseUrl)
            .getValue(zutoPageObject.financeCalculatorCreditScore).then(function(value) {
                expect(value).toEqual("good");
            })
            .selectByValue(zutoPageObject.financeCalculatorCreditScore, 'excellent')
            .getValue(zutoPageObject.financeCalculatorCreditScore).then(function(value) {
                expect(value).toEqual("excellent");
            })
            .call(done);
    });

    afterEach(function(done) {
        client.end(done);
    });
});

```

This test carries out the following steps:

- opens a browser
- navigates to a URL
- checks the current value of a select box is `good`
- selects another option from the drop down
- asserts that the value has changed and is now `excellent`

You'll see that I'm referring to `testConfig` a lot. I've basically put all the heavy stuff into a series of utility methods just to tidy up my code. The only one you need to look out for is `getWebdriver()`

```
//set node environment
process.env.NODE_ENV = 'prod';
process.env.BROWSERSTACK_USERNAME = "yourbrowserstackusername";
process.env.BROWSERSTACK_ACCESS_KEY = "youraccesskey";

var webdriverio = require("webdriverio"),
    config      = require("./env-config.json");

module.exports = {

    /**
     * Returns webdriver configuration
     * @param {Boolean} browserstack
     */
    getWebdriver: function(browserstack) {
        var client = {};

        if(browserstack === true) {
            //browser stack configuration
            client = webdriverio.remote({
                desiredCapabilities: {
                    browserName: 'chrome',
                    version: '27',
                    platform: 'XP'
                },
                host: 'hub.browserstack.com',
                port: 80,
                user : process.env.BROWSERSTACK_USERNAME,
                key: process.env.BROWSERSTACK_ACCESS_KEY,
                logLevel: 'silent',
            });
        } else {
            client = webdriverio.remote({desiredCapabilities: {browserName: 'chrome'}});
        }

        return client;
    },
    /**
     * Returns base url from config.json based on node env
     */
    getBaseUrl: function() {
        var zutoBaseUrl = config[process.env.NODE_ENV];
        return zutoBaseUrl;
    },
    waitForPageLoad: function() {
        return 3000;
    },
    waitForMenuAnimate: function() {
        return 100;
    }
};
```

## Page Objects
I'm going to talk a little bit about page objects as I think using them is a very good habit to get into, especially for further down when your test coverage grows.

You'll see in my test `zuto-homepage-spec.js` in the `getValue()` method call I'm passing in `zutoPageObject.financeCalculatorCreditScore`. Many of the functions in WebdriverIO take CSS selectors which makes it very easy to select or click elements etc.

The idea of a page object is to keep all of these selectors in one place. As we write more and more tests we don't want CSS selectors riddled throughout the tests so we put them in a page object. At the moment, my page object is nice and simple, just a JSON file.

```
{
    "financeCalculator": ".js-finance-calculator",
    "financeCalculatorApplyButton": ".js-calc-apply-now",
    "financeCalculatorCreditScore": ".js-hero-banner select",
    "financeCalculatorLoanAmountDecrease": ".js-hero-banner .js-loan-amount-decrease",
    "financeCalculatorLoanAmountIncrease": ".js-hero-banner .js-loan-amount-increase"
}
```

Now if any of the devs change a class and the tests fail, they don't have to trawl through all the tests and replace the class name - they simply amend the page object.

## Pluging into BrowserStack
If you would rather not go to the trouble of setting up a VM to run your tests on before you deploy why not plug your tests into BrowserStack? With a FREE account you get 160 minutes a month to run your tests (I think!).

If you refer back to my `getWebdriver()` method inside `test-config.js` you will see it has a parameter `browserstack`. I've set this method up so that I can pass in `true` and the tests will run against BrowserStack. With BrowserStack your also not limited to Chrome. There are several different browsers, browser versions, operating systems and even mobile devices to run your tests against. Below is a screenshot of my tests running in BrowserStack.

![BrowserStack Automated Tests](http://i.imgur.com/nyjE0iS.jpg)

## Conclusion
I guess there isn't really a conclusion to this post - I'm just sharing some of my work. How many times have you deployed to live, finished testing and all is well until someone realises you've broken something that you didn't even realise you might have touched? This is where automated testing will save your ass in future. It goes without saying, as you write more and more tests and you increase coverage, you can be confident in what your pushing to live.

Plus, I don't know about you but I think this shit's fun too!