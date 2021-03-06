---
title: "The story of GNU/Linux on an unbranded Chinese mini PC"
description: "How I made a mistake in 2018 and why shouldn't repeat it."
date: 2020-03-22

slug: "the-story-of-gnu-linux-on-an-unbranded-chinese-mini-pc"
---

Sometime in 2018, I decided to buy a $70 Chinese mini PC and use it as
my main machine. As you can see by the price, this wasn't by far the
best decision, but, after intensely torturing both myself and my
machine for some time, I did manage to get the shit working properly.
Here are the steps I had to take to do it:

## Fix the Intel Graphics color range problems

This problem became apparent almost instantly, as it was visible as
early as the OS starts to boot up. It manifested itself in form of
washed out colors, especially black. The problem was caused by
improper display color range detection and, according to [the bug
tracker], was apparently fixed *2 times*, but the fix still didn't
work on my machine until very recently with kernel version 5.6.0-rc5.
And I'm still not sure if the fix will last.

[the bug tracker]: https://bugs.freedesktop.org/show_bug.cgi?id=108821

Update: it didn't.

In any case, after I acknowledged the existence of the problem, the
solution was quickly found on [Arch wiki]:

[Arch wiki]: https://wiki.archlinux.org/index.php/Intel_graphics#Weathered_colors_(color_range_problems)

```
xrandr --output OUTPUT-DEVICE --set 'Broadcast RGB' 'Full'
```

And it worked fine until I decided to switch to Wayland. No Wayland
utility I know of provides an ability to set an "output property" yet,
so it took me quite some time to figure out how to do that, and I had
to launch an X server just to fix the problem after each reboot.
Luckily, I bumped into [this post]. The required utility was called
`proptest`, and it acted on DRM itself without touching the display
server, so it had to be launched in the framebuffer console. I used
the one from the `drm-utils` Fedora package and launched it manually
at first but (after failing to make a dracut module) wrapped it in a
systemd service later:

[this post]: https://www.brad-x.com/2017/08/07/quick-tip-setting-the-color-space-value-in-wayland/

`/usr/local/lib/systemd/system/broadcast-rgb-workaround.service`:

```
[Unit]
Description=Work around the broken Broadcast RGB detection

[Service]
Type=oneshot
ExecStart=/usr/local/bin/broadcast-rgb-workaround

[Install]
WantedBy=basic.target
```

`/usr/local/bin/broadcast-rgb-workaround`:

```
#!/bin/sh

chvt 12

proptest CONNECTOR-NUMBER connector 97 1

chvt 1
```

(The numbers may alternate, and they did one time, so they have to be
looked up manually by just launching `proptest`.)

## Fix the broken WiFi

The solution to this was found almost instantly by searching the
kernel error message. I just had to copy [this random firmware file]
to `/usr/lib/firmware/brcm/`.

[this random firmware file]: https://raw.githubusercontent.com/RPi-Distro/firmware-nonfree/master/brcm/brcmfmac43455-sdio.txt

## Fix the broken audio

The sound card on this machine is called `bytcr-rt5651`, and it was a
huge pain in the ass to fix. First off, I just refused to use
PulseAudio for quite some time. I'm not entirely sure what my motives
were, but I just accepted having no sound at all. Why it didn't work
with just bare ALSA is a whole other question. When I decided that
there's actually nothing wrong with PulseAudio, the problems didn't
end and it took me some time to find [this set of files] and apply it
properly. One problem was that some UCM files were already included by
default, but they were less correct than the ones in the set.

[this set of files]: https://github.com/plbossart/UCM/tree/master/bytcr-rt5651

But that fixed only the software problems, and the hardware problems
were much more ridiculous. The 4.5 mm audio jack on this machine acts
as a headphone jack and a microphone jack at the same time.
**Literally at the same**. Plugging the headphones there almost made
me shit my pants the first time as I heard my own movement sounds
amplified. But before I even thought of plugging the headphones
directly, I used a female-to-male jack adapter, which I didn't even
realize was dead. How stupid could I be to not check it for such a
long time. In any case, I just found another one and plugged my
headphones with it.

But then my headphones died. The only options available were the
full-blown speakers and the Bluetooth headphones. The first option
wasn't actually an option because the machine doesn't have enough
horsepower to power them, so I went for the second one and failed.
Bluetooth is supported by this machine and I could even detect the
headphones and connect them, but no audio played whatever I tried. I
didn't have much time to do anything with it, so I gave up for some
time and had no sound once again.

After some time, I suddenly remembered that I had another, very big
compared to the machine itself, speakers with amplification. The idea
looked kinda silly because the speakers are literally, like, ten times
the volume of the PC, and my setup was already fucked up through the
roof, so introducing another cable didn't sound too great. I didn't
have much choice, so I went for it in any case, and everything's been
(mostly) okay since.

## (Mostly) fix the C-State transition freeze

Intel Atom x5-Z8350 used on this machine, in addition to being a slow
piece of shit and having graphical problems, is vulnerable to freezing
on certain [C-State] transitions (see [this bug report]). And it seems
like other operating systems like Windows are also affected by this,
but I couldn't test that because I didn't use Windows on this machine
for longer than 10 minutes.

[this bug report]: https://bugzilla.kernel.org/show_bug.cgi?id=109051
[C-State]: https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface#Processor_states

The most common solution for this kind of problem is to use [this
script], but it doesn't work. Adding `intel_idle.max_cstate=1` to the
kernel boot parameters also doesn't help, but adding
`intel_idle.max_cstate=0` (which disables `intel_idle` as a whole and
switches to `acpi_idle`) appears to work. It seems like it doesn't
completely eliminate freezing, but it happens less often, and
*permanent* freezing almost doesn't happen. Overall, this issue is not
solved yet, and I'm still looking for updates.

[this script]: https://bugzilla.kernel.org/attachment.cgi?id=223851

## Conclusion

I hugely fucked up by bying this computer, and I hope you won't repeat
my mistakes. Everything, however, is not that bad after all because of
the incredible work of the developers of the Linux kernel, wl-roots,
GTK, Firefox, and others, and things are probably going to get even
better. I'm still going to use this machine for some time, but I will
choose my next one more carefully.
