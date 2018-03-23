# architecture-qualities

Most sources will agree that architecture qualities are those
cross-cutting concerns that are not strictly related to the
functionality of a system: availability, scalability, usability,
portability, testability, etc. As usual, different groups created
different sets of these terms, and fought viciously about which ones
were canonical and which were heretical. Everyone agrees that most of
them are words that end in -ility, but that's where agreement ends.

This page lays out a common set of conventions that we can use to make
sure we're all speaking the same language.

You will find that some of these attributes are in tension with each
other. For example, flexibility and maintainability are usually at
odds. That's why we have these conversations, so we can find out the
customer's priorities and jointly decide on the right tradeoffs and
balances.

Drawn from "Software Architecture in Practice", by Len Bass, Paul Clements, and Rick Kazman.

# Qualities Observable at Run-time

These qualities are things you might be able to test or measure while
the system is in operation.

## Performance

Response time of a single transaction, while the system is under a
certain level of load. Better performance equates to faster response
times.

We obviously expect performance to degrade as load increases. The
performance of a single kind of transaction (i.e., page request) may
also depend on the workload---the mix of transaction types being
requested.

It's best to express performance as an envelope rather than a single
number. For example, "95% of search requests must complete in less
than 800 ms, and 99.9% of them must complete in less than
1500ms". That's a fairly lenient requirement. A tougher one would be
"95% of API calls in under 100ms, 99% in under 200ms." Notice that
each of these requirements is a list of (quantile, time) pairs.

## Security

Three dimensions of security: data confidentiality, data integrity,
and data availability. (This is the "CIA model", but nothing to do
with government spooks.)

Express each of these in terms of vulnerability, threat, likelihood,
impact. Vulnerability is the "thing which could happen, and
how". Threat is "who is the bad guy". Likelihood is the odds of any
given attack on that vulnerability succeeding. Impact is the damage
caused if it does succeed. E.g.:

   - Vulnerability: The server is under Bob's desk. Attacker could
     gain physical access to DB.
   - Threat: Staff, cleaning service, building maintenance, Tom
     Cruise.
   - Likelihood: high.
   - Impact: Beanie baby inventory exposed to auction snipers
     everywhere, leading to complete loss of profits.

## Availability

In classical definition, availability refers to the probability that a
request at arbitrary time T will be processed.

For high-volume systems, we can unify availability and performance via
the performance envelope described above. Any response which takes
longer than the outermost threshold is identical to the system being
unavailable. (From an external observer's perspective, there's no
difference between "too slow" and "down".)

There's another dimension of availability that we should look at,
though, which is availability during events. Some questions to answer
in this dimension:

