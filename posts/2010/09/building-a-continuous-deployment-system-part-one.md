<!--
.. title: Building a continuous deployment system, part one
.. date: 2010/09/26 13:37
.. slug: building-a-continuous-deployment-system-part-one
.. link:
.. description:
.. tags: 
-->

<p>Lately, I've been thinking a lot about building a continuous deployment system.</p>
<p>I was really glad to find out that Timothy Fitz was doing <a href="http://timothyfitz.wordpress.com/2009/02/10/continuous-deployment-at-imvu-doing-the-impossible-fifty-times-a-day/">continuous deployment over at IMVU</a>&nbsp;(partially because Timothy and I like some of the same software). Eric Ries, former CTO and co-founder of IMVU, has had a lot of <a href="http://www.startuplessonslearned.com/search/label/continuous%20deployment">interesting things</a> to say about it as well. I used to do this at my old day job, and it's been an absolute eye-opener -- yes, it does require quite a bit of discipline, but I'm convinced it pays off handsomely, especially in the long run.</p>
<p>Despite my newfound conviction, the system we used there wasn't all that fancy. That has its upsides, the less complex it is, the less can go wrong. Unfortunately, being a horrendous hodgepodge (I'd call it an unholy alliance, but there's no VB or COBOL) of bash, perl and Python distributing software with rsync, it's not very suitable for showing off.</p>
<p>The goal of this series of blog posts is to end up with a continuous deployment system people can agree on. (If you're laughing, yes, I realize that's probably a little naive -- I'm hoping to come out relatively unscathed and with a continuous deployment system I can agree on.)</p>
<p>&nbsp;</p>
