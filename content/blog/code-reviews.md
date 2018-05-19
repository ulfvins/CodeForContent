+++

author = "Richard Ulfvin"
categories = ["Quality"]
tags = ["Code","Patterns"]
date = "2017-02-10"
description = "My experience in the field"
linktitle = ""
title = "Running code reviews"
type = "post"
draft = true

+++

Every now and then I do code reviews. And this time I did some reflection and wanted to share some experience on the topic.

I like the fact that someone reaches out to me to get their work reviewed and I always feel humble and honored to take on this kind of work. I know there might be tension in some teams during these kinds of sessions and I always have an open mind to why the codebase looks like it does. Well in advance I like the team to check for things that might already be to their knowledge (like comments or things they just have not fixed yet) and have that information at hand during the review. They might even write down some lines on why they took on the patterns and practices that the code is built upon and that always lead to a better understanding.

When running the review, I always like to make sure that there are both new team members and the more senior once presence to help the process with better distributed experience. I also like how the discussions get going and find out how the team think, act and tackle challenges together. There are also many things new developers tend to pick up, like new language syntax or newer libraries to use that could benefit both the quality and readability of the reviewed codebase.

I tend to start a review session with some small talk on what tools the team uses, if they have something like ReSharper or StyleCop and what their general opinion is on the default rules in such tools. It also reviles something about naming convention etc. that the team might have decided on. This can be opinionated decisions (inside the team) and if the team is consistent in how they apply things. So, I always adapt to the situation. Some of the things I might come across is the absence of the "Async" word on task method names or that they don't use the "I" prefix on interfaces, and that's fine, as long as it's consistent.

Next, I focus on what tests the team has written for the reviewed code. It tends to show what are important to them from a quality or functionality perspective or even if they use something like TDD as a widespread practice. It also lead-in to some great starting points where in the code to look for things the team might be extra proud of. Another great aspect is what kind of test the team produces like unit-, integration- or even end-to-end tests.

There are always the basic things to look out for like:

* Checking for Null Reference Exceptions
* Proper Exception Handling, with try/catch finally blocks.
* How logging is implemented
* How classes are marked and how immutability is implemented.
* How code execution paths are implemented.
* And how the team in general work with defensive coding habits.

I like to ask about patterns and principals as the review goes along. You might find that SOLID is adopted or other things like CQRS etc. That might also help to quicker see how and where these practices are used and that everybody is onboard on how to use them.

In the end, the code should function, have clear readability and also be secure and effective.
