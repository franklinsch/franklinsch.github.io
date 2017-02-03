---
layout: post
title: "Animating UILabels for stroke drawing"
date: 2017-02-3
categories: iOS
---

It's been a while since my previous post, [A scrolling animation for Tab Bar Controllers (iOS)]({% post_url 2015-12-25-scrolling-tab-bar %}). Let's stay in the topic of animations, and see how we can create a `UILabel` subclass which gets animated whenever it's displayed, like so:

## UIStrokeAnimatedLabel, a UILabel subclass

