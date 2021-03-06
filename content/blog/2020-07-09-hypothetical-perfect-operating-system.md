---
title: "Hypothetical perfect operating system"
description: "Non-system programmer's vision of a perfect OS."
date: 2020-07-09

slug: "hypothetical-perfect-operating-system"
---

Disclaimer: I'm not a system programmer, and I don't know what I'm
talking about. Also your vision of a perfect OS can be quite different
from mine.

## Basic requirements

For obvious reasons, the OS would be [free software], 100% DRM and
spyware free, actively developed, and actually useful in its area. It
would also probably follow some of the common software standards
(like, for example, JSON for IPC and command line utilities), use
consistent naming and versioning schemes, and be written in a
memory-safe language.

[free software]: https://www.gnu.org/philosophy/free-sw.html

### Existing implementations

* Most of GNU/Linux and BSD systems (partially)
* Some of hobbyist systems (partially)

## Simple file hierarchy

Both of the 2 most popular file hierarchies, the Windows NT one and
the Unix one (including whatever is built on top of it), are quite
complicated due to historical reasons. Many of the directories are no
longer used for their intended functionality, and some are practically
duplicated.

A good one would look something like this:

* `/`
	* `system/`
		* *Stuff*
	* `users/`
		* `username/`
			* *Stuff*
		* ...

Where *Stuff* is:

* `apps`
	* `appname`
		* Whatever resources the app is supposed to be
		  installed with
	* ...
* `resources`
	* `binaries`
	* `libraries`
	* `themes`
	* Whatever else can be shared between apps
* `data`
	* `appname`
		* `cache`
		* `config`
		* Whatever else state or configuration the app can
		  have
	* ...
* `documents`, `downloads`, `music`, `videos`, etc

### Existing implementations

* macOS (kind of)
* Flatpak (partially)

## Everything is an object

Having a shitton of protocols, for which you often don't have enough
tools to easily explore, is terrible for discoverability. Everything
is a file was supposed to fix that, but the solution was awkward and
hard to deal with. So, instead of pretending that everything is a
file, it'd make sense to accept that some "files" are not actually
files and implement a universal way to explore them.

For example, a scheme could be created for each subsystem of the OS,
allowing access to its objects via a URL like `file:/system/apps/`.
Here's an example of what schemes could be created:

* `file` for the root file system and `disk` for any
* `http{,s}`, `{quic,tcp,udp}`, `ip` for different levels of
  network access
* `process` for signals, process information, and some IPC
* `network`, `settings`, `audio`, `video`, etc for high-level
  configuration/IO
* `tty`, `desktop`, `window`, `notification`, etc for UI
* `device` for low-level IO
* A custom scheme registered by its (in-library or standalone)
  handler

### Existing implementations

* Redox
* D-Bus (kind of)

## Mandatory app sandboxing

For obvious security reasons, no app should be ever accepted as 100%
trustworthy and bug-free, and sandboxing never lets that happen. That
is unless there's a way to disable the sandbox or let some apps act
outside of it, which will unavoidably get used, defeating the whole
purpose of sandboxing.

Considering the file hierarchy from the previous point, it'd look
something like this:

* By default, an app only has access to the `file` protocol
  and can only access its own files (read only), data, and
  (shared) resources it needs.
* It can ask for access to an additional protocol, resource,
  or files/data of another app.
* An app can be given a temporary or a permanent permission
  for such access.
* Each permission request must be approved or disapproved by
  the user.
* File dialogs are used to acquire permission to files that
  the user needs to open.

### Existing implementations

* Android
* Flatpak
* OpenBSD

## Conclusion

It seems to be reasonable to just improve GNU/Linux instead of
creating a whole new OS, and some progress is currently being made.
