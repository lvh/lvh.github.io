<!--
.. title: A child's garden of inter-service authentication schemes
.. slug: a-childs-garden-of-inter-service-authentication-schemes
.. date: 2018-06-12 15:27
.. tags:
.. category: security, cryptography
.. link:
.. description:
.. type: text
-->

Modern applications tend to be composed from relationships between smaller applications. Secure modern applications thus need a way to express and enforce security policies that span multiple services. This is the “server-to-server” (S2S) authentication and authorization problem (for simplicity, I’ll mash both concepts into the term “auth” for most of this post).

Designers today have a lot of options for S2S auth, but there isn’t much clarity about what the options are or why you’d select any of them. Bad decisions sometimes result. What follows is a stab at clearing the question up.

## Cast Of Characters
Alice and Bob are services on a production VPC. Alice wants to make a request of Bob. How can we design a system that allows this to happen?

Here’s, I think, a pretty comprehensive overview of available S2S schemes. I’ve done my best to describe the “what’s” and minimize the “why’s”, beyond just explaining the motivation for each scheme. Importantly, these are all things that reasonable teams use for S2S auth.

### Nothing At All
Far and away the most popular S2S scheme is “no auth at all”. Internet users can’t reach internal services. There’s little perceived need to protect a service whose only clients are already trusted.

### Bearer Token
Bearer tokens rule everything around us. Give Alice a small blob of data, such that when Bob sees that data presented, he assumes he’s talking to Alice. Cookies are bearer tokens. Most API keys are bearer tokens. OAuth is an elaborate scheme for generating and relaying bearer tokens. SAML assertions are delivered in bearer tokens.

The canonical bearer token is a random string, generated from a secure RNG, that is at least 16 bytes long (that is: we generally consider 128 bits a reasonable common security denominator). But part of the point of a bearer token is that the holder doesn’t care what it is, so Alice’s bearer token could also encode data that Bob could recover. This is common in client-server designs and less common in S2S designs.

#### A few words about passwords
S2S passwords are disappointingly common. You see them in a lot of over-the-Internet APIs (ie, for S2S relationships that span companies). A password is basically a bearer token that you can memorize and quickly type. Computers are, in 2018, actually pretty good at memorizing and typing, and so you should use real secrets, rather than passwords, in S2S applications.

### HMAC(timestamp)
The problem with bearer tokens is that anybody who has them can use them. And they’re routinely transmitted. They could get captured off the wire, or logged by a proxy. This keeps smart ops people up at night, and motivates a lot of “innovation”.

You can keep the simplicity of bearer tokens while avoiding the capture-in-flight problem by exchanging the tokens with secrets, and using the secrets to authenticate a timestamp. A valid HMAC proves ownership of the shared secret without revealing it. You’d then proceed as with bearer tokens.

#### A few words about TOTP
TOTP is basically HMAC(timestamp) stripped down to make it easy for humans to briefly memorize and type. As with passwords, you shouldn’t see TOTP in S2S applications.

#### A few words about PAKEs
PAKEs are a sort of inexplicably popular cryptographic construction for securely proving knowledge of a password and, from that proof, deriving an ephemeral shared secret. SRP is a PAKE. People go out of their way to find applications for PAKEs. The thing to understand about them is that they’re fundamentally a way to extract cryptographic strength from passwords. Since this isn’t a problem computers have, PAKEs don’t make sense for S2S auth.

### Encrypted Tokens
HMAC(timestamp) is stateful; it works because there’s pairwise knowledge of secrets and the metadata associated with them.  Usually, this is fine. But sometimes it’s hard to get all the parties to share metadata.

Instead of making that metadata implicit to the protocol, you can store it directly in the credential: include it alongside the timestamp and HMAC or encrypt it. This is how Rails cookie storage works; it’s also the dominant use case for JWTs. AWS-style request “signing” is another example (using HMAC and forgoing encryption).

By themselves, encrypted tokens make more sense in client-server settings than they do for S2S. Unlike client-server, where a server can just use the same secret for all the clients, S2S tokens still require some kind of pairwise state-keeping.

### Macaroons
You can’t easily design a system where Alice takes her encrypted token, reduces its security scope (for instance, from read-write to read-only), and then passes it to Dave to use on her behalf. No matter how “sophisticated” we make the encoding and transmission mechanisms, encrypted tokens still basically express bearer logic.

