+++
title = "To log, or not to log, that is the question."
date = 2020-06-14T13:13:55.496Z
author = "stanleynguyen"
keywords = ["logging", "devops", "architecture", "observability", "reliability", "productivity", "code quality"]
cover = "posts/when-to-log/img/salt-logger.jpg"
summary = "An alternative logging strategy to make loggers your friends, not enemies"
+++

Logging is one of the those things are universally present in software projects with their different, unique forms (business & technical requirements) and flavors (technical stacks).
Logging is everywhere, from small 1-person-startups' products to large enterprises' systems, or even a simple algorithmic programming question where it's needed for checking outputs at implementation steps.
We are relying so much on log for develop, maintain, and keep our programs up and running, however, not much attention has been paid on how to design logging within our systems.
Often times, logging is treated as a second thought and only sprinkled into source code upon implementation like some magic powder that helps lighten the day-to-day operational abyss in our systems.

{{<figure src="img/salt-logger.jpg" width=600 >}}

Just like how any pieces of code written will eventually become technical debts - a process that we can only slow down with great discipline, loggers rot at an unbelievable speed that after a while, we find ourselves fixing problems caused by loggers more often than loggers giving us information to debug our operational faults.
So how can we manage this mess called loggers and turn them into one of our allies rather than legacy ghosts hunting from past development mistakes?

## "State of The Art"

Before I dive deeper into my solution proposal, let's define a concrete problem statement based on my observations.

