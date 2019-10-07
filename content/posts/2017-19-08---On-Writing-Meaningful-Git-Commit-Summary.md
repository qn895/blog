---
title: Writing Meaningful Git Commit Summary
date: "2019-07-16T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/writing-meaningful-git-commit-summary/"
category: "Random Musings"
tags:
  - "Git"
description: "Writing meaningful, well-formated git commit summary saves time in the long run. Here's a few of my golden rules."
---

I originally wrote these obversations to share as guide with new folks who join my ever growing team (we are about 50 people now!). I hope you find it somewhat helpful.

# The Golden Rules

## Always write meaningful, well formated git commits in a _consistent_ pattern

Reasons:

- Easy for you and others to read
- Easy for you to compile release notes/change log based on commit history

## Reference the issue/task (e.g. add `#123` to reference Github issue)

Reasons:

- An issue is always more descriptive and transparent
- Easily traceback what commit did what with a simple search through issue
- See how your code evolve along with the discussions in the thread (and particularly as specs and requirements change)
- See how your commit is linked to multiple issues

## Frequent, smaller commits are better than one big commit

Reasons:

- Easily [cherry-pick](https://git-scm.com/docs/git-cherry-pick) relevant changes to another branch if needed (also super duper helpful if you're rebasing)
- Quickly identify which commit causes a problem using [git-bisect](https://git-scm.com/docs/git-bisect) (which, by the way, is the best thing since sliced bread if you are hunting for a bug)

# Patterns

Cosmetically I recommend to capitalize the subject, not use period at the end, and to use verbs in present tense. There's no particular reason, I just think it's clean and descriptive that way.

Also, my manager has a very disciplined way of writing his git commit messages. He always prefix his commits with `[Fix]`, `[New]`, `[Enhance]`, `[Cleanup]` or some variations that specify the type of task.

Originally I thought that's way too much work, but the more and more I work on a project the more I appreciate this system.

```
[Fix] blah blah blah #241
[Enhance] blah blah blah #234
[New] search side bar so user can search for metric #321
[Upgrade] @com/blah to v.2.1.0 #123
[Clean up] blah blah
[Refactor] blahh blah to improve performance
[Version Bump] 1.2.3
```

## When to write detailed commit messages?

In my team, we typically add in detailed commit messages when we feel like something needs to be clarified. For example:

1. Breaking changes (this allows us to go back the commit easily)
2. Explains bugs that we had a hard time catching
3. Design decisions that are meant to be temporary

Of course, these are things that could be better placed in the code as comments or Github issues. However, I have found that putting it in git commit messages make it easier to search and track down.
