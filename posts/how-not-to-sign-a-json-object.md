<!--
.. title: How (not) to sign a JSON object
.. slug: how-not-to-sign-a-json-object
.. date: 2019-07-24 18:56:06 UTC-07:00
.. tags: security, cryptography
.. category: 
.. link: 
.. description: 
.. type: text
-->

Last year we did a blog post on interservice auth. This post is mostly about authenticating consumers to an API. That’s a related but subtly different problem: you can probably impose more requirements on your internal users than your customers. The idea is the same though: you’re trying to differentiate between a legitimate user and an attacker, usually by getting the legitimate user to prove that they know a credential that the attacker doesn’t.

# You don’t really want a signature

When cryptography engineers say "signature" they tend to mean something asymmetric, like RSA or ECDSA. Developers reach for asymmetric tools too often. There are a lot of ways to screw them up. By comparison, symmetric “signing” (MACs) are easy to use and hard to screw up. HMAC is bulletproof and ubiquitous.

Unless you have a good reason why you need an (asymmetric) signature, you want a MAC. If you really do want a signature, check out our Cryptographic Right Answers post to make that as safe as possible. For the rest of this blog post, "signing" means symmetrically, and in practice that means HMAC.

# How to sign a JSON object

1. Serialize however you want.
2. HMAC. With SHA256? Sure, whatever. We did [a blog post](https://latacora.singles/2018/04/03/cryptographic-right-answers.html) on that too.
3. Concatenate the tag with the message, maybe with a comma in between for easy parsing or something.

# Wait, isn’t that basically a HS256 JWT?

Shut up. Anyway, no, because you need to parse a header to read the JWT, so you inherit all of the problems that stem from that.

# How *not* to sign a JSON object, if you can help it

Someone asked how to sign a JSON object "in-band": where the tag is part of the object you’re signing itself. That's a niche use case, but it happens. You have a JSON object that a bunch of intermediate systems want to read and it’s important  none of them mess with its contents. You can't just send `tag || json`: that may be the cryptographically right answer, but now it's not a JSON object anymore so third party services and middleboxes will barf. You also can't get them to reliably pass the tag around as metadata (via a HTTP header or something). You need to put the key _on the JSON object_, somehow, to "transparently" sign it. Anyone who cares about validating the signature can, and anyone who cares that the JSON object has a particular structure doesn't break (because the blob is still JSON and it still has the data it's supposed to have in all the familiar places).

This problem sort-of reminds me of format-preserving encryption. I don’t mean that in a nice way, because there’s no nice way to mean that. Format-preserving encryption means you encrypt a credit card number and the result still sorta looks like a credit card number. It’s terrible and you only do it because you have to. Same with in-band JSON signing.

As stated, in-band JSON signing means modifying a JSON object (e.g. removing the HMAC tag) and validating that it’s the same thing that was signed. You do that by computing the HMAC again and validating the result. Unfortunately there are infinitely many equal JSON objects with distinct byte-level representations (for some useful definition of equality, like Python’s builtin ==).

Some of those differences are trivial, while others are fiendishly complicated. You can add as many spaces as you want between some parts of the grammar, like after the colon and before the value in an object. You can reorder the keys in an object. You can escape a character using a Unicode escape sequence (\u2603) instead of using the UTF-8 representation. "UTF-8" may be a serialization format for Unicode, but it’s not a canonicalization technique. If a character has multiple diacritics, they might occur in different orders. Some characters can be written as a base character plus a diacritic, but there’s also an equivalent single character. You can’t always know what the “right” character out of context: is this the symbol for the unit of resistance (U+2126 OHM SIGN) or a Greek capital letter Omega (U+03A9)? Don’t even get me started on the different ways you can write the same floating point number!

Three approaches:

1. Canonicalize the JSON.
2. Add the tag and the exact string you signed to the object, validate the signature and then validate that the JSON object is the same as the one you got.
3. Create an alternative format with an easier canonicalization than JSON.

## Canonicalization

Canonicalization means taking an object and producing a unique representation for it. Two objects that mean the same thing ("are equal") but are expressed differently canonicalize to the same representation.

