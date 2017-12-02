+++
author = "Tom Pennington"
date = "2017-07-14T10:54:24+02:00"
draft = false
title = "Atomic Design at Zuto"
tags = ["Atomic Design","Front End","CSS"]
comments = false
+++

At Zuto we've recently redesigned our website and application form and are continuing to redesign most of the other customer facing services we have. Previously, we had a very disjointed UX across the site and this was simply because we didn't have any real design standards or guidelines to follow. Starting again has provided us with the opportunity to put these standards and guidelines in place, so it's clear to everyone in the business how we build our user interfaces and enable us to create a consistent UX across all of our services.

10 months and nearly 1,000 commits ago, we (a few front end developers) sat down and started thinking about our new CSS library. We had an existing SASS library and we wanted to first discuss any issues we saw with that and see if there were any elements we wanted to take forward into the new world. Before we wrote a single line of code we decided on a few things.

- Follow the Atomic Design methodology.
- To create and maintain a living and breathing style guide/documentation.
- Cross squad communication was to improve through regular cross squad meetups.

# Atomic Design

> Atomic design is a methodology composed of five distinct stages working together to create interface design systems in a more deliberate and hierarchical manner.
>
> -- <cite>Brad Frost</cite>

We chose to follow Atomic Design simply because it made sense to us as a team. We liked the concept of creating a design system as opposed to web pages because at code level this is how we've built sites for a while now - we just didn't have a name or methodology for it.

An exciting thing for us is that our product and UX teams have fully bought into the idea of Atomic Design. They are really supportive of the idea and work hard to design new experiences using components that already exist in our style guide. The relationship between UX and engineers has greatly improved since we started using Atomic Design and building out our documentation.

# Style Guide
The motivation behind our style guide was to help create a more consistent UX for our customers while at the same time, improving out front end development process.

![Pattern Library](http://i.imgur.com/ytMOeRt.png)
> [styleguide.zuto.com](https://styleguide.zuto.com/)

At Zuto we follow a structure where our customer facing products are developed in several pieces by different squads and are deployed independent of each other. All our engineers are 'T' shaped, they specialise in an area of expertise such as .Net but they also have the ability to apply themselves to the front end of the stack too.

By building our style guide, we were able to provide all of our engineers across the business with a tool to help them build new user interfaces. Any one of our engineers can quickly prototype new web pages, using our style guide as their toolbox. The style guide has enabled us to move faster on the UI and we can prototype 70% of a new design very quickly by simply copying code directly from the documentation.

When building out our style guide we took a lot of inspiration from [Material Design](https://material.io/guidelines/) by Google, [Mail Chimp](http://styleguide.mailchimp.com/) and [Future Learn](https://www.futurelearn.com/pattern-library). There are also hundreds of great examples of style guides at [styleguides.io](http://styleguides.io/examples.html).

# Future Development
Currently, all our CSS is output to one file. Over time this will become larger and larger and some pages won't require half the CSS that's downloaded by the browser. Our next challenge is to split up our compiled code into several smaller CSS files to improve performance.

At the same time, we need to break up our Git repository. At the moment all the SASS is in one repo. This creates a deployment bottleneck when multiple squads want to release changes at the same time. We want to get to a position where squads can manage their own CSS, using our SASS library as a collection of variables, mixins and functions to help them along their way. 

# Last Thoughts
In my opinion, good UX comes from the entire business. It's not about having a talented UX team. I believe there are several factors that can contribute to having a good customer experience.

<ul style="list-style:none;padding:0;columns:2;">
<li>Design</li>
<li>Product</li>
<li>Engineers</li>
<li>Company Culture & Values</li>
<li>Marketing strategy</li>
<li>Our tone of voice</li>
<li>Our contact centre</li>
</ul>

The list goes on and on but I think this all contributes to user experience. Any one of these factors changes and the customer experience will be affected.

I believe with the style guide and new CSS library we've built a solid foundation for our new UX at Zuto. We now have guidelines and documentation available to everyone in the business around how we build user interfaces. This gives us a platform to create a consistent UX across all of our customer facing products.