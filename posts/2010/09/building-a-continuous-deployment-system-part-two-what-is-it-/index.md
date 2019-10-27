<!--
.. title: Building a continuous deployment system, part two: what is it?
.. date: 2010/09/26 13:37
.. slug: building-a-continuous-deployment-system-part-two-what-is-it-
.. link:
.. description:
.. tags: 
-->

<p>Hi! This is part two of my continuous deployment <span style="text-decoration: line-through;">rant</span>series which started <a href="http://lvh.posterous.com/building-a-continuous-deployment-system-part">here</a>. In this part,&nbsp;I'll try to explain what I'm talking about when I say continuous deployment.</p>
<p>Strictly speaking, continuous deployment just means that you push code out to production servers all the time, practically as it gets written.</p>
<p>Think about that for a second. Are you scared? If so, why? Don't feel too bad if you are. Most people I've talked to about this are somewhere halfway between intrigued and terrified, and are mostly glad it's not happening to their code. A lot of those people also produce great software anyway. The people that don't feel anxious at all either haven't produced enough production software yet, or are already busy doing continuous deployment.</p>
<p>People who already practice continuous integration, with all sorts of tests and code quality metrics, often talk about how trunk is always ready for production. Continuous deployment is about putting your money where your mouth is and actually doing just that. The basic idea is similar to that of continuous integration: test as many things as you sensibly can and fail as quickly as possible once things go awry. Hopefully, that'll contain minor hiccups before they become real problems (generally defined as "the kind that costs revenue").</p>
<p>However, I don't think the strict definition is a very useful thing to do or talk about purely by itself. I've learned that continuous deployment doesn't come alone. Much more so than continuous integration, which feels like a natural extension of TDD, continuous deployment pretty much turns your development process upside down. Doing continuous deployment without a disciplined team to back it up and a testing infrastructure to constantly check your work is a bit like having a transmission without an engine and a steering wheel. (I apologize for the car analogy, and promise to try and not make a habit out of it.)</p>
<p>From what I've read and my own experience, teams who have successfully applied continuous deployment see these two things come back&nbsp;inseparably. If you want to make sure your code isn't plain broken, you better have a wide array of tests. If you want to make sure the code you're now responsible for in production isn't terrible, you better be doing code review (pair programming and/or tool-assisted code review is great -- we've had success doing both). There's nothing quite like the first time you commit a file and then&nbsp;watching your shiny new piece of code do something on a production server less than a minute later.</p>
<p>So, I guess the series is partially about continuous deployment proper, and partly about greenfielding a development process that rocks.</p>
<p>&nbsp;</p>
