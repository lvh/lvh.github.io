<!--
.. title: Designing a REST API: logging in
.. date: 2011/11/27 13:37
.. slug: designing-a-rest-api-logging-in
.. link:
.. description:
.. tags: 
-->

<div>Hey. As many of you probably know, I'm building a business, and that business involves a REST API being consumed by browsers (for now).</div>
<p />
<div>The problem I'm hitting is registration and logging in. It just doesn't seem to be a problem that fits REST very well -- maybe I'm thinking too much about services and not enough about resources, so I'm writing this blog posts to get my thoughts in order and hopefully get some useful feedback.</div>
<p />
<div>This post is about user registration and login, and how they're hard to fit into REST.</div>
<p />
<div>Context: users are uniquely identified by a user id, which is a random string of sufficient size. These ids are immutable and uniquely refer to this user from registration until forever. User emails are also unique, but they can change, so they are not a good way to refer to a user. Users register and log in using email and password.</div>
<p />
<div><strong><span style="font-size: medium;">Option 1: separate authentication endpoint</span></strong></div>
<p />
<div>The idea here is that there are two things:</div>
<div>
<ul>
<li>the REST API, which authenticated things talk to</li>
<li>the authentication endpoint, where unauthenticated things go to become authenticated</li>
</ul>
The downside is that there are two things. One of them speaks REST, and the other speaks gnarly ad-hoc RPC or maybe JSON-RPC or something.</div>
<p />
<div>The upside is that the REST API only needs to understand one kind of token.</div>
<p />
<div><span style="font-size: medium;"><strong>Option 2: everything in REST</strong></span></div>
<p />
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">Special cases aren't special enough to break the rules.</blockquote>
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">-- Zen of Python</blockquote>
<p />
<div>The advantage is that there is one canonical source for everything. Everything including access grants (which should be nice for OAuth later). You can use the REST API to perform any action, see anything in the system...</div>
<p />
<div>The downsides:</div>
<div>
<ul>
<li>the REST API has to understand more than one kind of authentication. This also might make it a bit uglier to interoperate with OAuth later.</li>
<li>Logging in takes too many round trips. First I have to query <span style="font-family: courier new,monospace;">users/?email={EMAIL}</span><span style="font-family: arial,helvetica,sans-serif;">&nbsp;to go from e-mail address to user id. Then, I have to POST to users/accessGrants -- which is if I'm allowed to guess the URL, which I really shouldn't, I should GET users/ first (HATEOAS and all that -- but I'm willing to ignore that for now). All of this happens over HTTPS with HTTP Basic auth. TLS handshakes are slow, and verifying securely stored passwords is even slower.</span></li>
</ul>
</div>
<p />
<div><strong><span style="font-size: medium;">Option 2b: everything in REST, with shortcuts</span></strong></div>
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">Although practicality beats purity.</blockquote>
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">-- Zen of Python&nbsp;</blockquote>
<div>... but, unfortunately, also:</div>
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">
<div>There should be one-- and preferably only one --obvious way to do it.</div>
</blockquote>
<blockquote class="gmail_quote" style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; margin-left: 0.8ex; border-left-width: 1px; border-left-color: #cccccc; border-left-style: solid; padding-left: 1ex;">-- Zen of Python&nbsp;</blockquote>
<p />
<div>The upsides are the same as for the REST API, except we get rid of the downside that logging in takes too many round trips, because there's a shortcut for that.</div>
<p />
<div>That shortcut would be something that speaks some kind of ad-hoccy RPC, or possibly JSON-RPC. One of the exported methods/procedures would be "getGrant", which takes a username and password and returns the key. These endpoints talk to the same database behind the scenes. Killer feature: one round trip.</div>
<p />
<div>Downside is that the API should probably still be able to take user credentials, since this is is a shortcut....</div>
<p />
<div>Yes, I know OAuth exists. For now, we have no interest in opening this API up yet. We'll do that as soon as we have a platform worth caring about, an API that doesn't change every three seconds, and a security expert that's reviewed it.</div>
