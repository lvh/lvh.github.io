<!--
.. title: 2016 rMBP caveats
.. slug: 2016-rmbp-caveats
.. date: 2016-11-22 07:49:16 UTC-08:00
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

I bought the 2016 15" retina MacBook Pro as soon as it became available. I've
had it for a week now, and there have been some issues you might want to be
aware of if you'd like to get one.

(There are a bunch of links to Amazon in this article. They're not affiliate
links.)

# System Integrity Protection is often disabled

I noticed [via Twitter][twitter-sip] that some people were reporting that [System Integrity
Protection (SIP)][sip] was disabled by default on their Macs. SIP is a mechanism via
which macOS protects critical system files from being overwritten.

You can check if SIP is enabled on your system by running `csrutil status` in a
terminal. Sure enough, SIP was disabled for both me and my wife's new rMBPs. To
enable SIP, boot into the recovery mode (hold ⌘-R when booting), open a
terminal, type `csrutil enable` and reboot.

Perhaps unrelatedly, different out-of-the-box rMBPs appear to have different
builds of OS X Sierra 10.12.1.

[twitter-sip]: https://twitter.com/schwa/status/799160866209828864
[sip]: https://en.wikipedia.org/wiki/System_Integrity_Protection

# Thunderbolt 2 dongle doesn't work with external screens

I have a Dell 27" 4k montior (P2715Q). I used it with my previous-generation
rMBP with a DisplayPort-to-mDP2 cable to connect it to its Thunderbolt 2 port.
When buying my laptop, it suggested I get a Thunderbolt 3 to Thunderbolt 2
dongle. I was expecting to get a Thunderbolt 2 port like the one on my previous
Mac. When I plugged it in to my monitor, it told me that there was a cable
plugged in, but no signal coming from the computer.

My understanding was that the Thunderbolt spec implies PCIe lanes and other
protocols over the same port. Specifically, Thunderbolt 2 means 4 PCI Express
2.0 lanes with DisplayPort 1.2; at a cursory glance, [Wikipedia agrees][tb].
(Thunderbolt 3 adds HDMI 2.0 and USB 3.1 gen 2.)

I spent about an hour and a half on the phone with AppleCare folks. The Apple
support people were very friendly. (I'm guessing their instructions tell them to
never, under any circumstances, interrupt a customer. It was a little weird.) I
was redirected a few times. They had a variety of suggestions, including:

* Changing my monitor to MST mode, which shouldn't be necessary for DisplayPort
  1.2-supporting devices, and did nothing but make my monitor not work with my old
  rMBP either. Fortunately I was able to recover via HDMI to my old laptop.
* Buying the Apple Digital AV Adapter instead. That adapter used HDMI instead of
  mDP2. That's a significant downgrade; my use of DisplayPort was intentional,
  because DisplayPort 1.2 is the only way I can power the 4K display at 60Hz.
  (The new adapter does not support HDMI 2.0, which is necessary for 4K@60Hz.)
* Buying a third-party DisplayPort adapter or dock. This is precarious at best.
  Most existing devices [don't work with the new rMBP][incompat], because they
  use a previous-generation TI chip. There are plenty of docks that wont work,
  by [StarTech][startech], [Dell][dell], [Kensington][kensington]
  and [Plugable][plugable]. I found one Dock by [CalDigit][caldigit] that will
  ostensibly work with the new rMBP, but doesn't supply enough power to charge
  it.

Eventually, we found [a KB article][kb] that spells out that the Thunderbolt
dongle doesn't work for DisplayPort displays:

> The Thunderbolt 3 (USB-C) to Thunderbolt 2 Adapter doesn't support connections to these devices:
>
> * Apple DisplayPort display
> * DisplayPort devices or accessories, such as Mini DisplayPort to HDMI or Mini DisplayPort to VGA adapters
> * 4K Mini DisplayPort displays that don’t have Thunderbolt

I'm a little vindicated by the [Mac Store][macstore] review page for the dongle;
apparently I wasn't the only person to expect that. (I was unable to see the
reviews before my purchase, because I purchased it with my Mac, which doesn't
show reviews. Also, the product was brand new at the time, and didn't have these
reviews yet.)

