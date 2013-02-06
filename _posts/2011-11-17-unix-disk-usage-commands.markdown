---
layout: post
title: Unix disk usage commands
published: true
tags:
- unix
---

When a unix filesystem fills up the first question asked is usually, "Who is using the most disk space?". Naturally there's many ways of finding out but this works for me.

du -hc * | sort -n | grep "[0-9]G"

This will look for files being measure in Gigabytes and give you a list of the directories using the most. If your file system has smaller files on them substitute the "G" for "M".

If that produces too many result it can be limited this this.

du -hc --max-depth=1 *snapshots* | sort -n | grep "[0-9]G" | tail

This will only go to one level of depth in the directory structure and will be looking only at directories with the word "snapshots" in them.

This also produces a grand total of files. This will always be displayed last in the output because the sort is numeric and it's the highest number.
