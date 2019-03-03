---
title: Open Sourcing things
date: 2019-03-03 13:03:03 +0100
categories: software development
---

Over the last couple of months I have been open sourcing various smaller projects on GitHub. Some of these projects were initially private repos and some of them I just had lying around locally on my machine. Giving them a new life in the public has been fun and rewarding.

In this post I want to share more details about these projects.

# [Edifunct](https://github.com/orhantoy/edifunct)

At a previous employer we needed to handle EDIFACT messages. There already exists a few EDIFACT gems but none of them seemed to address the need to *structurally* parse EDIFACT messages. I didn't have the intention to create a gem for this but the outcome was a library that could be used quite broadly to parse any type of EDIFACT message. The resulting implementation was also substantial enough without being trivial, that it ended up making a lot of sense to open source it.
<br>
Also, it was my first gem! 🎉

# [Doküman](https://github.com/orhantoy/dokuman)

This is essentially a mash-up of a Markdown parser, Mustache templating, GitHubs Primer styles and headless Chrome which in combination can produce decent looking PDFs from Markdown. To make the output consistent a `Vagrantfile` has been provided.

I developed and used this at a previous employer to generate API documentation in a PDF format.

# [Timeseddel](https://github.com/orhantoy/timeseddel)

This is essentially just me trying to build something with React that is not based on a tutorial. My approach to learning is that I define the problem myself and then try to solve it with the given technology.

# [Müşteri](https://github.com/orhantoy/musteri)

This project is an example of how I would solve a test assignment that I defined at a previous employer for assessing applicants. The assignment is to create a customer importer with Rails.
<br>
Test assignments and exercises in relation to job applications and interviews is an interesting topic that I hope to dive into in a future post.

# Final words

Looking back, I'm not sure why I didn't more comfortably open source things until now. Obviously as you become more experienced you'll feel more comfortable doing work in the public so starting early is just better in the long term. But I would also say, as I have told a few fellow developers, thinking about what and how you open source your code is important. Having 2 solid projects that you work on on the side and update once in a while is much better than having 20 half-baked projects. Ultimately your GitHub profile is your portfolio so having good READMEs, good commit messages, etc is always a good idea.
