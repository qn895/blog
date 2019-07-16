---
title: Writing Meaningful Git Commit Summary
date: "2019-07-16T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/writing-meaningful-git-commit-summary/"
category: "Code Snippets"
tags:
  - "Docker"
  - "Python"
  - "Jupyter"
description: "Writing meaningful, well-formated git commit summary saves time in the long run. Here's a few of my golden rules."
---

I originally wrote these obversations to share as guide with new folks who join my ever growing team (we are about 50 people now!). I hope you find it somewhat helpful.

# The Golden Rules

## Always write meaningful, well formated git commits in a consistent pattern

Reasons:

- Easy for you and other to read
- Easy for you to compile release notes/change log based on commit history

## Reference the issue/task (e.g. add `#123` to reference Github issue)

Reasons:

- Issue is always more descriptive
- Easily traceback what commit did with a simple issue search
- Interwolve snapshot of code with comments/discussion along the thread and see how it changes over time

## Frequent, smaller commits are better than one big commit

Reasons:

- Easily cherry-pick changes
- Easily identify which commit causes a problem using [git-bisect](https://git-scm.com/docs/git-bisect) (which by the way, is the best thing since sliced bread if you are hunting for a bug)

# Patterns

Cosmetically I recommend to capitalize the subject, not use period at the end, and to use verbs in present tense. There's no particular reason, I just think it's clean and descriptive that way.

Also, my manager has a very disciplined way of writing his git commit messages. He always prefix his commits with `[Fix]`, `[New]`, `[Enhance]`, `[Cleanup]` or some variations that specify the type of task.

Originally I thought that's way too much work, but the more and more I work on a project the more I appreciate this system.

Another approach which I currently use is to namespace the commit by **feature**, so `[FeatureNameA]`, `[FeatureNameB]`, and so on. For example:

```
SC: Fix blah blah blah #241
SC: New blah blah blah #234
Textbox: Update to new API v.2.1.0 blah blah blah #123
Textbox: Clean up
Textbox: Refactor blahh blah to improve performance
Version Bump: 1.2.3
```
