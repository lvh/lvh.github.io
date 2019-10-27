<!--
.. title: Building a continuous deployment system, part three, preparing for continuous deployment
.. date: 2010/09/27 13:37
.. slug: building-a-continuous-deployment-system-part-three-preparing-for-continuous-deployment
.. link:
.. description:
.. tags: 
-->

<p>This post is just about the process of moving a dev shop to continuous deployment. If you're just interested in the things I'd like to build, feel free to skip this one.</p>
<p>Despite the ominous messages in my last post about continuous deployment changing your dev shop dramatically, I do think it's possible to prepare properly. That means that you get up to the point where you <em>could</em> be doing continuous deployment, but you just aren't. Or, to put it differently, imagining you had to release right now, and getting trunk up to the point where that doesn't raise your blood pressure.</p>
<p>The first step is getting your testing up to an adequate level. I can't stress this enough -- I'm hoping it won't be a tough sale considering the amount of people who've hammered it before me -- given today's tools, there is very little excuse left for not having an extensive suite of things that check your code. I'm deliberately not saying "unit tests", because it encompasses a lot more than just unit tests:</p>
<ol>
<li>unit tests (These still go up as number one because they're what you're dealing with most. Plus, out of the things in your code that are just plain broken, unit tests will help you find at least 90%.)</li>
<li>static analysis (<a href="http://divmod.org/trac/wiki/DivmodPyflakes">pyflakes</a> is great, but Python is quite hard to do sensible static analysis on -- statically typed languages, for example, can expect to reap much greater benefits)</li>
<li>integration tests (although these are generally part of my normal test suite and are ran together with unit tests, they're really quite different)</li>
<li>user interface tests (if you're building a webapp, <a href="http://seleniumhq.org/">Selenium</a> is wonderful)</li>
</ol>
<p>Testing is very important, but it's hardly the whole story. Testing (with the minor exception of some forms of static analysis) make sure your code <em>works</em>&nbsp;-- it makes no guarantees about that code being any good. For now, computers aren't too great at judging code in such as subtle fashion, and you really want humans to do it for you. Especially in languages that let you do pretty much anything, code review is very important if you want to keep your code <em>sane</em>. Like test-driven development, a lot of people much smarter than me have driven this one home for a long time, so I won't try to convince you too much.</p>
<p>Another advantage of code review is collective code ownership and a resulting increase in <a href="http://c2.com/cgi/wiki$?BusNumber">bus number</a>. High bus numbers are very important, especially so when you're doing continuous deployment. You can't, or at least <em>shouldn't</em>, delay a release. Once something does go wrong, fixing the situation should be everyone's immediate priority. Only one person understanding how a particular part of the project works slows things down. That sounds like a restriction, and in some ways it is, but it's the kind of restriction that forces you to do the right thing.</p>
<p>I have found that a lot of people do one kind of code review, and contrast them. I've had good experiences with doing extensive and multiple kinds of code review. Most people believe that to be a waste of time and resources. I'm not convinced. First of all, there's Linus' Law arguing for many reviewers. Secondly, I've found that different kinds of review (and different kinds of reviewers) spot different problems, because they care about different things:</p>
<ol>
<li>A pair programmer will spot nitpicks about code quality and save you a bunch of "duh" moments</li>
<li>A reviewer working on the same project reviewing a feature branch will pay more attention to&nbsp;interoperability&nbsp;issues with the rest of the code</li>
<li>Someone who's not even a programmer (yep, "everybody writes code" works out great) will spot documentation issues</li>
<li>... (this is not, by any means, an exhaustive list)</li>
</ol>
<p>Adding a form of code review takes a load off for all of the other forms of code review. The curve is sublinear: adding more code review doesn't make code review take proportionally longer (it won't make it shorter either, of course).</p>
<p>A lot of people dislike being criticized about their code. <strong>Stamp this behavior out as soon as possible.</strong> Everyone's only human, and plenty of ugly, broken or otherwise inadequate code gets written. The point is to get it out, not berating the person that wrote it.</p>
<p>Pressing the big red button once you're done will still be scary. You might even have a hiccup or two. But ideally, nothing will blow up. After a while (and probably in less time than you think it will), releases will become, as Timothy Fitz puts it, a non-event.</p>
