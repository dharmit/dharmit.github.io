---
layout: post
title: Fedora DNF `fastestmirror=1` For The Win!
categories:
- blog
---

If you're using one of the recent versions of [Fedora](https://getfedora.org/),
chances are you've been using
[DNF](https://github.com/rpm-software-management/dnf) as default package
manager. DNF (a.k.a Dandified Yum) was introduced in Fedora 18 and became the
default package manager in Fedora 22. You can read more about DNF on its
official [Fedora wiki](https://fedoraproject.org/wiki/Dnf).

Lately, I was observing slow speeds while performing `dnf update` or `dnf
install` especially when the cache is being updated. I thought it was an issue
with my ISP. However, a few speed tests proved that my speculation was wrong
and ISP was not the culprit.

It then struck me that it might have something to do with the Fedora mirrors.
I dunno which mirror was being used but the speed was downright pathetic! A
bit of Googling led me to search for fastestmirror plugin for DNF. Thankfully,
before spending too much time on it, I stumbled upon an [answer on Ask
Fedora](https://ask.fedoraproject.org/en/question/7960/how-to-choose-a-specific-mirror-source/?answer=74473#post-id-74473)
forums which indicated that DNF has the fastestmirror plugin built into it!

All I had to do was set `fastestmirror=1` in the DNF configuration file under
`/etc/dnf/dnf.conf` and check things again. I was pleasantly suprised when the
download speed increased by upto 3 times it was earlier.

Obviously, I should have done a bit of RTFM, but gladly this time I wasn't
hurt by spending hours searching around for something that didn't exist and was
built-in! :smile:
