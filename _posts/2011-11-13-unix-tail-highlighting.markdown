---
layout: post
title: Unix tail highlighting
published: true
tags:
- unix
---

Often whilst tailing a log file in Unix you want to have the specific thing you are watching for highlighted as it appears. Here are two ways of achieving this.

Option 1 - tail/perl

t() {
tail -f -n5000 $1 | perl -pe "s/$2/\e[1;31;43m$&\e[0m/g"
}

Option 2 - sed

sed filename.txt
/string-to-watch-for
F