- Is it acceptable to bring the system down for deployments?
- If so, during what hours or what days of the week?
- Which features require higher availability than others? (E.g., "The
  system must always accept mail, even if it cannot be delivered.")
- If the system requires HA, how many uncorrelated failures must we
  withstand?

Also be sure to check on availability requirements for back-end,
administrative or content production systems. These often get short
shrift, but create just as much angst around downtime as the front
end. (Especially true if the system is part of a service provider with
external users for those production tools.)

## Usability

On the spectrum of complexity from "airline gate agent system" to
"Grandma's photo sharing", where should this system be? Most web
systems will require very high usability. A common, if unstated,
requirement is "the system should be discoverable, learnable, and easy
to use for anyone on Earth, with no training at all."

It will be rare for us to encounter a system where training the users
is even an option.

# Qualities Not Observable at Run-time

The other set of qualities has more to do with the creation and
evolution of the system, rather than it's live behavior. As such,
these are often much harder to measure.

Beware! These qualities are a common source of mismatched expectations
between a developer and a sponsor. The sponsor may expect to
automatically get every one of these qualities, and may even use
phrases like "any responsible professional would have X, Y, and Z!"

## Scalability

Scalability is the marginal increase in capacity resulting from
additional computing resources. It's closely related to performance,
but not quite the same.

Suppose you have 10 servers and can handle 12,000 requests per second
across those servers, while maintaining a 100ms response time. That's
your capacity at a specific level of performance.

Now suppose you add 2 more servers. What is your new capacity in terms
of requests per second? It could be more, less, or the same as the
original 12,000 rps.

   - Increased: If you have perfect, linear scalability, then you
     should have a new capacity of 14,400 rps. (Tip: nobody has linear
     scalability.)
   - Unchanged: If you have zero scalability, then you still have
     12,000 rps. In other words, you're bottlenecked somewhere else.
   - Decreased: You might find that the 2 extra servers reveal a
     horrible bottleneck and resource contention causes your capacity
     to drop to 9,500 rps. In this case, you'd actually describe the
     system as negatively scalable.

The only reason anybody cares about scalability is to guarantee the
same or better performance as the capacity increases. Everything else
you read about scalability is about how to achieve that.

At small scales (up to 100 nodes or so), you don't need to worry too
much about the dot product of scalability and availability. Don't
worry about the CAP theorem and all that stuff unless you need
to. There are a lot of large sites that run just fine on relational
databases.

Represent scalability requirements in terms of an envelope of
growth. How large can the system get, in what dimension (users, data
set, transactions, requests per second), and with what kind of
scaling. Most of the time, you'll represent this with a "more of the
same" requirement: must be able to add application servers, web
servers, search servers, etc.

Tip: if the requirement says something about 10x growth with the same
architecture, you need to have a talk with your customer. Some
significant amount of rearchitecture will definitely be needed. 10x is
not a "more of the same" requirement.

## Modifiability

We do a lot better these days on modifiability than when all this work
on architecture started. Still, there are some kinds of changes which
are easy and some which are big.

Ask your customer about their business model. Find out what their
primary revenue and cost drivers are, then make sure that the system
is most modifiable along those dimensions.

These requirements will usually be expressed in terms of (kind of
change, time needed, by whom).

Some examples of big things to look out for:

   - Will their customers be charged on a flat fee or for usage?
   - Does the system need to be skinnable? If so, who should be able to do it and how long should it take?
   - Single tenant? Multi tenant? Instanced? By whom?
   - What kind of business rules can change?

Examples:
   - A very stodgy shoe retailer changes their pricing rules once a
     quarter, but never, ever discounts anything. They're OK with us
     making the change. (So we just do pricing rules in code, and
     accept that it takes part of an iteration to update them.)
   - A giant shoe retailer does promotions that last a week, decided a
     month in advance, with rich media and content. (So we build this
     into the regular content pipeline.)
   - A freewheeling shoe retailer does spur-of-the-moment, carnival
     style promotions. Two minutes from idea to activation, but the
     promotion only lasts for an hour. (So we build a UI for a segment
     manager to create the promotion, make the pricing rules more
     dynamic, create an event queue to publish the promotions in near
     real time to the site, etc.)

## Portability

Not usually an issue for us. Would only matter if we build a desktop
app or embedded software.  Could become significant if we build mobile
apps that have to support iOS and Android from the same services.

## Integrability

None of the systems we build exist in a vacuum; they always integrate
with an existing environment. Generally speaking, this means either
providing a service to or consuming a service from another system.

The two main concerns when you design for integration are:

- Technology choices: Use languages and protocols that are acceptable
  to the other party (e.g., XML or JSON over HTTP is widely usable but
  doesn't always provide the necessary semantics or performance; gems
  only work for Ruby, and maybe not for JRuby; etc). In many cases
  these options will be dictated by the other party, especially if
  it's a client's internal system. Whatever choices you make, document
  integration interfaces precisely.
- Dependency management: Establish a relationship with the team(s)
  responsible for the system(s) you're integrating with. This is
  critical at dev-time in order to resolve problems uncovered while
  coding or testing, understand when new versions are moving into
  staging and then production, etc. It's also critical at run-time in
  order to resolve quality issues (e.g. performance, availability,
  etc).

This is a tar pit of first rank. Beware of any integrability
requirements on your system and handle them carefully!

## Reusability

In some cases, a customer will want their system or a piece of their
system to be reusable. In many ways, this is just a variation on being
integrable (see above). The difference is that reusability implies a
broader audience for a provided service that magnifies integrability
concerns.

Represent these in terms of the features that need to be reused, the
level of semantic and syntactic coupling which is allowed, the scope
of the population who is allowed to reuse this, and the time frame
that the re-user should expect.

E.g.:

   - "Flight search by city" should be usable by any application
     within Braniff Airlines, without permission, and with less than 1
     month of integration effort.
   - "Bookface connect" must be usable by any application, written by
     anyone, anywhere in the world, with less than 1 hour of time to
     integrate. Changes to Bookface Connect must be transparent to all
     users and integrators.

Obviously, these two different requirements would motivate different architecture choices.

Sometimes the idea to reuse a system comes up later in the development
cycle, after initial design work has been done. When this happens, you
have to revisit both design and implementation choices to ensure they
still apply and/or identify changes that need to be made. For example,
if a client decides to reuse a system created to transmit messages
with 50MB files and fire-and-forget semantics to send 1GB files with
exactly-once semantics, changes will probably be required!

## Testability

The ease with which pieces of the system can be made to demonstrate
its faults.

Our web based systems have high inherent testability. Our use of unit
and behavioral testing also helps guarantee we create testable
systems. When you do need to document these requirements, though, make
sure to capture: feature and component to be tested, in what
environment, with what confidence of detecting a fault. These
requirements may motivate you to split or merge components in order to
get good observability and controlability. They may also motivate you
to create stubs or test harnesses to isolate components at execution
time.
