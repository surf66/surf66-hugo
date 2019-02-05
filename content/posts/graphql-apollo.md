+++
author = "Tom Pennington"
date = "2019-02-01T12:02:00+02:00"
draft = false
title = "An insight into our use of GraphQL with React and Apollo at Ditto Music"
tags = ["Apollo", "React", "GraphQL"]
+++

GraphQL is a hot topic at the moment in the front end development community. This short post will hopefully give you insight into why we have used GraphQL at Ditto Music and also how I have queried our GraphQL API in the React layer. First let me start with a bit of background.

Currently, once Ditto customers or artists login to their account area, they can view stats about how they are performing. They can see things like how many streams they've had split by store or country and how much money they have earned in the last 6 months. This page is very slow and can even crash if an artist with lots of sales accesses it. We had three main goals in mind while improving/rebuilding this area of the website.

- Improve the page speed/performance
- Split the sales area out of the existing PHP monolith system into microservices
- Surface the data with a better UX (using graphs instead of tabular data)

The big one here is splitting out a rather large part of Ditto's system into microservices. As interesting as this was, I won't go into too much detail here - that's for another time.

![Graphs](https://i.imgur.com/FIg7X4O.png)

> sales data in the image above is all fake

We made the decision quite early on to migrate all the sales data over from a MySQL database and into ElasticSearch. That way we could rapidly query massive amounts of data more efficiently. We could then sit our GraphQL API inbetween ElasticSearch and our .NET Core Web Application, which served as our UI. The diagram below shows the new microservice architecture.

![Architecture](https://i.imgur.com/oyK3BG2.png)

The main reason we use GraphQL is probably the same reason you do or why you want to. It provides a nice way of gathering data from several different data sources without writing your own complicated API methods that become very specific. It also massively speeds up development once setup too. Our UI can query the same endpoint over and over with different queries and get back data in all different shapes. It meant we could build quite a lot of our graph components by simply adjusting the queries we sent up to the API.

## Example Code
During this project I got to learn some pretty cool stuff in the JavaScript layer. I wanted to create a simplified version of what I learned in GitHub to share and also to look back on.

In the React layer I have used [Apollo Client](https://github.com/apollographql/apollo-client) and [React Apollo](https://github.com/apollographql/react-apollo) to allow me to query our GraphQL API. To tell the truth it would have taken so long to break all this down in a blog post so if you are interested and want to learn more about Apollo in React then feel free to check out the code. The example repository includes the following.

- A NodeJS server running [Apollo](https://www.apollographql.com/)
- Basic React client
- A React component which uses [React Apollo](https://www.npmjs.com/package/react-apollo) to fetch data from the GraphQL API
- Jest tests for the React component

> https://github.com/surf66/apollo

## TLDR;

> We use GraphQL at Ditto Music to query massive amounts of data in ElasticSearch and existing legacy data stores.
  GraphQL provides a nice way to get all the different shapes of data we need for the pretty graphs in our UI.
  I made a repo demonstrating how the React layer works which includes how to test it using Jest and a GraphQL API using Apollo https://github.com/surf66/apollo