---
layout: post
title: "What it's like to be an Android developer"
date: 2020-10-09 12:00:00 -0500
permalink: "/androidy-quirks/"
---

I've often struggled to explain to developers who have never touched Android
about the subtle complexities when building an Android app.

It's the sort of thing where there's so many small instances of inconsistencies
and strange behavior in the SDK that none of them really stick out when you try
and recall them but you run into them constantly.

I recently ran into one such inconsistency and I thought it'd be worthwhile to
jot it down as an example of how tedious the Android SDK can be.

### The Problem

I was recently tasked with fixing a minor style issue in the login page of my
current clients Android app. The issue was that the password input field seemed
to be using a different font than the username input field.

Specifically, the password field was using what looked to be a `monospace` font
whereas the username input field was using the more traditional `sans-serif`
font face.

I figured this was a remnant of some styling change that had moved from a custom
font to the default Roboto font, so the fix would presumably be to remove any
custom font styling on the password input so it would use the default font face.

Alas, I was wrong about what was causing this behavior.

### The Cause

This app, like many apps, has a toggle you can use to decide whether to display
the password in plain text or masked. To do that, it sets up a listener and when
the switch is toggled it updates the password fields `inputType` to either
`InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_VARIATION_PASSWORD` if the
password should be masked or `InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD` if
the password should be visible.

However, it turns out that when you update an `EditText`s input type to use
`InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD` it actually sets the font
family to be monospace. This is specifically called out in official Android
developer guides documentation. In the `Dialogs` section. Again, the `Dialogs`
section:

> Tip: By default, when you set an EditText element to use the "textPassword"
> input type, the font family is set to monospace, so you should change its font
> family to "sans-serif" so that both text fields use a matching font style.

### The Fix

At this point the fix is trivial, albeit absurd. I ended up holding onto a
reference of the old typeface of the `EditText` before changing the input type,
then restoring that typeface so the font family wouldn't change.

This is one minor instance of the complexity in the Android SDK, but I think
it's illuminative  of what it's like to be a day to day Android developer.

Hopefully Jetpack Compose fixes some of these more onerous parts of the Android
View stack.
