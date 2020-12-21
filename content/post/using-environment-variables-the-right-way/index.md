+++
title = "Dispel the curse of environment variables"
date = 2020-12-02T10:26:54+08:00
author = "stanleynguyen"
keywords = ["environment variables", "application", "configurability", "12 factor app"]
cover = "posts/using-environment-variables-the-right-way/img/cover.jpg"
summary = "How to use environment variables the right way"
+++

Environment variables is one of the foundational concepts for application developers and something we use on a daily basis.
Environment variables even cemented a part in [the de-facto twelve-factor app](https://12factor.net/config).
We have a long list of benefits that includes applications' configurability and security covered in a lot of literatures like [this one](https://hyperlane.co/blog/the-benefits-of-environment-variables-and-how-to-use-them), or even [this one from StackOverflow](https://serverfault.com/questions/892481/what-are-the-advantages-of-putting-secret-values-of-a-website-as-environment-var).

Using environment variables is definitely great and I'm completely behind the idea.
However, everything comes at a cost and environment variables, if wielded carelessly, can have harmful effects to our codebases and applications too.

## The curse of environment variables

How could environment variables is a bad thing if they help us write more secure code and have an easier time configure our applications for different environments?
Funny enough, environment variables' shortcomings actually derive from their very own natures that make them great: global and external, through which application developers are able to inject configurations and manage these secrets somewhere that's more difficult to compromise.

As developers, we all know how bad global states are to our applications.
And please don't take my word for it, these evils have been discussed in a lot of places like [here](https://softwareengineering.stackexchange.com/questions/148108/why-is-global-state-so-evil), [here](https://thevaluable.dev/global-variable-explained/), and [here](https://stackoverflow.com/questions/19209468/why-is-global-state-bad).

For this article, I will be focusing on the 2 main flaws that I mostly encounter when dealing with environment variables:

- Inflexibility / Poor testability
- Code comprehension / readability

## How to dispel this curse?

Just like how I often deal with global variables or global patterns like singleton applied in bad locations , my favourite weapon is [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection).

It's not going to be the exact same thing as what we do to code dependencies, but the principles are the same: Instead of using the environment variable (dependency) directly, we inject them at callsites (i.e. the place they are actually used) and hence inverse the relationship from "callsites depending" to "callsites requiring".
Dependency injection would solve the afore-mentioned issues by:

- allowing developers to inject configurations more easily at test time
- reducing the mental scope for code readers to the package only, eliminating all externalities

## How to apply the above principles?

I will be using a Node.js example to demonstrate how we can refactor a codebase and eliminate scattered environment variables.

### Hypothetical situation

Let's say we have a simple app with a single endpoint that will query for all TODOs in a PostGres database.
Here is our database module with environment variables scattered into it

```js
const { Client } = require("pg");

function Postgres() {
  const c = new Client({
    connectionString: process.env.POSTGRES_CONNECTION_URL,
  });
  this.client = c;
  return this;
}

Postgres.prototype.init = async function () {
  await c.connect();
  return this;
};

Postgres.prototype.getTodos = async function () {
  return this.client.query("SELECT * FROM todos");
};

module.exports = Postgres;
```

and this module will be injected into our HTTP controller through the application's entrypoint

```js
const express = require("express");
const TodoController = require("./controller/todo");
const Postgres = require("./pg");

const app = express();

(async function () {
  const db = new Postgres();
  await db.init();
  const controller = new TodoController(db);
  controller.install(app);

  app.listen(process.env.PORT, (err) => {
    if (err) console.error(err);
    console.log(`UP AND RUNNING @ ${process.env.PORT}`);
  });
})();
```

Glancing at the above entry point file, we have no way of telling what the app's requirements for environment variables (or environment config in general) are (minus point for code glanceability 👎 ).

### Refactoring

The first step to improve the previously laid-out code is to identify all locations that environment variables are being used directly.
For our specific case above, it's pretty straight forward as the codebase is small, however, for larger codebase, linting tools like [eslint](https://github.com/eslint/eslint) could be used with rule forbidding environment variables (like `node/no-process-env` from [eslint-plugin-node](https://github.com/mysticatea/eslint-plugin-node#readme)) to scan for all locations that use environment variables directly.

Now it's time to remove environment variable direct usages from our app's modules and include these configurations as part of the module's requirements

```js
...
function Postgres(opts) {
  const { connectionString } = opts;
  const c = new Client({
    connectionString,
  });
  this.client = c;
  return this;
}
...
```

These configurations will then be supplied only from our application's entrypoint

```js
...
const db = new Postgres({
  connectionString: process.env.POSTGRES_CONNECTION_URL,
});
...
```

It's much clearer what the environment requirements for our application are now looking at the entrypoint.
This would prevent potential problems with forgotten-to-be-added environment variables.

The full source code for the above demo can be found [here](https://github.com/stanleynguyen/dispelling-environment-variable-curse-demo)

## Bonus: Frequently-asked-questions

These are some of the questions that **me thinks** might be asked by whoever reading this post, not actual frequently asked question but hey what's the harm in addressing possible alternative opinions?

### Why not a central config file/module

I have seen quite a few attempts to solve this problem using a central location for drawing out these value (like a `config.js` file/module for Node projects).
However, this approach is no better than actually using the environment variables provided by application's runtime (i.e. `process.env`) because everything is still consolidated in a somewhat global state (a single config object used throughout the app).
It could be even worse as now we're introducing another location for code to rot.

### What if I want a zero-config setup for my module

Yes, who doesn't love zero-config, ready-to-roll modules.
Again, I would like to re-iterate that building software is all about making trade-offs and this comes at a cost of readability that the whole post has been discussing.
If you would still like a possible zero-config setup, I would like to suggest to have config objects (i.e. the `opts` constructor argument in the prior code example) and direct environment variable usage only as a fallback, something like

```js
function Postgres(opts) {
  const connectionString =
    opts.connectionString || process.env.POSTGRES_CONNECTION_URL;
  const c = new Client({
    connectionString,
  });
  this.client = c;
  return this;
}
```

This way, readers of our code would still be able to recognise (although with less glanceability as it's been traded for zero-configurability) the module's requirements.