So what exactly is logging? An interesting and on-point one-liner that I found from [Colin Eberhardt's article](https://www.codeproject.com/Articles/42354/The-Art-of-Logging)

> Logging is the process of recording application actions and state to a secondary interface.

This is exactly how logging is weaved into systems. We all seem to subconciously agree with loggers not belonging to any layers of our systems but rather being application-wide and shared amongst different application components

{{<figure src="img/stackoverflow-answer.png" caption="A much approved answer on StackOverflow">}}

A simple diagram where logging is fitted into a system that is designed with [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) would look something like

![Logging in Clean Architecture](img/logging-arch.png)

We can safely say that logging itself is a subsystem within our application.
And that, without careful considerations, often spirals out of control faster than we think.
While designing logging to be a subsystem within our applications is not wrong, the traditional perception of logging (with 4 to 6 levels of `info`, `warn`, `error`, `debug`, etc.) often makes developers focus on the wrong thing - the format of our logs, rather than the actual purposes why we are writing logs.
This is the reason why we are logging errors out without thinking twice about how to handle them, why we are logging at every step of our code while ironically unable to debug effectively should there be a production issue.

This is why I am proposing an alternative thinking framework about logging and in turn, how we can design logging into our system reliably.

## The Good, The Bad, and The Ugly

This is a framework on how I think we should strategise our logging with three and only three categories / concerns for our logs.

### First rule of logging: Don't log

Overlogging is detrimental to our teams' productivity and ability to handle business-as-usual operations. There're tons of reason why we should not "log whenever you can" as advised by some observability fanfare.
Just to name a few: logging means more code to maintain, logging incurs expense in terms of system performance, and more logging subject us to more data privacy regulatory audits.
If you need more reasons to refrain yourself from logging, check out [this post by Nikita Sobolev](https://sobolevn.me/2020/03/do-not-log) or [this post by Jeff Atwood](https://blog.codinghorror.com/the-problem-with-logging/).

Nevertheless, I'm not advising eliminating logs altogether.
I think logging, used correctly, can significantly help in keeping our system running reliably.
I'm proposing we start without log and work our way up to identify places where we need to log, rather than "log everywhere as we **might possibly** need to look at them".
My rule of thumb for adding a line of log is "if we can't pin an exact reason or a scenario that we will look at the log content, **don't log**".

With that being said, how can we safely introduce log when it's absolutely necessary?
How should we structure our logs and format their content? What are the necessary information to include in logs?

### The Ugly

This is the first type of logs that I want to describe, and they are also ones with the least frequency that I would expect (If we find it otherwise, we might have bigger issues in our systems!).
"The Ugly" is the kind of log under catastrophic or unexpected scenarios that require immediate action like catastrophic errors that need an application restart.
We can argue that, under these circumstances, it makes more sense to use alerting tools like Sentry.
Nevertheless, an error log might still be useful to provide ourselves with more contexts surrounding these errors which are not available in their stack trace but could help with reproducing these error like users' inputs.
Just like the errors that they accompany, these logs should be kept to a minimum in our code and placed in a single location of the code.
They should also be designed/documented in the spec as a required system behavior for error handling, and weaved in source code next to where error handling happens.
While format and level for "The Ugly" logs are completely preferential on a team-by-team basis, I would recommend using `log.error` or `log.fatal` (before a graceful shutdown and restart of applications) attached with the full error stack trace and functions' or requests' input data for reproduction if necessary.

### The Bad

"The Bad" is the type of logs that addresses expected, handled errors like network issues, user inputs validation.
In the same manners as these expected errors, "The Bad" logs only require developers' attention if there's an anomaly of occurences
Together with a monitor set up to alert developers upon anomolies, these logs are handy to mitigate potential serious infrastructure or security problems.
This type of logs should be spec-ed inside error handling technical requirements as well, and can actually be bundled if we are handling expected and unexpected errors in the same code location.
Based on the nature of what they are making "visible" for developers, `log.warn` or `log.error` can be used for "The Bad" logs as teams' convention.

### The Good

Last but definitely not least, "The Good" is the type of logs that would appear most often in our source code while being the most difficult to get right.
"The Good" kind of logs are those associated with the "happy" steps of our applications, indicating the success of operations.
For its very nature of indicating starting/successful executions operations in our system, "The Good" is often abused by developers who are seduced by this mantra "just one more bit of data in the log, we might need it".
Again, I would circle back to our very first rule of logging: "Don't log unless you absolutely have to".
To prevent this abusive pattern from happening, we should document "The Good" as part of our technical requirements complementing the main business logics.
On top of that, for every single one of "The Good" logs to be inside our technical specs, it needs to pass the litmus test of whether there are any circumstances under which we would look at the log - be it a customer support request, an external auditor's inquiry.
Only this way, `log.info` won't be a dreaded legacy that obscure developers' vision into our applications.

### The Rest (that you need to know)

By now I assume you would have noticed the general theme of my proposed logging strategy revolves around clear and specific documentating logs' purposes.
It is important that we treat logging as part of our requirements, and be specific about what keywords, messages we want to tag in log context for them to be effectively indexed.
Only by doing that, we can be aware of each and every piece of logs that we produce, and in turn a clear vision into our systems.

As logs are upgraded to first-class citizens with concerete technical requirements inside our specs, the implications are that they would need to be:

- maintained and updated as the business and technical requirements evolves
- covered by unit, integration test

These might sound like a lot of extra work to get our logging right.
However, I argue that this is the kind of attention and effort logging deserve so it can be useful.

**Serve our logs, and we will be rewarded splendidly!**

## A practical migration guide

I reckon there's no use for a new logging strategy (or any new strategies/frameworks if that matters) for legacy projects if there's no way of moving them from their messy states to the proposed ideal.
Hence, I have a three-step general plan for anyone who are frustrated with their systems' logs and willing to invest their time to log more effectively.

### Identify The Usual Suspects

Since the idea is to reduce garbage logs, our first step is definitely identifying where the criminals are hiding.
With the powerful text editors and IDEs we have nowadays (or `grep` if you are reading this in the past through a window-to-the-future), all occurences of logging can be easily identified.
A document (or spreadsheet if you would like to be organised) documenting all of these logging occurences might be necessary if there are too many of them.

### Convict them bad actors!

After identifying all suspects, it's time to weed out the bad apples!
Logs which are duplicated, unreachable are low hanging fruits that we can immediately eliminate from our source code.
For the rest of our logging occurences, it's time to involve other stakeholders like the "inception" engineer who started the project (if that is possible), product managers, customer supports, compliance folks to answer the question: Do we need each one of these logs, and if so, what is it being used for?

### Light at the end of the tunnel

Now that we have a narrowed-down list of absolutely necessary logs, turning them into technical requirements with documented purpose for each one of them (i.e. what to do when a `log.error` happen? who are we `log.info`-ing for?) is essential to nail down a contract (or we can call it specification) for our logging subsystem.
After this, it's just a matter of discipline, in the same manner as how we write and maintain software in general, to make our logging great again!
