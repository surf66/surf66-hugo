+++
author = "Tom Pennington"
date = "2018-11-12T20:21:24+02:00"
draft = false
title = "Implementing a design system across a microservice architecture"
tags = ["Atomic Design","Front End","AWS"]
+++

I'm sure you've read lots of posts online about how to build design systems and discussions about front-end methodologies, I've even written one myself. However, my role as a software engineer is not only to build a well-structured design system and pattern library. It is also to ensure it's implemented in the right way across all of our systems and products. I've built a few different design systems now, all on different scales and in this post I will share how I like to implement design systems across several different products in a microservice architecture.

## Design System Overview
Our current design system at Ditto Music has been built while following the Atomic Design methodology. There are lots of different front-end methodologies, however Atomic Design works for myself, my team and for me that's what matters. What I feel is even more important is that I'm very lucky that my lead UX designer and software engineering manager are both fully on board with this methodology. This makes a massive difference. On previous projects I've had what feels like endless battles with other colleagues trying to suggest that this is the right way to build for the web over the classic page by page, design then build waterfall method.

In terms of the technical details, I've built a custom SASS framework which contains a flexbox based grid system and some Atoms, Molecules and Organisms. Alongside the SASS framework, in the same repository is the pattern library website. This serves as Ditto's brand bible. Anything we build into the SASS framework, we document at the same time in the pattern library. Just a quick one on the decision to manually document everything. I feel automated documentation is bad and developers don't properly buy into you're design system if everything's done for them. A living and breathing design system is created when everyone buys into it and actively contributes to it.

The pattern library is still in infancy and it's built using a static website generator. Nothing too fancy and it's not overcomplicated because it doesn't need to be - it serves the purpose. It documents how the design system works, how integrate it into your project and how to use our reusable components with code examples. It also has a handy search feature.

![Ditto Pattern Library](https://i.imgur.com/LMa55EY.png)

## Architecture
As mentioned previously, at Ditto Music we have a microservice architecture and in order to use our patterns across all our different services, we use AWS CloudFront. I've used Terraform to create the infrastructure below. Our DNS is configured via Route53 and we have an instance of CloudFront which points to an S3 bucket where all our content is deployed.

![CDN Architecture](https://i.imgur.com/u8NnC4y.png)

When the pattern library builds in our CI environment, the output is of course the pattern library website but also `dittomusic-framework.css`. This is a minified CSS file that is deployed to our CDN inside AWS.

More info on `dittomusic-framework.css` and what it contains.

- 8kb minified (not gzipped *yet*)
- Fully responsive flexbox grid
- Global styles such as typography, colour palette and icons
- Reusable Atoms such as buttons and form elements
- Larger, reusable Molecules and Organisms such as dialog windows, header and footer
- Heper/utility classes

## Usage
Now that we have a global CSS file on our CDN, any developer at Ditto can reference it and start to quickly build out prototypes of new pages and using the documentation, they can develop a large percentage of their new page.

### What about project specific styling?

Our main `dittomusic-framework.css` file is generated from SASS. Part of our core SASS setup is a custom module called SASS Core. SASS Core is a reusable module containing all the integral parts of our SASS framework such as variables, mixins and functions. We extract these key elements out into a module so our brand can be reused across many different services in our microservice architecture. Any service at Ditto containing UI elements can do so by installing SASS Core as an `npm` dependency.

The diagram below shows how a project could use SASS Core to generate it's own CSS. We don't want the pattern library to become a monolithic SASS repo where we throw everything. Instead, the idea is for the pattern library to only contain reusable components and patterns. Different projects or applications will need to generate their own CSS as some project specific components may exist, or you may wish to extend a component that currently exists in the pattern library, changing its appearance.

![SASS Core](https://i.imgur.com/hgXbYQd.png)

Just to make it absolutly clear in the above diagram, once SASS Core is added via npm, it is imported inside `main.scss`. 

```
@import "sass-core";

@import "components/component-a";
@import "components/component-b";
```

Now it is possible to use any of our variables or mixins etc. Any project that generates it's own CSS also deploys that CSS to the CDN. All our CSS, JS, images and fonts at Ditto are deployed to our CDN.

### CloudFront Cache Invalidation
This is really not an issue and you'd think it would be. Via the AWS CLI they make it very easy and quick to invalidate certain files. Built into our CI pipelines I have a script which will create invalidations for the relevant files that are updated on the CDN. Happy Days.

> [https://docs.aws.amazon.com/cli/latest/reference/cloudfront/create-invalidation.html](https://docs.aws.amazon.com/cli/latest/reference/cloudfront/create-invalidation.html)

## Future Development

I think our setup is great and it's working well for now. However, I'd like to look at a solution whereby the entire pattern library is a module like SASS Core. You could then add this as a project dependency and cherry-pick only the parts of the framework you need. This way you are certain that there is no CSS on the client that is not required.

## TLDR;
At Ditto Music, we use Atomic Design as a front-end methodology and employ a microservice architecture in AWS. The output of our pattern library is a CSS file containing core reusable elements such as a grid system, colours, buttons, headers, footers etc and this is deployed to an S3 bucket which sits behind an instance of CloudFront.

Any projects within the business which contain UI simply reference this CSS file in their HTML. They can then use our SASS Core node module to use predefined SASS functions, variables and mixins to build project specific CSS. SASS Core exposes things like colour variables and grid breakpoints etc.
