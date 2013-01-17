# Premis

## tl;dr

Given heavy Javascript apps which people keep open for hours (or days) at a time, how do you deploy
multiple times a day without screwing the user over or making the developer's life really hard? Discuss!

## In more detail

Reliable continuous deployment for stateless web applications can be done reasonably easily. For example:

  - Have 2 instances (A and B) of your application on each server
  - With A instances in the load balancer, upgrade green instances
  - Add B instances to the load balancer
  - Remove A instances from the load balancer

You scripting this little dance isn't too hard, nor is orchestrating it across the multiple machines
with mcollective.

My company even has a little project to wrap all of this up: https://github.com/youdevise/orc/

However, for JS heavy applications a lot of state is preserved on client side, and most of the site
just becomes a REST API serving JSON..

Maintaining an inter-version boundary between the application API and what the Javascript expects is really
quite hard on developers, and doesn't help solve the fact that you want to be able to upgrade the Javascript
multiple times a day also.

Lets discuss application architecture strategies for solving this, and the inherent tradeoffs involved in each.

# Issues

There are three main possible issues we identified when live upgrading apps

## Assets from various apps loaded.

Even if you compress your Javascript and CSS into a single file each, it
is possible to get the wrong version for CSS / Javascript.

The usual fix for this is to make each version of your app have it's assets
at a different URI prefix, and to make version N+1 also serve the assets for
version N.

## Old frontend talking to new backend.

You can make the front => backend application API be compatible at all times
This introduces a non-trivial development requirement, and a non-trivial
testing / continuous integration required.

# Application useage scenarios

## Facebook

People are entering short(ish) text strings as user input, and there

Google docs

Other - streaming etc

Event sourcing apps

# Prior art

Require.js

DB migration patterns


# Problems

## CSS reloading

We believe this is fairly easy, at least if you're using less.css to generate
your CSS files files for you, you should be able to combine the CSS for
version 1 and version 2 of your app, and then generate a 'diff', i.e. CSS to
apply to the page to take your CSS from version 1 to version 2

### Issues with this

This obviously means that you can't change the semantic tags you're using on
your HTML, otherwise your new CSS rules won't apply, or won't apply as expected.

There are a whole class of cases where this strategy won't work unless you're
also regenerating all of the generated markup for your application to include
new classes / ids etc.

Javascript reloading

Upgrading local storage


