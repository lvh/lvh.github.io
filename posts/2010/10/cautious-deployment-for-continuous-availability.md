<!--
.. title: Cautious deployment for continuous availability
.. date: 2010/10/04 13:37
.. slug: cautious-deployment-for-continuous-availability
.. link:
.. description:
.. tags: 
-->


<div>In <a href="http://lvh.posterous.com/designing-a-continuous-deployment-system-caut">my last post</a>, I talked about cautious deployment as a technique for recovering from botched deployments. Cautious deployment&nbsp;also solves a second problem by helping you get continuous availability.</div>
<p />
<div>In the article about cautious deployment, I had the following graph for hypothetical, optimistic deployment:</div>
<div>[[posterous-content:L2vALkn09tFOvqjIkPGw]]</div>
<p />
<div>Obviously, that's a bit simplistic. It would be correct, if all your servers decide to restart the service in a perfectly synchronized fashion, and all of your servers manage do that operation in precisely identical amounts of time, and no interdependencies between your services.</div>
<p />
<div>Typically: none of those things are true. starting services is a stochastic process and you really don't quite know how long it'll take. Stuff depends on each other all the time. Hence, the graph is more likely to look something like this:</div>
<p />
<div>[[posterous-content:67uOLIMpCLyTivUYZ5MS]]</div>
<div>Of course, it's occasionally way worse than this. Your users generally don't know nor care that you're upgrading, so those few early starters get to deal with a disproportionately high load. That could happen up to the point where watchdogs mistakenly believe the process has just crashed. That watchdog then orders for that process to be restarted, which of course only makes it worse for all of the other processes still up.</div>
<p />
<div>(If you're thinking "Oh come on. It's nowhere near that bad" -- how do you know? Did you measure that? For some people, it definitely is.)</div>
<p />
<div>This problem is often overlooked, because there's tons of ways of missing it completely (mostly because&nbsp;statistics is hard).</div>
<div><ol>
<li>Not even bothering to analyze (unfortunately the most common)</li>
<li>Not recognizing the worst case</li>
<li>Normal distribution not being a good fit</li>
<li>Failure to recognize the weakest link</li>
</ol></div>
<div>Unfortunately, most of the people fall in group one. Come on guys, we can all do better than this. If there's two recurring patterns in modern (I hate to use the word, but...) agile businesses, that comes back across the board it's "measure" and "iterate". Even rudimentary tests are better than flying blind.</div>
<p />
<div>Number two, failure to recognize the worst case, generally happens when people do some kind of basic analysis. That generally amounts to taking a bunch of values and then computing the mean/median and working from there. Not recognizing the worst case is really a special case of not really assessing the distribution of your values correctly.</div>
<p />
<div>A mean is pretty interesting by itself, but you can't get too much useful information out of it completely by itself. Knowing a service starts within a second on average is pretty useless when it has a standard deviation of a minute. (If that's the case, descriptive statistics just showed you that your data probably doesn't fit a normal distribution very well: the lower limit for physically possible events is at Z=-1<span style="font-family: sans-serif; font-size: 13px; line-height: 19px;">/60, unless your servers start up in negative amounts of time...</span>)</div>
<p />
<div>First of all, that service might have things that are waiting on it to get started. This is the weakest link argument: your slowest step just became a defining factor for the performance of the entire process.&nbsp;Secondly, people perceive jittery things worse than consistent things, all other things being equal. Imagine if Google loaded a lot faster often (would you even really notice?) but about as often as it would load faster, it would take five seconds.</div>
<p />
<div>Even when you have people who do some pretty reasonable statistical analysis on individual parts, that becomes only half the story in a distributed system. Even with measurements from real deployments, it's extremely hard to predict how these things will behave.</div>
<p />
<div>Bottom line? Statistics is hard, and it's easy to get fooled into thinking you've got it right even when you're miles off (I've been bitten by this myself, more than once). Once you've done that, set up a sandboxed miniature model for your entire (distributed) system, and check the assumptions from your statistics against it. After that, you're only likely to miss scale issues.&nbsp;</div>
<p />
<div>Even when you take all of that into account, you will get things wrong. That was my original point about cautious deployment: as long as you have that and it works, you're allowed to get it wrong once in a while. Fail gracefully. It's not so scary to leap the chasm when there's a net to catch you.</div>
