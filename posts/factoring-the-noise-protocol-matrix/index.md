<!--
.. title: Factoring the Noise protocol matrix
.. slug: factoring-the-noise-protocol-matrix
.. date: 2018-07-18 10:59
.. tags: security 
.. category: 
.. link: 
.. description: 
.. type: text
-->

TL;DR: if I ever told you to use Noise, I probably meant Noise\_IK and should
have been more specific.

The Noise protocol is one of the best things to happen to encrypted protocol
design. [WireGuard](https://www.wireguard.com) inherits its elegance from Noise.
Noise is a cryptography engineer's darling spec. It's important not to get
blindsided while fawning over it and to pay attention to where implementers run
into trouble. Someone raised a concern I had run into before: Noise has a
matrix.

<table>
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>N(rs):<br>  ← s<br>  ...<br>  → e, es</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NN:<br>  → e<br>  ← e, ee</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KN:<br> → s</span><br><span>  ...<br> → e<br> ← e, ee, se</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XN:<br>  → e<br>  ← e, ee<br>  → s, se</span></p>
                <p><span></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IN:</span><br><span>  → e, s<br>  ← e, ee, se</span><br><span></span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>K(s, rs):<br>  → s<br>  ← s<br>  ...<br>  → e, es, ss</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NK:<br>  ← s<br>  ...<br>  → e, es<br>  ← e, ee</span></p>
                <p><span></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KK:</span><br><span>  → s</span><br><span>  ← s</span><br><span>  …</span><br><span>  → e, es, ss</span><br><span>  ← e, ee, se</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK:<br>  ← s<br>  ...<br>  → e, es<br>  ← e, ee<br>  → s, se</span></p>
                <p><span></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK:</span><br><span>  ← s<br>  ...<br>  → e, es, s, ss<br>  ← e, ee, se</span><br><span></span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>X(s, rs):<br>  ← s<br>  ...<br>  → e, es, s, ss</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NX:<br>  → e<br>  ← e, ee, s, es</span></p>
                <p><span></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KX:<br>  → s<br>  ...<br>  → e<br>  ← e, ee, se, s, es</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX:<br>  → e<br>  ← e, ee, s, es<br>  → s, se</span><br><span></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span> IX:</span><br><span>  → e, s</span><br><span>  ← e, ee, se, s, es</span><br><span></span></p>
            </td>
        </tr>
    </tbody>
</table>

To a cryptography engineer, this matrix is beautiful. These eldritch
runes describe a grammar: the number of ways you can meaningfully
compose the phrases that can make up a Noise handshake into a proper
protocol. The rest of the document describes what the trade-offs between
them are: whether the protocol is one-way or interactive, whether you
get resistance against key-compromise impersonation, what sort of
privacy guarantees you get, et cetera.

(Key-compromise impersonation means that if I steal your key, I can
impersonate anyone to you.)

To the layperson implementer, the matrix is terrifying. They hadn't
thought about key-compromise impersonation or the distinction between
known-key, hidden-key and exposed-key protocols or even forward secrecy.
They're going to fall back to something else: something probably less
secure but at least unambiguous on what to do.

As Noise matures into a repository for protocol templates with wider
requirements, this gets worse, not better. The most recent revision of
the Noise protocol adds 23 new "deferred" variants. It's unlikely these
will be the last additions.

Which Noise variant should they use? Depends on the application of
course, but we can make some reasonable assumptions for most apps.
Ignoring variants, we have:

<table style="text-align: center">
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>N</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IN</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>K</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span>X</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IX</span></p>
            </td>
        </tr>
    </tbody>
</table>

Firstly, let's assume you need bidirectional communication, meaning
initiator and responder can send messages to each other as opposed to
just initiator to responder. That gets rid of the first column of the
matrix.

<table style="text-align: center">
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>N</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XN</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IN</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>K</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>X</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IX</span></p>
            </td>
        </tr>
    </tbody>
</table>

The other protocols are defined by two letters. From the spec:

<hr>
The first character refers to the initiator\'s static key:

* N = No static key for initiator
* K = Static key for initiator Known to responder
* X = Static key for initiator Xmitted (\"transmitted\") to responder
* I = Static key for initiator Immediately transmitted to responder, despite reduced or absent identity hiding

The second character refers to the responder\'s static key:

* N = No static key for responder
* K = Static key for responder Known to initiator
* X = Static key for responder Xmitted (\"transmitted\") to initiator
<hr>

NN provides confidentiality against a passive attacker but neither party
has any idea who you're talking to because no static (long-term) keys
are involved. For most applications none of the \*N suites make a ton of
sense: they imply the initiator does not care who they're connecting to.

<table style="text-align: center">
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>N</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>KN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>XN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>IN</s></span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>K</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>X</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>NX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IX</span></p>
            </td>
        </tr>
    </tbody>
</table>


For most applications the client (initiator) ought to have a fixed
static key so we have a convenient cryptographic identity for clients
over time. So really, if you wanted something with an N in it, you'd
know.

<table style="text-align: center">
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>N</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>KN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>XN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>IN</s></span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>K</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NK</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>X</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NX</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>KX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IX</span></p>
            </td>
        </tr>
    </tbody>
</table>

The responder usually doesn't know what the key is for any initiator that
happens to show up. This mostly makes sense if you have one central initiator
that reaches out to a lot of responders: something like an MDM or sensor data
collection perhaps. In practice, you often end up doing egress from those
devices anyway for reasons that have nothing to do with Noise. So, K\* is out.

<table style="text-align: center">
    <tbody>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>N</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>KN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>XN</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>IN</s></span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>K</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NK</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>KK</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XK</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IK</span></p>
            </td>
        </tr>
        <tr>
            <td colspan="1" rowspan="1">
                <p><span><s>X</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>NX</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span><s>KX</s></span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>XX</span></p>
            </td>
            <td colspan="1" rowspan="1">
                <p><span>IX</span></p>
            </td>
        </tr>
    </tbody>
</table>

These remaining suites generally trade privacy (how easily can you
identify participants) for latency (how many round trips are needed).

IX doesn't provide privacy for the initiator at all, but that's the side you
usually care about. It still has the roundtrip downside, making it a niche
variant.

XX and XK require an extra round trip before they send over the
initiator's static key. Flip side: they have the strongest possible
privacy protection for the initiator, whose identity is only sent to the
responder after they've been authenticated and forward secrecy has been
established.

IK provides a reasonable tradeoff: no extra round trip and the
initiator's key is encrypted to the responder's static key. That means
that the initiator's key is only disclosed if the responder's key is
compromised. You probably don't care about that. It does require the
initiator to know the static key of the responder ahead of time but
that's probably true anyway: you want to check that key against a
trusted value. You can also try private keys for the responder offline
but that doesn't matter unless you gratuitously messed up key
generation. In conclusion, you probably want IK.

This breakdown only works if you're writing a client-server application
that plausibly might've used mTLS instead. WireGuard, for example, is
built on Noise\_IK. The other variants aren't pointless: they're just
good at different things. If you care more about protecting your
initiator's privacy than you do about handshake latency, you want
Noise\_XK. If you're doing a peer-to-peer IoT system where device
privacy matters, you might end up with Noise\_XX. (It's no accident
that IK, XK and XX are in the last set of protocols standing.)

*Protocol variants* Ignore deferred variants for now. If you needed them you'd
know. PSK is an interesting quantum computer hedge. We'll talk more about
quantum key exchanges in a different post, but briefly: a shared PSK among
several participants protects against a passive adversary that records
everything and acquires a quantum computer some time in the future, while
retaining the convenient key distribution of public keys.

*Conclusion* It's incredible how much has happened in the last few years
to make protocols safer, between secure protocol templates like Noise,
new proof systems like Tamarin, and ubiquitous libraries of safer
primitives like libsodium. So far, the right answer for a safe transport
has almost always been TLS, perhaps mutually authenticated. That's not
going to change right away, but if you control both sides of the network
and you need properties hard to get out of TLS, Noise is definitely The
Right Answer. Just don't stare at the eldritch rune matrix too long. You
probably want Noise\_IK. Or, you know, ask your security person :)

Thanks to Katriel Cohn-Gordon for reviewing this blog post.

(This post was syndicated on the Latacora blog.)
