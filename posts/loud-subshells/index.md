<!--
.. title: Loud subshells
.. slug: loud-subshells
.. date: 2018-06-21 11:21
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

Default shells usually end in $. Unless you're root and it's #. That
tradition has been around forever: people recognized the need to highlight
you're not just some random shmoe.

These days we have lots of snazzy shell magic. You might still su, but you're
more likely to sudo. We still temporarily assume extra privileges. If
you have access to more than one set of systems, like production and staging,
you probably have ways of putting on a particular hat. Some combination of
setting an environment variable, adding a key to ssh-agent, or assuming aws AWS
role with [aws-vault](https://github.com/99designs/aws-vault). You know, so you don't accidentally blow away
prod.

If a privilege is important enough not to have around all the time, it's
important enough to be reminded you have it. You're likely to have more than one
terminal open. You might want to be reminded when your extra privileges are
about to expire. That might be something you just set up for your own
environment. But as your organization grows, you'll want to share that with
others. If you develop software, you might want to make it easy for your users
to get a similarly loud shell.

Major shells you might care about: POSIX sh, bash, zsh, and [fish](https://fishshell.com/). POSIX
sh is a spec, not an implementation. POSIX sh compatibility means it'll probably
work in bash and zsh too. You might run into dash or busybox in a tiny Docker
container image. Both are POSIXy. There's large overlap between all of them but
fish, which is different and proud of it.

There are lots of ways to configure a shell prompt but PS1 is the default.
Just PS1="xyzzy>" $SHELL doesn't work. You get a new shell, but it will
execute a plethora of configuration files. One of them will set PS1
indiscriminately and so your fancy prompt gets clobbered.

So what do you do? Major techniques:

* An rc-less shell.
* A dedicated environment variable that your shell's PS1 knows about.
* Sourcing scripts
* Crazy hacks like PROMPT_COMMAND or precmd or reconstituted rcfiles

**rcless shells** If you interpret the problem as shells having configuration, you
can disable that. Unfortunately, the flags for this are different across shells:

* bash uses the --norc flag
* zsh uses the --no-rcs flag
* dash doesn't have a flag but instead reads from ENV

You can't necessarily count on any particular shell being available. The good
news is you don't have to care about fish here: it's uncommon and you've already
committed to giving people a limited shell. So, either count on everyone to have
bash or write something like this:

```sh
SHFLAGS=""
case "$SHELL" in
    *zsh*) SHFLAGS="--no-rcs" ;;
    *bash*) SHFLAGS="--norc" ;;
esac
```

Eventually, you run $SHELL with PS1 set and you're done. The good news is
that rcless shells will work pretty much anywhere. The bad news is that the
shell customization people are used to is gone: aliases, paths, colors, shell
history et cetera. If people don't love the shell you give them, odds are
they're going to look for a workaround.

**Dedicated environment variables** If you interpret the problem as setting PS1
being fundamentally wrong because that's the shell configuration's job, you
could argue that the shell configuration should also just set the prompt
correctly. Your job is not to set the prompt, but to give the shell everything
it needs to set it for you.

As an example, aws-vault already conveniently sets an environment variable for
you, so you can do this in your zshrc:

```sh
if [ -n "${AWS_VAULT}" ] ; then
  echo -e "$(tput setab 1)In aws-vault env ${AWS_VAULT}$(tput sgr0)"
  export PS1="$(tput setab 1)<<${AWS_VAULT}>>$(tput sgr0) ${PS1}";
fi;
```

Now, just use aws-vault's exec to run a new shell:

```sh
aws-vault exec work -- $SHELL
```

... and you'll get a bright red warning telling you that you have elevated
privileges.

I use tput here and you should too. Under the hood, it'll produce regular old
ANSI escape codes. $(tput setab 1) sets the background to color 1 (red).
$(tput sgr0) resets to defaults. Three reasons you should use tput instead of
manually entering escape codes:

1. It's more legible.
1. It's more portable across shells. If you did PS1="${REDBG}shouty${NORMAL}"
   it'll work fine in bash but zsh will escape the escape codes and your
   prompt will have literal slashes and brackets in it. Unless you put a dollar
   sign in front of the double quote, which bash doesn't like.
1. It's more portable across terminals.

The downside to this is that it fails open. If nobody changes PS1 you don't
get a fancy prompt. It's a pain to enforce this via MDM.

**source** If you reinterpret the problem as trying to create a subshell at all,
you could try to modify the shell you're in. You can do that simply by calling
*source somefile*. This is how Python's virtualenv works.

The upside is that it's pretty clean, the downside is that it's pretty custom
per shell. If you're careful, odds are you can write it in POSIX sh and cover
sh, bash, zsh in one go, though. Unless you need to support fish. Or eshell or
whatever it is that one person on your team uses.

**PROMPT_COMMAND** Let's say you have no shame and you interpret the problem as
shells refusing to bend to your eternal will. You can get past the above
restriction with the rarely used PROMPT_COMMAND environment variable. [Quoth
the docs](http://tldp.org/HOWTO/Bash-Prompt-HOWTO/x264.html):

> Bash provides an environment variable called PROMPT_COMMAND. The contents of
> this variable are executed as a regular Bash command just before Bash displays
> a prompt.

Because nobody sets PROMPT_COMMAND you can set it in the environment of a new
shell process and it won't get clobbered like PS1 would. Because it's bash
source, it can do whatever it wants, including setting environment variables
like PS1 and un-setting environment variables like PROMPT_COMMAND itself. You
know, something like:

```sh
PROMPT_COMMAND='PS1="${PS1} butwhy>";unset PROMPT_COMMAND'
```

Of course, you have now forfeited any pretense of being reasonable. This doesn't play well with others. Another
downside is that this doesn't work in zsh. That has an equivalent precmd()
function but won't grab it from an environment variable. Which brings us to our
final trick:

**Reconstituted rcfiles** If you absolutely must, you could technically just
cobble together the entire rcfile, including the parts that the shell would
ordinarily source itself. I don't really want to help you do that, but it'd
probably look something like this:

```sh
$ zsh --rcs <(echo "echo why are you like this")
why are you like this
```

Please don't do that. Depending on context, use an rc-less shell or a dedicated
environment variable or source something. Please?

(This post was syndicated on the Latacora blog.)
