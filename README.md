# Isomorphic Angular

This repo is dedicated to the community efforts to make Angular2 isomorphic. Angular2 will not
have server rendering built in, but it does have several key features that should make this
possible with some work including:

1. DOM Facade - Angular2 is not tightly coupled to the DOM
1. TypeScript - Most Angular specific code is in annotations which means the remaining code is largely vanilla JavaScript
1. View Hydration - Angular2 allows you to dehydrate/hydrate views

The Angular core team believes that server rendering is possible with Angular2, but it is lower on
their priority list. So, the primary goal of this initiative is to gather help from the community
to try and push this initiative along so that there is a viable server rendering solution in
place for Angular2 by the time it ships later this year.

## Why Isomorphic?

I [talked about this at ng-conf](https://www.youtube.com/watch?v=2fx4F0tdvN0) and
[wrote a blog post](https://medium.com/p/more-better-unified-javascript-9384fc77ed5c) but there are three primary
reasons why you should care about making Angular Isomorphic:

1. UX (and thus SEO) - Most SPAs (including Angular SPAs) [are slow during the initial page load](http://www.filamentgroup.com/lab/mv-initial-load-times.html). Server rendering is the easiest way to get the initial page load consistently under 1 second. This has a huge impact on user experience and thus often increases your search engine ranking.
1. DRY Code - Sharing common code among different layers of your stack makes your app easier to maintain.
1. Supporting Legacy Browsers - When done properly for certain use cases, you can use server rendering to support legacy browsers. This would allow (again, for some uses cases) you to support IE6 - IE10 despite the fact you are using Angular2 (which only targets IE11 and above).

## Goals of this Initiative

This section will outline goals we are trying to achieve with this initiative. These may change over time as we
learn more about Angular2 and what the community needs.

* JavaScript only
    * The Angular core team has in mind a long term solution that would enable server rendering with any back end (i.e. not just Node.js but Ruby, Java, .NET, etc.).
    * This initiative will ONLY be for Node.js because it will be easier and more powerful (i.e. because of DRY code).
* Basic solution available through Angular core libraries
    * Potentially extra libs just to handle more advanced use cases, but want to limit code outside core.
    * Any libraries created outside core should be considered temporary until a more permanent solution is created in core
    * Probably the only libraries that may make sense to live on would be ones focusing more on server side specific functionality
* @server: true
    * Basic rendering should be as easy (for devs) as configuring the server + putting a server:true annotation in any top level components you want to pre-render on the server side.
* Isomorphic data retrieval
    * Isomorphic view rendering requires isomorphic data retrieval
* Server framework agnostic
    * While this will be only for Node.js back ends, we should be able to easily use any server side framework (i.e. Express, Hapi, Koa, etc.)

In addition to these primary goals, there are several secondary goals that we will try our best to include in the final solution:

* Extensibility
    * There are several situations where you want to be able to add server-only code. For example, server side re-directs or security checks. The solution should ideally have a way for the server to hook into extensibility points for this.
* Server only / client only
    * It should be easy to define a component or part of a template should only be rendered on the server or only rendered on the client.
* Anonymous server side
    * Depending on the app, it can be hugely beneficial to allow the server to render for an anonymous user all the time so that the server never has to deal with state (and thus more caching, more efficient processing, etc.)
    * Should be easy to switch back and forth between context-aware and non-context-aware server side rendering
* Server side caching configuration
    * To improve server side rendering performance, there should be annotations that allow developers to configuration different levels of caching

## Approaches

###### Approach 1 - Generate Artifacts

The approach that the Angular core team is taking for the long term solution is one where you can generate a set of artifacts
from your Angular2 app that can be fed into any back end. The onus with this type of solution is on the back end to implement
routing, data retrieval, rendering workflow, etc. for those artifacts.

###### Approach 2 - Run Angular on the Server

The approach we are looking at here is slightly different. We want to actually execute the JavaScript code for an Angular
app on the server with Node.js. This includes both the Angular core code (as much of it as we can) as well as the custom
code for components, services, etc.

###### Overlap with Approaches

1. Both approaches will require the server generated page to include some serialized JSON data in the DOM that can be used by Angular to hydrate the view on the client side
1. Should hook into the same annotations and supporting configuration. So, if you do @server:true, approach #1 will use that to generate artifacts while approach #2 will use that at run time
1. There will be an interface that needs to be implemented on the server side with both approaches. The primary difference, however, is that with approach #1, the implemented server code will be much larger than with #2 (i.e. because with #1 you are implementing your own data retrieval, etc.)

###### Benefits of Approach 2

The primary benefit of approach 1 is obvious. It can be used with any back end. There are many good reasons for approach 2, however, including:

1. It isn't just about server rendering. You can share any and all JavaScript code throughout your stack.
1. Data retrieval with approach 1 may be an issue. I am not sure how they are going to solve that. With approach 2 it is simply a matter of ensuring all data retrieval is in a specific function in your component.
1. Server or client only variations. There is a lot more power in approach 2 for making small variations on the client or server to suite the needs of your app.
1. It should be a lot easier to implement approach 2 than 1.  In fact, I think that the Angular core team could use the work from this effort to help create a solution for approach 1 down the road.

## Issues to Address

This is brain dump of potential issues to address. Some of these come from ignorance as we are still learning
Angular2. Other issues are going to be problems regardless of what is in Angular2 (i.e. challenges with isomorphic in general).

* Routing - Can we run the new router on the server? Are there any restrictions when we want server rendering? Can the router wait on a resolve for async?
* Data retrieval - Do we need to have a dedicated function for all components or can we just execute the entire component? My experience, FYI, is that since there are many parts of a component that are client only (i.e. event handlers), you need a dedicated data retrieval function that can be executed on the server or client.
* View hydration - What exactly is needed to hydrate the view and eliminate re-rendering? How do we format that in the server rendered HTML? Does Angular2 have mechanism to trigger view hydration from the server?
* Adding annotations - How do we add custom annotations? How does the server read annotations?
* Server rendering - How much of Angular core can we re-use?
* Stuff outside Angular - Assuming we can get Angular core to do the bulk of rendering and we can re-use the router to some degree, what would be left to implement on the server to get this working with Express, Hapi, etc.
* Research Ember FastBoot - It is clear how React does server rendering, but we should do research into the low level details of Ember's server rendering solution to see how they solved some of these problems.

## Logistics

Right now we are in discovery/research mode as we get familiar with Angular2 and just try to get a handle on what is possible. We will open up
issues in this repo for each area of research so we can have an open discussion.

As of 3/14/2015, this is just a small discussion between [PatrickJS](https://github.com/gdi2290) and [jeffwhelpley](https://github.com/jeffwhelpley) but
we are looking for other developers that are interested in helping out. Please ping us on Twitter.
