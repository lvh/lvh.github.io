<!--
.. title: The default OpenSSH key encryption is worse than plaintext
.. slug: the-default-openssh-key-encryption-is-worse-than-plaintext
.. date: 2018-08-03 19:00:18 UTC-07:00
.. tags: security, cryptography 
.. category: 
.. link: 
.. description: 
.. type: text
-->

The eslint-scope npm package got compromised recently, stealing npm credentials from your home directory. We started running tabletop exercises: what else would you smash-and-grab, and how can we mitigate that risk?

Most people have an RSA SSH key laying around. That SSH key has all sorts of privileges: typically logging into prod and GitHub access. Unlike an npm credential, an SSH key is encrypted, so perhaps it’s safe even if it leaks? Let’s find out!

```clojure
user@work /tmp $ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa): mykey
...
user@work /tmp $ head -n 5 mykey  
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,CB973D5520E952B8D5A6B86716C6223F

+5ZVNE65kl8kwZ808e4+Y7Pr8IFstgoArpZJ/bkOs7rB9eAfYrx2CLBqLATk1RT/
```

You can tell it’s encrypted because it says so right there. It also doesn’t start with `MII` -- the base64 DER clue that an RSA key follows. And AES! That’s good, right? CBC with ostensibly a random IV, even! No MAC, but without something like a padding oracle to try modified ciphertexts on, so that might be OK?

It’s tricky to find out what this DEK-Info stuff means. Searching the openssh-portable repo for the string DEK-Info only shows sample keys. The punchline is that the AES key is just MD5(password \|\| IV\[:8]). That’s not good at all: password storage best practice holds that passwords are bad (low entropy) and in order to turn them into cryptographic key material you need an expensive function like Argon2. MD5 is very cheap to compute. The only thing this design has going for it is that the salt goes after the password, so you can’t just compute the intermediate state of MD5(IV[8:]) and try passwords from there. That’s faint praise, especially in a world where I can rent a machine that tries billions of MD5 calls per second. There just aren’t that many passwords.

You might ask yourself how OpenSSH ended up with this. The sad answer is the OpenSSL command line tool had it as a default, and now we’re stuck with it.

That’s a fair argument to say that standard password-encrypted keys are about as good as plaintext: the encryption is ineffective. But I made a stronger statement: it’s *worse*. The argument there is simple: an SSH key password is unlikely to be managed by a password manager: instead it’s something you remember. If you remember it, you probably reused it somewhere. Perhaps it’s even your device password. This leaked key provides an oracle: if I guess the password correctly (and that’s feasible because the KDF is bad), I know I guessed correctly because I can check against your public key.

There’s nothing wrong with the RSA key pair itself: it’s just the symmetric encryption of the private key. You can’t mount this attack from just a public key.

How do you fix this? OpenSSH has a new key format that you should use. “New” means 2013. This format uses bcrypt_pbkdf, which is essentially bcrypt with fixed difficulty, operated in a PBKDF2 construction. Conveniently, you always get the new format when generating Ed25519 keys, because the old SSH key format doesn’t support newer key types. That’s a weird argument: you don’t really need your key format to define how Ed25519 serialization works since Ed25519 itself already defines how serialization works. But if that’s how we get good KDFs, that’s not the pedantic hill I want to die on. Hence, one answer is ssh-keygen -t ed25519. If, for compatibility reasons, you need to stick to RSA, you can use ssh-keygen -o. That will produce the new format, even for old key types. You can upgrade existing keys with ssh-keygen -p -o -f PRIVATEKEY. If your keys live on a Yubikey or a smart card, you don't have this problem either.

We want to provide a better answer to this. On the one hand, aws-vault has shown the way by moving credentials off disk and into keychains. Another parallel approach is to move development into partitioned environments. Finally, most startups should consider not having long-held SSH keys, instead using temporary credentials issued by an SSH CA, ideally gated on SSO. Unfortunately this doesn't work for GitHub.

PS: It’s hard to find an authoritative source, but from my memory: the versioned parameter in the PEM-like OpenSSH private key format only affect the encryption method. That doesn’t matter in the slightest: it’s the KDF that’s broken. That’s an argument against piecemeal negotiating parts of protocols, I’m sure. We’ll get you a blog post on that later.

The full key is available here, just in case you feel like running john the ripper on something today: [gist.github.com/lvh/c532c...](https://gist.github.com/lvh/c532c8fd46115d2857f40a433a2416fd)

(This post was syndicated on the Latacora blog.)
