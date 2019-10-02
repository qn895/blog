---
title: Protecting VBA Source Code is Futile
date: "2019-08-03T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/vba-security/"
category: "Random Musings"
tags:
  - "Leadership"
description: "And why you probably should not write proprietary code with Excel."
---

A few years ago, I came across a very comprehensive Excel file that has macros handling various engineering calculation and modeling. It was a massive project that even come with its own Microsoft Access database files. I thought it was very nice, but like any engineer, I had lots of doubt regarding how accurate this model is as it is a tricky, heavily debated subject. As such, I wanted to check if 1) the logic behind whatever scripts running is doing what I think it should be doing, and 2) if the calculations and the formulas used are accurate and appropriate for the different scenarios.

Thus I did what anyone would probably do: I enabled developer mode and tried to take a look at the source scripts. Unfortunately, the macros (not the workbook) were protected with a password. At this point I thought to myself "Huh, maybe there's a way". Indeed there was. A google search and fifteen minutes later, I was able to step into the source scripts and look at everything uncompiled. Admittedly, the code was not very well documented nor the variable names self-explainatory, so it wasn't particularly helpful. However, at this point, I realized it was extremely easy to unlock a protected a Excel document.

If unlocking password protected macro xlsm file took me 15 minutes, unlocking password protected documents took me about 2 minutes. All the while step-by-step instruction is literally available with a quick search on stackoverflow. That's ... pretty bad.

Then I decided to check on the access database files that also came with the package. All of which were cracked open in about 5 minutes.

At this point, dear reader, I was truly horrified. What's even more horrifying is it's been years since then, and I was still able to replicate the same steps to unlock.

Please do you coworkers and friends a favor and steer clear of writing proprietary code with Excel, but it is in no way closed source. If you can open source, open source :)!
