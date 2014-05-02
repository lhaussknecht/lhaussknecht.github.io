---
layout: post
title:  "Building Webanalytics with elasticsearch"
date:   2014-04-28 18.01:00
categories: elasticsearch analytics
---

A client of mine is starting her first web based project. The application is an inhouse google chrome app. One of our
requirements are, that we should not depend on third party services, such as Google Analytics.

So the first step for me was to look for of-the-shelf solutions. I tried [Piwik](http://www.piwik.org). Piwik is an
open-source web analytics tool. Piwik looks good, but our decision was against it. Mainly because it depends on
MySql and it is written in PHP, which is absolutely not our technology stack.

After Piwik we looked at our fresh elasticsearch installation.
We are using elasticsearch together with Logstash for our IIS Logs and are really happy.

To be honest, elasticsearch, logstash and kibana (the web frontend) are awesome! The configuration was really simple.
We had to adjust the grok filters to match our "all switches on" IIS logs and were ready to build great looking
kibana dashboards. Elasticsearch handles all the low-level lucene stuff for you!



For a new project I was keen on taking one of the so called Micro-Web-Frameworks for a test drive.
My favourite Framework is NancyFx. Nancy makes just fun!

__It's HTTP in essence.__

But is one fancy new Framework enough for a fresh project? What about persistence? Okay, let's add a some NoSQL flavour to the mix!

I'm following [Ayende](http://ayende.com/blog) for years. So reading about [RavenDB](http://ravendb.net/) in his blog and in the [newsgroup](https://groups.google.com/group/ravendb) made me wanting to get my hands on it.

Nancy and RavenDB are all on Nuget, so getting the bits and pieces together is just:

    Create empty Web Project
    install-package Nancy
    install-package Nancy.Hosting.Aspnet
    install-package RavenDB

I really like the Idea of working with the database directly without having infrastructure code in my Modules. We are talking about "Session per Request". Every request to the server opens a new unit of work. It should be easy to query for data but also to write and modify.

In an MVC application we wire everything up in global.asax. But there's no global.asax in Nancy.

With nancy your controllers are NancyModules.

So my goal is to have a Property called DocumentSession in every Module, so that I can work with my datastore frictionless (<- Kudos to Ayende for this cool notion).

[@TheCodeJunkie](https://twitter.com/TheCodeJunkie) gave me a hint on twitter how to do it.

The idea is to have a hook in the building phase of a module (your controller) to inject the Raven session.
But before injecting something you need a source for your session:

{% gist 1124884 %}

We lazily initialize a static DocumentStore. Creating a DocumentStore should happen only once in your application. The DocumentStore provides IDocumentSessions.

I've extracted the interface IRavenSessionProvider from this class.

Now we need to tell Nancy that she sould use this provider to add the session to our module. In your NancyBootstrapper we customize the construction process:

{% gist 1124901 %}

This tells Nancy to use a custom ModuleBuilder to build our module. And here's where the bird comes to her:

{% gist 1124905 %}

In the constructor we add a dependency on IRavenSessionProvider. Dependencies are resolved by the internal TinyIoc. How cool is that!

BuildModule is where the magic happens. We request a new IDocumentSession and store it in the Request Context. When our request ends we want to save all changes and get rid of our session. So we add this action to our AfterRequest-Pipeline.

We can now access our session by accessing the Context-Items. But this is inconvenient and I want to stick with the DRY principle. So lets factor this out.

{% gist 1124973 %}

This gives us a base class with the property DocumentSession. And finally here's a sample of our Module in action:

{% gist 1125166 %}

We have two actions in this Module. If the module receives a HTTP GET request it queries RavenDB for Customers who's name start with the given first letters.

The other action is the PUT action. Put creates a new Customer in the database.

All actions in one Request are transactional save by the way. This is ravens "Save by Default" approach.

For my project it seems like the perfect combination (+ I can play with cutting edge technology)!