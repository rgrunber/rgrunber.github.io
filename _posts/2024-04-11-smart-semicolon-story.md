---
layout: post
title:  "On smart semicolon insertion"
date:   2024-04-11 17:00:00 -0400
categories: vscode java eclipse
---

Sometimes, you just stare up at the sky and wonder:

_Why doesn't Eclipse enable the smart semicolon insertion typing feature by default?_

![typing-preference-page](/assets/smart-semicolon/typing-preferences-smart-semi.png)

It was introduced in **May 2003** and was [enabled by default](https://github.com/eclipse-jdt/eclipse.jdt.ui/commit/09042735f28cab1fe445facb207aea684cdb92f1#diff-db395f92942188463ea1b85dcf787e05b26258f5d399d77c66ea5dc00657594aR1882) . It was a work in progress at that point so I'm not surprised it was disabled. Later it was moved into [WorkInProgressPreferencePage](https://github.com/eclipse-jdt/eclipse.jdt.ui/commit/ec7b2dd37505d470cae76df52750aa85214479ce) where that page was eventually removed.

It was proposed at [Bug 363231](https://bugs.eclipse.org/bugs/show_bug.cgi?id=363231) but denied for being a fairly "advanced" feature, and the perception that users prefer a minimalistic typing environment.

On the last discussion of this feature, it was considered for default enablement along with the auto-insertion of braces at [Bug 505816](https://bugs.eclipse.org/bugs/show_bug.cgi?id=505816). The only remaining issue mentioned that applies today is [Bug 506148](https://bugs.eclipse.org/bugs/show_bug.cgi?id=506148) (support for wrapped statements), and I can confirm it exists.

So basically it comes down to, either :

1. Ignore the last issue and enable by default because it's good enough
2. Fix [Bug 506148](https://bugs.eclipse.org/bugs/show_bug.cgi?id=506148) and then enable the feature by default
3. Do nothing

In my experience, if a feature isn't enabled by default, most won't see it so it's a shame to see this disabled for so long.
