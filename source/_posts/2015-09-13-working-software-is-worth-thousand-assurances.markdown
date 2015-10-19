---
layout: post
title: "Working software is worth thousand assurances"
date: 2015-09-13 19:09:14 +0530
comments: true
categories: agile walking-skeleton MVP
---

In the beginning of a software project, every one starts with a big list of features. If you ask the product owner, what is the minimum set of features that we can go live with? most of the times you'll hear a lot more than you expected.


It gets lot harder to talk about minimum scope when you are rewriting an existing software. We would usually want to go live with the minimum viable product(MVP) and build reset of it incrementally. If people are new to agile methodologies, these questions on reducing scope might look stupid & annoying. Fortunately there are ways to get these questions answered, I'm sharing my experience from couple of projects in past few years.

Lets start with a story, where we are writing a new version of the popular website.

<!-- More -->

### Stage 1: Inception / Discovery

<pre>
Team: According to the stats, features X & Y used only by 5%. 
	  Can we deprioritize it for first release?
	  We can redirect to old site for users who need them.
Product owners: No! Everything goes live or nothing does.
</pre>

Start with implementing walking skeleton[[1]](http://alistair.cockburn.us/Walking+skeleton) [[2]](http://blog.codeclimate.com/blog/2014/03/20/kickstart-your-next-project-with-a-walking-skeleton/) to validate the approach. After first few iterations showcase,

### Stage 2: Walking skeleton is ready

<pre>
Team: We have the thin slice of end to end user journey. This is how it works.
Product owners: Wow, thats great. Whats left then?
Team: This one does not handle some of these rare scenarios. Lets prioritize what is needed.
</pre>

Prioritize the backlog to do most important features first. Few weeks before planned release for all features, ask the question again

### Stage 3 : Important features are production ready (MVP)

<pre>
Team: We have everything except feature X & Y. It would take another month for implementing X & Y. 
	  Can we go live with users who need X & Y being redirected to old site?
Product owners: Lets go live!! We need to go there before our competitors
</pre>

## Moral of the stor(y/ies):

* You can't get all the answers in the beginning. Prioritize your backlog to do the most important features first. Ask your unanswered question(s) again after each milestone, you'll be surprised to see how easy it is to get the answers this time.

* It is hard for people to understand the benefits of agile methodologies, kanban etc when it is just theory based on your  past experiences which they can't relate to. Build something small & tangible, show the working software to build their confidence.