Macaroons are an interesting (and criminally underused) construction that directly provides both _delegation_ and _attenuation_. They’re a kind of token from which you can derive more restricted tokens (that’s the “attenuation”), and, if you want, pass that token to someone else to use without them being able to exceed the authorization you gave them. Macaroons accomplish this by chaining HMAC; the HMAC of a macaroon is the HMAC secret for its derived attenuated macaroons.

By adding encryption along with HMAC, Macaroons also express “third-party” conditions. Alice can get Charles to attest that Alice is a member of the super-awesome-best-friends-club, and include that in the Macaroon she delivers to Bob. If Bob also trusts Charles, Bob can safely learn whether Alice is in the club. Macaroons can flexibly express whole trees of these kinds of relationships, capturing identity, revocation, and… actually, revocation and identity are the only two big wins I can think of for this feature.

### Asymmetric Tokens
You can swap the symmetric constructions used in tokens for asymmetric tokens and get some additional properties.

Using signatures instead of HMACs, you get non-repudiability: Bob can verify Alice’s token, but can’t necessarily mint a new Alice token himself.

More importantly, you can eliminate pairwise configuration. Bob and Alice can trust Charles, who doesn’t even need to be online all the time, and from that trust derive mutual authentication.

The trade-offs for these capabilities are speed and complexity. Asymmetric cryptography is much slower and much more error-prone than symmetric cryptography.

### Mutual TLS
Rather than designing a new asymmetric token format, every service can have a certificate. When Alice connects to Bob, Bob can check a whitelist of valid certificate fingerprints, and whether Alice’s name on her client certificate is allowed. Or, you could set up a simple CA, and Bob could trust any certificate signed by the CA. Things can get more complex; you might take advantage of X.509 and directly encode claims in certs (beyond just names).

#### A few words about SPIFFE
If you’re a Kubernetes person this scheme is also sometimes called SPIFFE.

#### A few words about Tokbind
If you’re a participant in the IETF TLS Working Group, you can combine bearer tokens and MTLS using tokbind. Think of tokbind as a sort of “TLS cookie”. It’s derived from the client and server certificate and survives multiple TLS connections. You can use a tokbind secret to sign a bearer token, resulting in a bearer token that is confined to a particular MTLS relationship that can’t be used in any other context.

### Magic Headers
Instead of building an explicit application-layer S2S scheme, you can punt the problem to your infrastructure. Ensure all requests are routed through one or more trusted, stateful proxies. Have the proxies set headers on the forwarded requests. Have the services trust the headers.

This accomplishes the same things a complicated Mutual TLS scheme does without requiring slow, error-prone public-key encryption. The trade-off is that your policy is directly coupled to your network infrastructure.

### Kerberos
You can try to get the benefits of magic headers and encrypted tokens at the same time using something like Kerberos, where there’s a magic server trusted by all parties, but bound by cryptography rather than network configuration. Services need to be introduced to the Kerberos server, but not to each other; mutual trust of the Kerberos server, and authorization logic that lives on that Kerberos server, resolves all auth questions. Notably, no asymmetric cryptography is needed to make this work.

## Themes
What are the things we might want to achieve from an S2S scheme? Here’s a list. It’s incomplete. Understand that it’s probably not reasonable to expect _all of these things_ from a single scheme.

### Minimalism
This goal is less obvious than it seems. People adopt complicated auth schemes without clear rationales. It’s easy to lose security by doing this; every feature you add to an application  especially security features  adds attack surface. From an application security perspective, “do the simplest thing you can get away with” has a lot of merit. If you understand and keep careful track of your threat model, “nothing at all” can be a security-maximizing option. Certainly, minimalism motivates a lot of bearer token deployments.

The opposite of minimalism is complexity. A reasonable way to think about the tradeoffs in S2S design is to think of complexity as a currency you have to spend. If you introduce new complexity, what are you getting for it?

### Claims
Authentication and authorization are two different things: who are you, and what are you allowed to do? Of the two problems, authorization is the harder one. An auth scheme can handle authorization, or assist authorization, or punt on it altogether.

Opaque bearer token schemes usually just convey identity. An encrypted token, on the other hand, might bind _claims_: statements that limit the scope of what the token enables, or metadata about the identity of the requestor.

Schemes that don’t bind claims can make sense if authorization logic between services is straightforward, or if there’s already a trusted system (for instance, a service discovery layer) that expresses authorization. Schemes that do bind claims can be problematic if the claims carried in an credential can be abused, or targeted by application flaws. On the other hand, an S2S scheme that supports claims can do useful things like propagating on-behalf-of requestor identities or supporting distributed tracing.

