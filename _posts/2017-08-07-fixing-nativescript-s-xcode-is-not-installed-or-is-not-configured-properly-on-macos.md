---
layout: blog
category: blog
published: true
title: >-
  Fixing NativeScript's "Xcode is not installed or is not configured properly"
  on MacOS
splash: ''
tags: ''
---
# Run the following command:

`sudo xcode-select --reset`

# Why does this fix it?

As [GitHub user @thowerton explains](https://github.com/NativeScript/nativescript-cli/issues/1064#issuecomment-275924430):

> "The default path for Xcode when installing Xcode command line tools was incorrect. You can confirm if this is your issue by running $ xcodebuild , which will show an error like this if this is your issue:
>
> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance"

The command above resets this.
