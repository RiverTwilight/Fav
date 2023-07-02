> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [vadimdemedes.com](https://vadimdemedes.com/posts/generating-income-from-open-source)

> Ink has been getting some good traction lately and it's already being used by some well-known compani......

[Ink](https://term.ink/) has been getting some good traction lately and it’s already being used by some well-known companies for a while. Like most other open source projects though, Ink has zero revenue.

I’ve started looking into various options to change that and start charging money for it in some way, so it can support me and further development of Ink and adjacent projects like [Ink UI](https://term.ink/ui) and [Pastel](https://term.ink/pastel).

This post is a compressed version of what I learned on this topic.

What doesn’t work
-----------------

Here’s my opinion why maintainers are struggling to extract revenue from their projects.

### Counting on donations from individuals

It’s great that there are people willing to support you, but donations of $5 per month won’t make a living for you. It’s a great way for community to express gratitude for your work, but it shouldn’t be perceived as stable income.

Unless you’re one of the few super popular developers in the community, accept the fact that there won’t be enough people subscribing to a monthly donation.

That’s ok though, because I don’t think individual donations are the answer anyway.

### Expecting donations from companies

You built a project that companies enjoy using, they run it in production and they’re benefitting from it a lot. Surely they must know it’d be nice to give back and all that, since they got all that money, right? Wrong.

We need to finally understand a few simple truths and change our expectations.

Running a business involves maximizing revenue and minimizing expenses. Businesses won’t add another recurring expense out of goodwill, just to be nice to you.

Businesses are used to exchanging value for money. Open source maintainers need to take that into account. You provide the value, they benefit and pay for it.

There are certainly a number of companies with a strong open source culture who can consistently afford giving back significant monthly donations to projects they rely on. Unfortunately, they’re an exception and not the rule.

We’re asking for donations with more or less similar phrasing:

> Please consider sponsoring me, so I can continue working on my open source projects.

We put up a nice GitHub Sponsors page and sit around waiting until someone signs up. Can you imagine a business taking a similar strategy? It’d go bankrupt in a month and close the shop.

We need to understand the value our projects deliver to companies and start charging money, as if we’re running a business too and selling a useful product.

### Thinking that no one would pay or not charging enough

Having worked at several small and medium size startups, I now understand how foolish I was years ago, thinking that $30/mo is a high price tag for a subscription or companies are hesitant to pay for tools at all. Absolute non-sense.

Companies pay hundreds of thousands of dollars to their employees to solve daily problems and develop their product. If your project solves a problem they have, so their team doesn’t have to do it themselves, they’ll pay 10, 100 or even 1,000 times more than you think it’s worth. Moreover, they’ll be happy about it.

Companies pay thousands of dollars per month for all kinds of tools and expenses already. Whatever you ask, realistically, it’s a drop in the ocean for them. It’s a cliché, but really do double or triple your prices.

### Afraid or ashamed to ask for a credit card

We don’t need to justify why we charge money for our work. There’s nothing to be ashamed of.

You worked hard to build something that solves a problem. Someone has that problem and pays you to solve it. Don’t overthink it.

What does work
--------------

We like to complain how nobody pays the maintainers, but there are actually many successful businesses built on open source. Here are the approaches they took to generate a stable and sustainable income.

### Commercial licenses

[Metafizzy](https://metafizzy.co/) by Dave DeSandro makes various JavaScript libraries, among which is Isotope — a library for creating flexible grid layouts. Isotope is open source, but there are [different licenses](https://isotope.metafizzy.co/license.html) depending on how you intend to use it:

1.  Open source license.

This license allows Isotope to be used in personal or open source projects for free.

2.  Commercial license.

This license permits you to use Isotope in almost any commercial app. Realistically speaking, any company wishing to use it, most likely will need to buy a commercial license.

Commercial license is priced differently based on how many people are going to use it:

*   $25 for one individual developer.
*   $110 for a team of up to 8 developers.
*   $320 for unlimited number of developers.

Note that these aren’t subscriptions, these are one-time payments.

Commercial license itself is a PDF file that’s emailed to you after a payment on Gumroad. It doesn’t get any simpler than this, I think.

3.  Commercial OEM license.

This license enables the remaining use cases, which the previous commercial license doesn’t cover. Specifically, UI builders, SDKs or toolkits. There’s no publicly available pricing for it, which means that it’s much more expensive than previous tiers. These use cases likely imply that Isotope is used as a critically important part of the user interface or product offering, so companies will be ready to pay good money.

#### What I like about this approach

This looks like the simplest possible way to charge for open source, because Metafizzy offers the same code under different licenses and the license itself is a PDF file. There are no Pro versions, no license keys and there’s nothing else to maintain. Individual developers enjoy the same tool for free, while companies pay a reasonable price.

### Charging for more features

[Sidekiq](https://sidekiq.org/) by Mike Perham is a well-known library for Redis-based background jobs in Ruby apps. Sidekiq offers 3 different plans:

1.  Open source.

Sidekiq offers a limited and open source version for free. Even though it’s named as “open source”, the LGPL license seemingly allows you to use the free version in a commercial app.

Open source plan doesn’t offer any customer support, so GitHub issues would be the place to figure things out yourself.

2.  Pro.

Pro plan costs $99/mo (or $995/yr) and offers an extended set of features. For example, batched background jobs, enhanced reliability via more advanced Redis APIs and expiring jobs. Pro plan also includes customer support via email.

3.  Enterprise.

Enterprise plan offers the entire set of features for $229/mo or higher, depending on how instances of Sidekiq you’re running.

Sidekiq has been doing incredibly well and according to Mike’s [recent comment](https://news.ycombinator.com/item?id=35572217) on Hacker News, it’s doing 10 million per year now.

Interestingly, he also mentions that you can assemble most of Sidekiq’s paid features via other open source Ruby gems, but it takes a lot of time to set up and maintain. You’ll likely end up with a worse system than a battle-tested Sidekiq, so purchasing a batteries-included Sidekiq looks like a no-brainer.

> Most of Sidekiq’s commercial features are available as OSS gems but the complexity sneaks up on you as you integrate 3-6 of those features together. Building your own almost always leads to a worse system than the mature, well-debugged system which I have curated.

Once you sign up for Sidekiq, you’ll get access to a private Ruby gem server to download and update the `sidekiq` gem in your app. He built that system himself and he says it doesn’t take much time to maintain.

#### What I like about this approach

Sidekiq is first and foremost a great open source project. It became an obvious choice in the Ruby community when you need a background queue. This is Sidekiq’s one and only marketing channel.

Developers then recommend Sidekiq to their friends and managers at their job. As their apps scale up, customers have a clear incentive to pay for Sidekiq to unlock more features.

### Hosted version

Recently there has been a rise in businesses open sourcing their entire product and offering a hosted version of it for money.

*   [Plausible Analytics](https://github.com/plausible/analytics) — a privacy-focused Google Analytics alternative. Hosted version starts at $9/mo.
*   [PostHog](https://github.com/PostHog/posthog) — product analytics, feature flags, A/B testing and more data tools in one package. Hosted version has a metered billing where a first million events is free and then it’s $0.0003068/event.
*   [Metabase](https://github.com/metabase/metabase) — dashboards for databases. Hosted version starts at $85/mo.

These examples are off the top of my mind, but there are many more like these.

#### What I like about this approach

You can build the app once and offer the same version as both open source and a hosted paid product. You might think “why would someone pay for something available for free”. However, Plausible Analytics earns 1 million a year, so there must be lots of people willing to pay a small monthly fee to enjoy their product without dealing with servers.

### Charging for maintenance and advanced materials

[React Flow](https://reactflow.dev/) by Moritz Klack, Christopher Möller, John Robb and Hayleigh Thompson is a React library for interactive flow charts. It’s a unique example of a sustainable open source project and it’s unlike anything I’ve seen before. React Flow has a [Pro subscription](https://pro.reactflow.dev/) for companies, which offers these features:

1.  Access to a set of Pro examples of advanced use cases.
2.  Prioritized resolution of issues on GitHub.
3.  Up to 1 hour of email support per month.
4.  And most interestingly, I quote, “Keep the library running and maintained under an MIT License”.

Through the whole pricing page, most of the copy is focused on that last bit. React Flow is not a library that’s easy to replace with something else, so companies are most likely interested in making sure that it’s maintained well and continues to be MIT-licensed.

John wrote an excellent post on their blog called [“Dear Open Source: let’s do a better job of asking for money”](https://reactflow.dev/blog/asking-for-money-for-open-source/), which I recommend you to read. I was so fascinated by it, that I emailed John some follow-up questions and he was super kind to reply with a goldmine of knowledge on this topic.

Below are my distilled takeaways from our email thread:

*   **Packaging matters.** Person with a credit card inside the company wants a “Pricing” page they’re used to seeing all the time. GitHub Sponsors page won’t cut it. React Flow had that initially and received very little money. Things changed when they rolled out a product-like website with several pricing tiers as if it’s a SaaS.
*   **Let people discover the Pro plan.** React Flow component shows a link to their website and asks developers to remove it, only if they’re subscribed to a Pro plan. It’s still completely legal and fine to remove it without doing so, but it serves as a nice non-pushy nudge to check out the Pro plan.
*   **Companies feel safer when direct support is available.** React Flow offers up to 1 hour of email support per month, so naturally I asked what happens if customer ends up taking more time than that. John said that they continue emailing despite that and it all evens out in the end, since there are many customers who don’t reach out to them at all. He also thinks that email support creates a feeling of insurance, so companies know they have a door to knock on if they need to, even if they never do it.
*   **Give people something to buy and access right now.** I was wondering how important are those advanced examples available to Pro customers, because compared to other benefits they seem as a nice-to-have. Surprisingly, John thinks otherwise. He’s convinced that offering something of value immediately after purchasing differentiates their Pro plan from a consultancy or a service. It also gives customers a resource that’s consistently available exclusively to them.

#### What I like about this approach

It’s an interesting model to consider, when you already have an established open source project and it’s not viable to start requiring commercial licenses, provide a Pro version or offer a hosted solution. The key idea here is to sell insurance that your project won’t go under and companies can confidently rely on it. This makes React Flow’s approach so interesting, because it might be possible to set this up for pretty much any open source project.

John, thank you for sharing unique insights of React Flow’s setup 💛

### Support packages

Last but not least, you can set up a consultancy around your open source work and offer an expert knowledge to companies who rely on it.

*   [Babel](https://babeljs.io/) offers a $24k/yr plan on their [Open Collective](https://babeljs.io/) page, where companies get access to 2 hours of email or video support per month.
*   curl offers [commercial support](https://curl.se/support.html), which even includes development of custom features and code reviews of how you use curl.
*   [Filippo Valsorda](https://words.filippo.io/full-time-maintainer/) offers retainer agreements to companies for 5 figures a year. Filippo meets the engineers, learns their needs and makes sure they’re fulfilled as he develops his open source software. Filippo is a cryptography expert, so companies can sign up for a more expensive contract to get his expertise on anything related to cryptography, not just his projects.

#### What I like about this approach

Offering paid support to companies keeps your project fully open source, while bringing in more money than a Pro subscription would. I don’t know how hard it is to get going, but it’s an attractive option for someone who’s used to working as an employee.

Conclusion
----------

Every once in a while, the front page of Hacker News highlights another blog post how open source model is broken and maintainers aren’t getting any money from companies who benefit from their work.

Is it fair? No. Can they do something about it? Yes.

There are multiple options available to generate sustainable income and there are many successful examples of people doing it today and been doing it for years. It might work out for you too, you’ll never know until you try.