### Confinement
The big problem with HTTP cookies is that once they’ve captured one, an attacker can abuse it however they see fit. You can do better than that by adding mitigations or caveats to credentials. They might be valid only for a short period of time, or valid only for a specific IP address (especially powerful when combined with short expiry),  or, as in the case of Tokbind, valid only on a particular MTLS relationship.

### Statelessness
Statelessness means Bob doesn’t have to remember much (or, ideally, anything) about Alice. This is an immensely popular motivator for some S2S schemes. It’s perceived as eliminating a potential performance bottleneck, and as simplifying deployment.

The tricky thing about statelessness is that it often doesn’t make sense to minimize state, only to eliminate it. If pairwise statefulness creeps back into the application for some other reason (for instance, Bob has to remember anything at all about Alice), stateless S2S auth can spend a lot of complexity for no real gain.

### Pairwise Configuration
Pairwise configuration is the bête noire of S2S operational requirements. An application secret that has to be generated once for each of several peers and that _anybody might ever store in code_ is part of a scheme in which secrets are never, ever rotated. In a relatively common set of circumstances, pairwise config means that new services can only be introduced during maintenance windows.

Still, if you have a relatively small and stable set of services (or if all instances of a particular service might simply share a credential), it can make sense to move complexity out of the application design and into the operational requirements.  Also it makes sense if you have an ops team and you never have to drink with them.

I kid, really, because if you can get away with it, not spending complexity to eliminate pairwise configuration can make sense. Also, many of the ways S2S schemes manage to eliminate pairwise configurations involve _introducing yet another service_, which has a sort of constant factor cost that can swamp the variable cost.

### Delegation and Attenuation
People deploy a lot of pointless delegation. Application providers might use OAuth for their client-server login, for instance, even though no third-party applications exist.  The flip side of this is that if you actually need delegation, you really want to have it expressed carefully in your protocol. The thing you don’t want to do is ever share a bearer token.

Delegation can show up in internal S2S designs as a building block. For instance, a Macaroon design might have a central identity issuance server that grants all-powerful tokens to systems that in turn filter them for specific requestors.

Some delegation schemes have implied or out-of-band attenuation. For instance, you might not be able to look at an OAuth token and know what it’s restrictions are. These systems are rough in practice; from an operational security perspective, your starting point probably needs to be that any lost token is game-over for its owner.

A problem with writing about attenuation is that Macaroons express it so well that it’s hard to write about its value without lapsing into the case for Macaroons.

### Flexibility
If use JSON as your credential format, and you later build a feature that allows a credential to express not just Alice’s name but also whether she’s an admin, you can add that feature without changing the credential format. Later, attackers can add the feature where they turn any user into an admin, and you can then add the feature that breaks that attack. JSON is just features all the way down.

I’m only mostly serious. If you’re doing something more complicated than a bearer token, you’re going to choose an extensible mechanism. If not, I already made the case for minimalism.

### Coupling
All things being equal, coupling is bad. If your S2S scheme is expressed by network controls and unprotected headers, it’s tightly coupled to the network deployment, which can’t change without updating the security scheme. But if your network configuration doesn’t change often, that limitation might save you a lot of complexity.

### Revocation
People talk about this problem a lot. Stateless schemes have revocation problems: the whole point of a stateless scheme is for Bob not to have to remember anything about Alice (other than perhaps some configuration that says Alice is allowed to make requests, but not Dave, and this gets complicated really quickly and can quickly call into question the value of statelessness but let’s not go there). At any rate: a stateless bearer token will eventually be compromised, and you can’t just let it get used over and over again to steal data.

The two mainstream answers to this problem are short expiry and revocation lists.

Short expiry addresses revocation if: (a) you have a dedicated auth server and the channel to that server is somehow more secure than the channel between Alice and Bob.; (b) the auth server relies on a long-lived secret that never appears on the less-secure channel, and (c) issues an access secret that is transmitted on the less-secure channel, but lives only for a few minutes. These schemes are called “refresh tokens”. Refresh tends to find its way into a lot of designs where this fact pattern doesn’t hold. Security design is full of wooden headphones and coconut phones.

Revocation lists (and, usually, some attendant revocation service) are a sort of all-purpose solution to this problem; you just blacklist revoked tokens, for at least as long as the lifetime of the token. This obviously introduces state, but it’s a specific kind of state that doesn’t (you hope) grow as quickly as your service does. If it’s the only state you have to keep, it’s nice to have the flexibility of putting it wherever you want.