Canonicalization is a quagnet, which is a term of art in vulnerability research meaning quagmire and vulnerability magnet. You can tell it’s bad just by how hard it is to type ‘canonicalization’.

My favorite canonicalization bug in recent memory is probably Kelby Ludwig’s SAML bug. Hold onto your butts, because this bug broke basically every SAML implementation under the sun in a masterful stroke. It used NameIds (SAML-speak for "the entity this assertion is about") that look like this:

    <NameId>barney@latacora.com<!---->.evil.com</NameId>

The common canonicalization strategy ("exc-c14n") will remove comments, so that side sees “barney@latacora.com.evil.com”. The common parsing strategy (“yolo”) disagrees, and sees a text node, a comment, and another text node. Since everyone is expecting a NameId to have one text node, you grab the first one. But that says barney@latacora.com, which isn’t what the IdP signed or your XML-DSIG library validated.

Not to worry: we said we were doing JSON, and JSON is not XML. It’s simpler! Right? There are at least two specs here: Canonical JSON (from OLPC) and an IETF draft (https://tools.ietf.org/id/draft-rundgren-json-canonicalization-scheme-05.html). They work? Probably? But they’re not fun to implement.

## Include the exact thing you’re signing

If you interpret the problem as "to validate a signature I need an exact byte representation of what to sign" and canonicalization is just the default mechanism for getting to an exact byte representation, you could also just attach a specific byte serialization to the object with a tag for it.

You validate the tag matches the specific serialization, and then you validate that the specific serialization matches the outside object with the tag and specific serialization removed. The upside is that you don’t need to worry about canonicalization; the downside is your messages are about twice the size that they need to be. You can maybe make that a little better with compression, since the repeated data is likely to compress well.

## The regex bait and switch trick

If you interpret the problem as being about already having a perfectly fine serialization to compute a tag over, but the JSON parser/serializer roundtrip screwing it up after you compute the tag, you might try to do something to the serialized format that doesn't know it's JSON. This is a variant of the previous approach: you're just not adding a _second_ serialization to compute the tag over.

The clever trick here is to add a field of the appropriate size for your tag with a well-known fake value, then HMAC, then swap the value. For example, if you know the tag is HMAC-SHA256, your tag size is 256 bits aka 32 bytes aka 64 hex chars. You add a unique key (something like `__hmac_tag`) with a value of 64 well-known bytes, e.g. 64 ASCII zero bytes. Serialize the object and compute its HMAC. If you document some subset of JSON serialization (e.g. where CRLFs can occur or where extra spaces can occur), you know that the string `"__hmac_tag": “000...”` will occur in the serialized byte stream. Now, you can use string replacement to shiv in the real HMAC value. Upon receipt, the decoder finds the tag, reads the HMAC value, replaces it with zeroes, computes the expected tag and compares against the previously read value.

Because there’s no JSON roundtripping, the parser can’t mess up the JSON object’s specific serialization. The key needs to be unique because of course the string replacement or regular expression doesn’t know how to parse JSON.

This feels weirdly gross? But at the same time probably less annoying than canonicalization. And it doesn't work if any of the middleboxes modiy the JSON through a parse/re-serialize cycle.

## An alternative format

If you interpret the problem as "canonicalization is hard because JSON is more complex than what I really want to sign", you might think the answer is to reformat the data you want to sign in a format where canonicalization is easy or even automatic.  AWS Signatures do this: there’s a serialization format that’s far less flexible than JSON where you put some key parameters, and then you HMAC that. (There’s an interesting part to it where it also incorporates the hash of the exact message you’re signing -- but we’ll get to that later.)

This is particularly attractive if there’s a fixed set of simple values you have to sign, or more generally if the thing you’re signing has a predictable format.

# Request signing in practice

Let’s apply this model to a case study of request signing has worked through the years in some popular services. These are not examples of how to do it well, but rather cautionary tales.

First off, AWS. AWS requires you to sign API requests. The current spec is "v4", which tells you that there is probably at least one interesting version that preceded it.

## AWS Signing v1

Let’s say an AWS operation CreateWidget takes attribute Name which can be any ASCII string. It also takes an attribute Unsafe, which is false by default and the attacker wishes were true. V1 concatenates the key-value pairs you’re signing, so something like Operation=CreateWidget&Name=iddqd became OperationCreateWidgetNameiddqd. You then signed the resulting string using HMAC.

The problem with this is if I can get you to sign messages for creating widgets with arbitrary names, I can get you to sign operations for arbitrary CreateWidget requests: I just put all the extra keys and values I want in the value you’re signing for me. For example, the request signature for creating a widget named `iddqdUnsafetrue` is exactly the same as a request signature for creating a widget named `iddqd` with Unsafe equal to true: OperationCreateWidgetNameiddqdUnsafetrue.

## AWS Signing V2

Security-wise: fine.

Implementation-wise: it’s limited to query-style requests (query parameters for GET, x-www-form-urlencoded for POST bodies) and didn’t support other methods, let alone non-HTTP requests. Sorting request parameters is a burden for big enough requests. Nothing for chunked requests either.

(Some context: even though most AWS SDKs present you with a uniform interface, there are several different protocol styles in use within AWS. For example, EC2 and S3 are their own thing, some protocols use Query Requests (basically query params in GET queries and POST formencoded bodies), others use REST+JSON, some use REST+XML… There’s even some SOAP! But I think that’s on its way out.)

## AWS Signing V3

AWS doesn’t seem to like V3 very much. The ["what’s new in v4 document"](https://docs.aws.amazon.com/general/latest/gr/sigv4_changes.html) all but disavows it’s existence, and no live services appear to implement it. It had some annoying problems like distinguishing between signed and unsigned headers (leaving the service to figure it out) and devolving to effectively a bearer token when used over TLS (which is great, as long as it actually gets used over TLS).

Given how AWS scrubbed it away, it’s hard to say anything with confidence. I’ve found implementations, but that’s not good enough: an implementation may only use a portion of the spec while the badness can be hiding in the rest.

## AWS Signing V4

Security-wise: fine.

Addressed some problems noted in V2; for example: just signs the raw body bytes and doesn’t care about parameter ordering. This is pretty close to the original recommendation: don’t do inline signing at all, just sign the exact message you’re sending and put a MAC tag on the outside. A traditional objection is that several equivalent requests would have a different representation, e.g. the same arguments but in a different order. It just turns out that in most cases that doesn’t matter, and API auth is one of those cases.

Also note that all of these schemes are really outside signing, but they’re still interesting because they had a lot of the problems you see on an inline signing scheme (they were just mostly unforced errors).

## AWS Signing V0

For completeness. It is even harder to find than V3: you have to spelunk some SDKs for it. I hear it might have been HMAC(k, service || operation || timestamp), so it didn’t really sign much of the request.

## Flickr’s API signing

One commonality of the AWS vulnerabilities is that none of them attacked the primitive. All of them used HMAC and HMAC has always been safe. Flickr had exactly the same bug as AWS V1 signing, but also used a bad MAC. The tag you sent was MD5(secret + your_concatenated_key_value_pairs). We’ll leave the details of extension attacks for a different time, but the punchline is that if you know the value of H(secret + message) and don’t know s, you get to compute H(secret + message + glue + message2), where glue is some binary nonsense and message2 is an arbitrary attacker controlled string.

A typical protocol where this gets exploited looks somewhat like query parameters. The simplest implementation will just loop over every key-value pair and assign the value into an associative array. So if you have user=lvh&role=user, I might be able to extend that to a valid signature for user=lvh&role=userSOMEBINARYGARBAGE&role=admin.

# Conclusion

* Just go ahead and always enforce TLS for your APIs.
* Maybe you don’t need request signing? A bearer token header is fine, or HMAC(k, timestamp) if you’re feeling fancy, or mTLS if you really care.
* Canonicalization is fiendishly difficult.
* Add a signature on the outside of the request body, make sure the request body is complete, and don’t worry about "signing what is said versus what is meant" -- it’s OK to sign the exact byte sequence.
* The corollary here is that it’s way harder to do request signing for a REST API (where stuff like headers and paths and methods matter) than it is to do signing for an RPC-like API.

(This post was syndicated on the Latacora blog.)
