---
layout: post
title: Git – Personal .gitignore
---

If you use git you propably know about `.gitignore`. A plain-text file residing in the root or any subfolder of your git repository that allows you to declare filename patterns that git should ignore if the corresponding files are untracked.

However, did you know that `.gitignore` has a less prominent cousin?

Let me introduce you to `.git/info/exclude`!

It works excactly like your root-level `.gitignore`, except it is not tracked.
This comes in very handy if you

1. use tools others in your team don't
1. want to dump temporary files
1. want to take notes

I mostly use it for smaller notes. A quick list of things I want to bring up in the next review, notes during a meeting, you name it. Things that are usually a form of draft for things that I want to either think about a bit more before communicating them to the team or are just for me. Rough thoughts, gut feelings, notes for upcoming reviews...

I my case `.git/info/exclude` simply looks like this:

```
*.mene.*
```

Any file that contains `.mene.` is excluded from git and I will never accidentally commmit my `retro.mene.md`.