[Belkin][belkin] and [OWC][owc] will be shipping docks that allegedly work with
the new rMBP, but Belkin's is currently unavailable with no ship date mentioned,
and OWC claims February 2017.

[kb]: https://support.apple.com/en-us/HT207266
[incompat]: https://9to5mac.com/2016/11/03/2016-macbook-pro-thunderbolt-compatibility-issues/
[tb]: https://en.wikipedia.org/wiki/Thunderbolt_(interface)
[belkin]: http://www.belkin.com/us/p/P-F4U095/
[owc]: https://9to5mac.com/2016/11/03/owc-announces-thunderbolt-3-dock-adds-13-ports-of-legacy-io-to-the-new-macbook-pros-over-a-single-cable/
[startech]: https://www.amazon.com/StarTech-com-Thunderbolt-Dual-4K-Docking-Station
[dell]: https://www.amazon.com/Dell-Dock-WD15-Adapter-Type-C
[kensington]: https://www.amazon.com/Kensington-Delivery-DisplayPort-Microphone-K38231WW
[plugable]: https://www.amazon.com/Plugable-Display-Docking-Charging-Delivery
[caldigit]: https://www.amazon.com/CalDigit-USB-C-Docking-Station-DisplayPort
[macstore]: http://www.apple.com/shop/reviews/MMEL2AM/A/thunderbolt-3-usb-c-to-thunderbolt-2-adapter

# WiFi failing with USB-C devices plugged in

Just as I was going to start writing this post, I noticed that I wasn't able
to sync my blog repository from GitHub:

```
Get https://api.github.com/repos/lvh/lvh.github.io: dial tcp 192.30.253.116:443: connect: network is unreachable
```

It didn't click at first what was going on. I restarted my router, connected to
different networks, tried a different machine -- all telling me it was this
laptop that was misbehaving. I started trying everything, and realized I had
recently plugged in my WD backup drive from which I was copying over an SSH key.
It's a USB 3.0 drive that I'm connecting via an AUKEY USB 3 to USB-C converter.
I removed the drive, and my WiFi starts working again. Plugging it back in does
not instantly, but eventually, break WiFi again.

After searching, I was able [to find someone with the same problem][wifi-video].
It is unclear to me if this issue is related to the first-gen TI chip issue
mentioned above. In that video, the authors are also using a USB 3.0 to USB-C
plug, albeit a different one from mine. I don't have a reference USB-C machine
that isn't a new 2016 rMBP to test with. However, this seems plausible, because
the USB 3.0 dongle I purchased from Apple ostensibly works fine.

This does not seem like a reasonable failure mode.

[wifi-video]: https://www.youtube.com/watch?v=NYVjIjBMx6o

# The escape key, and the new keyboard

I spend most of my day in Emacs. I'm perfectly happy with the new keyboard. I've
also used the regular MacBook butterfly keyboard, and the new version is
significantly better. I've never had a problem with not having an escape key;
every app where I would've cared to press it had an escape key drawn on the new
Touch Bar. However, not having tactile feedback for the escape key is annoying.
When I was setting up my box and quickly editing a file in vim, I successfully
pressed Escape to exit insert mode -- but I ended up pressing it five times
because I thought I didn't hit it. Apparently the visual feedback vim gives me
that I've exited insert mode is not, actually, what my brain relies on. I'll let
you know if I get used to it.

# Charging

I'll miss the safety of Magsafe, but being able to plug in your charger on
either side is an unexpected nice benefit.

# Conclusion

I was ready to accept a transition period of dongles; I bought into it,
literally and figuratively. However, most of the dongles don't actually work,
and that sucks. So, maybe wait for the refresh, or at least until the
high-quality docks are available.
