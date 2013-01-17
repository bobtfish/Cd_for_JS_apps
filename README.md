test
tl;dr - Given heavy Javascript apps which people keep open for hours (or days) at a time, how do you deploy
multiple times a day without screwing the user over or making the developer's life really hard? Discuss!

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
