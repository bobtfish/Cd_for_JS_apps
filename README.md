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

# Prior art

## Code loading

Require.js etc - there are whole hosts of libraries to help you load Javascript libraries dynamically
and do the path prefix per version thing etc, however there isn't any (known to me) prior art to help
with upgrading whilst not reloading

## DB migration / API versioning patterns

Some of these problems can be solved in the same ways as DB migrations, and or API versioning - which
work well, however these all impose non-trivial to implement at all times, and very non-trivial to
prove in CI or get right at all times.

# Common patterns for fixing these issues

## Facebook

People are entering short(ish) text strings as user input, and so it just reloads
the entire page regularly, avoiding this issue.

## Google docs

Persists the entire client side state to the server. Causes force reload on server side
code upgrade.

## Not good enough?

It feels like there must be a better way to do things than this - or at least we're missing
well known writeups about why this is hard / what the various tradeoffs are...

Seems to me to be worth spiking some solutions to these, just to prove out what is or isn't
easily possible.

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

## Upgrading local storage

If you're using client side local storage, then new application versions can
imply that you need a new data schema for local storage. In the minimal case where
you are using local storage as a cache, you can avoid this by just embedding a version number
and flushing the storage on upgrade. However in some applications this
can be much too slow (or much too costly to consumers, for mobile apps!)

Other approaches would be to use a rails DB migration like strategy, so
that when you reload your application model from local storage, you upgrade all of the
objects at that point (by having specific upgrade javascript code).

## Javascript reloading

This one is the real issue - whilst Javascript is highly dynamic, just replacing all the
modules in your app in place is unlikely to work. Other solutions such as loading everything
as new classes and transitioning over _should_ be theoretically possible, however this causes issues
with memory leakage as Javascript isn't (I believe) unloadable.


