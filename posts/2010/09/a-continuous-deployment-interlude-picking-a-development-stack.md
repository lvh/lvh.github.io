<!--
.. title: A continuous deployment interlude: picking a development stack
.. date: 2010/09/28 13:37
.. slug: a-continuous-deployment-interlude-picking-a-development-stack
.. link:
.. description:
.. tags: 
-->

<div>
<div>At some point in time, you're going to have to pick a set of tools to do your dev work with. You probably want a bunch of things, including:</div>
<div><ol>
<li>A version control system. Preferably not CVS, Visual SourceSafe, ClearCase, Perforce, RCS, SCCS...</li>
<li>A ticketing system. Preferably not Trac.</li>
<li>A code review system, possibly just informal code review as part of the ticketing system.</li>
</ol></div>
<div>Like pretty much everyone, I'm not without bias. I <em>love</em> Bazaar. It's the best version control system I've ever used (and I've used most of the popular ones). I've got a number of reasons why I prefer it over git and hg, but I'm not going to debate VCSes here. All of the version control systems I've just named are all pretty great pieces of software, far superior to what we had to put up with before we had them. That's unfortunate, because then&nbsp;<a href="http://en.wikipedia.org/wiki/Parkinson's_Law_of_Triviality" target="_blank">Parkinson's law of triviality</a>&nbsp;comes into effect, and people who really don't even care very much either way will feel the need to defend their favorite piece of software to the death. I think we should come to terms with VCS wars being the new editor wars. (The inverse relation between clue and volume is an unfortunate perennial in these things.)</div>
<p />
<div>At my old place of work, we found a way around that. We had an SVN trunk, and no human was allowed to commit to it. There's a bot takes patches which commits to trunk -- and patches are only sent after two (later three) people sign off on the code review. One obvious advantage is that you completely avoid religious wars. I don't have to convince people to switch to $AWESOME_VCS. There's also the problem of authorship erasure: because the bot commits everything and this was a very simplistic bespoke system, tools like <span style="font-family: courier new, monospace;">svn blame</span> stop working. At first sight, this seems pretty bad. I'm not entirely convinced: it's not great, but on the other hand, it is a sort-of neat way of enforcing collective code ownership. Overall, I think this was a nice experiment, but it's about time we stop mucking around and build some better tools.</div>
<p />
<div>I started by looking all of the hosted services and the DVCS that powers them:</div>
<div><ol>
<li>Launchpad with Bazaar.</li>
<li>Bitbucket with Mercurial</li>
<li>Github with git (and <a href="http://github.com/defunkt/hub">hub</a>)</li>
</ol></div>
<div>Because of my affinity for Bazaar, Launchpad obviously came first.&nbsp;Alas, there's a problem with Launchpad for small commercial projects. Pricing.</div>
<p />
<div>[[posterous-content:shOrCG05NvDQO8UBylbu]]</div>
<div>Here's how it works. Github (in dark red) has a <a href="http://github.com/plans" target="_blank">bunch of plans for private repositories</a>. Bitbucket (in light blue) has a <a href="http://bitbucket.org/plans" target="_blank">bunch of plans for private repositories</a>. Launchpad (in yellow) has a <a href="https://launchpad.net/+tour/join-launchpad" target="_blank">one size fits all $250/yr/project plan</a>.</div>
<p />
<div>I am in no way saying $250 is an unsurmountable amount of money for a company. I am saying that at the low end of the spectrum, you get a whole lot more bang for your buck at Github and especially Bitbucket than you get at Launchpad. The services they provide are not equivalent: merge proposals are quite a bit better than pull requests in both Github and Bitbucket (although Github's Pull Request 2.0's are halfway there, their pull-request-is-an-issue philosophy makes it impossible to use pull requests as proposed solutions to tickets), and the issue tracker feels a lot more polished than both Github's and Bitbucket's (although between those two, Bitbucket probably wins that round). I'm not saying it's not better. I'm saying I'm having a hard time justifying that it's around $200/year/project better. It's not that Launchpad's expensive as much as the&nbsp;competition&nbsp;is really, really cheap.</div>
<p />
<div>Let's say I'm a tiny bootstrapping startup with somewhere between $10k and $30k to spend. If you've got one project, $250/yr that's quite all right. However, building your big app as a set of smaller ones has a number of advantages, in terms of administration and deployment. Once I've got 5+ tiny projects (which, put together, make up the code in my startup), it's about $1250/yr. Still not insurmountable, but I've got more interesting things to spend $1k+ on than Launchpad. $1k/year for hosting is not a sensible way to spend your money when most of those projects only get a few tickets a week.</div>
<p />
<div>So, just to be clear: I'm not saying Launchpad sucks. I think Launchpad is great. I'm not saying Launchpad's pricing scheme is asinine. I'm saying their pricing scheme works out very badly for what I want to do, and that I'm having a hard time convincing other people the extra cost is worth the difference in functionality.</div>
<p />
</div>
<div>In the next post, I'll talk about rolling your own development stack.</div>