### Rigidity
It is hard to screw up a random bearer token. Alice stores the token and supply it on requests. Bob uses the token to look up an entry in a database. There aren’t a lot of questions.

It is extraordinarily easy to screw up JWT. JWT is a JSON format where you have to parse and interpret a JSON document to figure out how to decrypt and authenticate a JSON document. It has revived bugs we thought long dead, like “repurposing asymmetric public keys as symmetric private keys”.

Problems with rigidity creep up a lot in distributed security. The first draft of this post said that MTLS was rigid; you’re either speaking TLS with a client cert or you’re not. But that ignores how hard X.509 validation is. If you’re not careful, an attacker can just ask Comodo for a free email certificate and use it to access your services. Worse still, MTLS can “fail open” in a way that TLS sort of doesn’t: if a service forgets to check for client certificates, TLS will still get negotiated, and you might not notice until an attacker does.

Long story short: bearer tokens are rigid. JWT is a kind of evil pudding. Don’t use JWT.

### Universality
A nice attribute of widely deployed MTLS is that it can mitigate SSRF bugs (the very bad bug where an attacker coerces one of your service to make an arbitrary HTTP request, probably targeting your internal services, on their behalf). If the normal HTTP-request-generating code doesn’t add a client certificate, and every internal service needs to see one to honor a request, you’ve limited the SSRF attackers options a lot.

On the other hand, we forget that a lot of our internal services consist of code that we didn’t write. The best example of this is Redis, which for years proudly waved the banner of “if you can talk to it, you already own the whole application”.

It’s helpful if we can reasonably expect an auth control to span all the systems we use, from Postgres to our custom revocation server. That might be a realistic goal with Kerberos, or with network controls and magic headers; with tunnels or proxies, it’s even something you can do with MTLS  this is a reason MTLS is such a big deal for Kubernetes, where it’s reasonable for the infrastructure to provide every container with an MTLS-enabled Envoy proxy.  On the other hand it’s unlikely to be something you can achieve with Macaroons or evil puddings.

### Performance and Complexity
If you want performance and simplicity, you probably avoid asymmetric crypto, unless your request frequency is (and will remain) quite low. Similarly, you’d probably want to avoid dedicated auth servers, especially if Bob needs to be in constant contact with them for Alice to make requests to him; this is a reason people tend to migrate away from Kerberos.

## Our Thoughts
Do the simplest thing that makes sense for your application right now. A true fact we can relate from something like a decade of consulting work on these problems: intricate S2S auth schemes are not the norm; if there’s a norm, it’s “nothing at all except for ELBs”.  If you need something, but you have to ask whether that something oughtn’t just be bearer tokens, then just use bearer tokens.

Unfortunately, if there’s a second norm, it’s adopting complicated auth mechanisms independently or, worse, in combination, and then succumbing to vulnerabilities.

Macaroons are inexplicably underused. They’re the Velvet Underground of authentication mechanisms, hugely influential but with little radio airplay. Unlike the Velvets, Macaroons aren’t overrated. They work well for client-server auth and for s2s auth. They’re very flexible but have reassuring format rigidity, and they elegantly take advantage of just a couple simple crypto operations. There are libraries for all the mainstream languages. You will have a hard time coming up with a scenario where we’d try to talk you out of using them.

JWT is a standard that tries to do too much and ends up doing everything haphazardly.  Our loathing of JWT motivated this post, but this post isn’t about JWT; we’ll write more about it in the future.

If your inter-service auth problem really decomposes to inter-container (or, without containers, inter-instance) auth, MTLS starts to make sense. The container-container MTLS story usually involves containers including a proxy, like Envoy, that mediates access. If you’re not connecting containers, or have ad-hoc components, MTLS can really start to take on a CORBA feel: random sidecar processes (here stunnel, there Envoy, and this one app that tries to do everything itself). It can be a pain to configure properly, and this is a place you need to get configurations right.

If you can do MTLS in such a way that there is exactly one way all your applications use it (probably: a single proxy that all your applications install), consider MTLS. Otherwise, be cautious about it.

Beyond that, we don’t want to be too much more prescriptive. Rather, we’d just urge you to think about what you’re actually getting from an S2S auth scheme before adopting it.

(But really, you should just use Macaroons.)

(This post was syndicated on the Latacora blog.)
