<!--
.. title: A continuous deployment interlude: rolling your own development stack
.. date: 2010/09/29 13:37
.. slug: a-continuous-deployment-interlude-rolling-your-own-development-stack
.. link:
.. description:
.. tags: 
-->

<p>Hi, and welcome to the second part to my ranting about development stacks.&nbsp;In my previous post, I talked about the issues with some of the popular hosted solutions (Github, Bitbucket and Launchpad). In this post, I'll try to roll my own dev stack that hopefully doesn't suck too much.</p>
<p />
<div>Recall from the last post that I'm looking for three things:</div>
<div><ol>
<li>version control</li>
<li>issue tracking</li>
<li>preferably some form of code review</li>
</ol></div>
<div>I think it's a good idea if these things are integrated. Specifically, I think it's a <em>great</em>&nbsp;idea for all of these things to live in the repository. That makes them very easily accessible at every step along the way: your continuous deployment system shouldn't care about deploying software when it has known major bugs in the issue tracker, for example. Code review is integral in resolving tickets, because resolving tickets generally involves changing or adding code (and in both cases, you really should have code review).</div>
<p />
<div>It's fine for this code meta-data to be available in other ways than reading it out of the repository, as long as it satisfies some useful definition of "available". You <em>could</em> just speak HTTP to your issue tracker or code review tool's web page . It just adds another layer that can break -- people who've had to manage these tools can probably imagine the problems when your fancy new CI system reports failed builds, and it's really about Trac being down, or some software upgrade destroying your ability to scrape a web page.</div>
<p />
<div>With that in mind, let's look at the available stuff for rolling your own stack. First, I filtered and ordered tools by the amount I'm willing to put up with them, and came up with:</div>
<div><ol>
<li>Bazaar</li>
<li>Fossil</li>
<li>Mercurial</li>
</ol>(That doesn't mean I think everything else won't work. It means I didn't consider them. I'm only one guy -- barely have enough time to research this, let alone produce an exhaustive list.)</div>
<p />
<div>For Bazaar, well, outside of Launchpad life isn't very interesting and you come up with the usual suspects. For issue tracking and project management you've got stuff like Trac and Redmine. In terms of distributed bug tracking you've got <a href="http://bugseverywhere.org/be/show/HomePage" target="_blank">Bugs Everywhere</a> and <a href="http://ditz.rubyforge.org/" target="_blank">Ditz</a>&nbsp;(I can't figure out which is nicer -- I'm defaulting to BE because it's Python so easier for me to extend -- if anyone has experience with both please chime in). In terms of code review, well, there's Review Board. None of these things are Bzr-specific (which is great, less lock-in is better, except some of them don't have Bazaar support either, which is annoying if it's the tool you're going to be using all day and every day.)</div>
<p />
<div>Next up: <a href="http://www.fossil-scm.org/index.html/doc/tip/www/index.wiki">Fossil</a>. Fossil is pretty obscure; it's an SCM built on top of SQLite, written by the same author. That's a big plus, if you didn't know it yet, SQLite is some of the most extensively tested, well-engineered yet still simple code out there. Fossil works well with my earlier argument about as much metadata as possible being managed inside your repository. It comes with an integrated wiki and issue tracker. The entire repository is stored in a SQLite database. A nice feature that it shares with Trac is the ability to create your own queries over tickets using SQL. Twisted has used that successfully to improve the code review part of their <a href="http://twistedmatrix.com/trac/report">issue system</a>&nbsp;(see {15}, which is review tickets in the order they should be reviewed). Fossil is also very easy to host: in fact, Fossil's website <em>is</em>&nbsp;Fossil serving itself. For. I've had a brief chat with Zed Shaw about Fossil since he's used it quite extensively in real projects such as <a href="http://mongrel2.org/home">Mongrel2</a>&nbsp;(note that that's Fossil serving that webpage again), and he seemed pretty happy with how it works in general. Fossil's easy enough to use for hosting multiple repositories itself: there's a somewhat unfortunate but understandable choice for using CGI, so you can use pretty much anything that can serve CGI to serve Fossil web pages for you.</div>
<p />
<div>Last up: Mercurial. When it comes to code review, I've already explained in a previous post why I think Bitbucket's pull requests are a missed opportunity. (Short version: they make a distinction between local branches and branches in different repositories, which is in my opinion unnecessary. I think the way Launchpad allows merge proposals between arbitrary branches is much better. Github has recently copied this behavior somewhat with Pull Requests 2.0.). However, the main reason I wanted that is for a point to introduce code review.&nbsp;Georg Brandl (birkenfeld) pointed out that Mercurial has the pretty impressive <a href="https://bitbucket.org/sjl/hg-review/src" target="_blank">hg-review extension</a>. It has a sleek, usable&nbsp;<a href="http://review.stevelosh.com/" target="_blank">web interface</a>&nbsp;(try it!). It also has the advantage of the point I made earlier: data is included with your repository, you don't need to go fish for it. Even if you don't use that now, it has the great advantage of being able to do cool things with it later (such as identifying recurring problems in the review process and pointing those out to the reviewer/author).</div>
<p />
<div>Stay tuned for the next installment, where I try something totally crazy.</div>
