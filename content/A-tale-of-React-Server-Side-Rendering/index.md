---
title: "A tale of React Server Side Rendering"
description: "I want to start by mentioning that as of starting this I didn’t have a lot of experience with SSR besides adding styled components in a NextJS app. So I have this website I created called…"
date: "2018-06-04T17:18:18.242Z"
categories: 
  - React
  - Server Side Rendering
  - Apollo
  - JavaScript
  - Open Source

published: true
canonicalLink: https://medium.com/yld-engineering-blog/a-tale-of-react-server-side-rendering-cb95a441ca01
---

![](./asset-1.jpg)

I want to start by mentioning that as of starting this I didn’t have a lot of experience with SSR besides adding styled components in a NextJS app.

### What’s the story here?

So I have this website I created called [https://awesometalks.party](https://awesometalks.party) that was being rendered on the client. I was having issues with SEO and I also wanted to be cool of course and render it on the server for performance and coolness bonus.

I did this using [Razzle](https://github.com/jaredpalmer/razzle/) that is this really awesome tool and this meant that I had a **server.js** that ran on the server, an **app.js** that was isomorphic and also a **client.js** for all your client needs.

Here is an example of the most basic server you can use with Razzle: [https://github.com/jaredpalmer/razzle/blob/next/examples/basic/src/server.js](https://github.com/jaredpalmer/razzle/blob/next/examples/basic/src/server.js)

Before you call me a noob, that I am, I want to say that this project was not just a [React](https://www.yld.io/speciality/react-js/) project but it also included several things that needed to be SSR’d as well, like:

-   [React Router](https://github.com/ReactTraining/react-router)
-   [Font Awesome](https://github.com/FortAwesome/react-fontawesome)
-   [React Helmet](https://github.com/nfl/react-helmet)
-   [Styled Components](https://github.com/styled-components/styled-components)
-   [React Apollo](https://github.com/apollographql/react-apollo)

So we are going to start with the ones that went super smoothly:

### React Router (AKA: At least Michael Jackson loves me)

React Router is the best, we all know that but what I didn’t know is that server side rendering it didn’t involve a single tear and it was by far the fastest one to do so. All you need is to spread the magic over three files.

So in your app that is isomorphic you place all the routes but without the actual react-router provider like so:

Embed placeholder 0.39210344818745524

In the actual client.js you place the provider since this will only run on the client but the routes are for everything:

Embed placeholder 0.2819411735439068

Why isomorphic-fetch? You never know in these things!

What about the server? Well you just have to wrap your component to be SSR’d in a Static Router and that is about the only change you need to do:

Embed placeholder 0.3817774821066193

That’s it! Those are the only differences we have to make in the server.js to get React Router working on the server as well 😀

### Font Awesome (Yes, it needs SSR otherwise weird stuff happens)

I actually didn’t do this one, it was a PR but I have no idea where there are docs for this, but ok.

Font Awesome was implemented in a way where only the icons we needed were imported and added to the library so that everyone doesn’t get a 3mb icon font.

There is a icons.js with:

Embed placeholder 0.29712467869200276

Notice the config where we tell font awesome that we will add the CSS and instead of it being added by the fontawesome package automatically. After that, this file needs to be imported into our app.js, but on this side, that is it. You just need to know that this config exists.

Then on the server some markup had to be modified for us to add the actual CSS:

Embed placeholder 0.07614213316857055

By adding **_<style>${fontawesome.dom.css()}</style>_** we now have perfect icons on both the client and the server and we can carry on.

Here is a [link to actual PR](https://github.com/SaraVieira/awesome-talks/pull/48).

### React Helmet (GOOD DOCS ARE IMPORTANT)

Their docs are actually very good so you can see them [here](https://github.com/nfl/react-helmet#server-usage) but what you need to change on the server is:

Embed placeholder 0.47590660355619163

There are no changes needed on the client and your SEO will be on fleek from now!

### Styled Components (css-in-js team)

These docs are also pretty amazing so let’s start by the changes in the server. First thing we need to do is wrap our root component in their theme provider because I am also using themes with styled components, so our root components will look like:

Embed placeholder 0.7087354005845474

After this provider component has been added to our root, we can follow the [docs](https://www.styled-components.com/docs/advanced#server-side-rendering) that tell us to create a new server stylesheet, collect the styles of the Root and get the style tags so this means we need to add this code:

Embed placeholder 0.3704207798794219

And now add these tags into the response we send to the client:

Embed placeholder 0.025744601674155376

Now a word of warning: I also had some globally injected styles for the body and general clean up of the page and these were being ignored by the SSR.

This was happening because I was importing them in the client.js instead of the app.js and this meant it only ran in the client.

If you have any type of global styles make sure to import them in the isomorphic part of the app like so:

Embed placeholder 0.3204474763731606

And you are all set for styled components and now Apollo !

### Apollo

I love Apollo, I do, but this was a pain also because I started with Apollo Boost just to make the config easier but then hit a wall and had to download all the packages.

Let’s start by getting the place where we create the Apollo Client away from the main initial files and put it into a file that in my case looked something like:

Embed placeholder 0.8850674598317751

And now you are thinking: What is this process.browser ? It’s just a way of figuring out if we are on the client or the server and render accordingly.

So we will initiate the ApolloClient with no devTools and in SSR Mode and then those will be switched.

And now comes this cache thing. So the way Apollo handles SSR is a little like styled components does where it gets all the things you are fetching in the page and dumps it in the dom so what this restore function does is look for the variable **_window.\_\_APOLLO\_STATE\_\__** and restore it as the cache for the client as well. But we haven’t gotten this to work on the server so for now nothing will happen.

On the server there is a bunch of stuff we need to do, first thing is wrap our Root in the Apollo Provider:

Embed placeholder 0.20765242575917853

After this we need to use **_getDataFromTree_** and what this function does is get all the queries you have in your page and fetches them all and returns a promise, after this promise is resolved we can actually get the initialState and place it in the response like so:

Embed placeholder 0.11442981344051395

And this is where Apollo gets the initial state we saw in the in the above code and like this Apollo will render everything like magic.

This is what I learned in the lengthy discovery to get the App running on SSR using [Razzle](https://github.com/jaredpalmer/razzle/) and I hope this is useful to someone going through the same struggles into SSR and React Apps.

For anything more here is the Github repo with all the code: [https://github.com/SaraVieira/awesome-talks/tree/master](https://github.com/SaraVieira/awesome-talks/tree/master)

Also feel free to reach out to me for any questions.

Written by [Sara Vieira](https://twitter.com/NikkitaFTW) — Developer Advocate at [YLD](https://www.yld.io).

---

#### Interested in React? Read more about it:

[**Deploy your Create React App with Docker and Nginx**  
_First thank you to Simona Cotin and Super Diana for answering the noob docker and nginx questions and reminding me that…_medium.com](https://medium.com/yld-engineering-blog/deploy-your-create-react-app-with-docker-and-ngnix-653e94ffb537 "https://medium.com/yld-engineering-blog/deploy-your-create-react-app-with-docker-and-ngnix-653e94ffb537")[](https://medium.com/yld-engineering-blog/deploy-your-create-react-app-with-docker-and-ngnix-653e94ffb537)

[**Rolling your own Redux with React Hooks and Context**  
_For managing shared state in complex JavaScript applications, Redux is undisputedly the most popular choice. At the…_medium.com](https://medium.com/yld-engineering-blog/rolling-your-own-redux-with-react-hooks-and-context-bbeea18b1253 "https://medium.com/yld-engineering-blog/rolling-your-own-redux-with-react-hooks-and-context-bbeea18b1253")[](https://medium.com/yld-engineering-blog/rolling-your-own-redux-with-react-hooks-and-context-bbeea18b1253)